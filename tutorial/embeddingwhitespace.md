# Section 4: Integrating Whitespace into HaPyLi Applications

## Whitespace Assembler

There are only two "built-in" expressions in HaPyLi, the `if` statement and the
`do` statement. Everything else - including arithmetic and IO - are defined
within HaPyLi. HaPyLi allows you to embed Whitespace commands directly into your
applications.

Of course, coding with nothing but spaces, tabs, and line feeds is tedious,
unreadable, and error prone. HaPyLi includes within it a little assembly
language that closely parallels Whitespace itself. The commands for this
assembly language are listed below (please refer to the
[Whitespace tutorial](https://web.archive.org/web/20150618184706/http://compsoc.dur.ac.uk/whitespace/tutorial.php)).

### Stack Manipulation

Applicable operands are single-quoted character literals, decimal integers, and
hexadecimal integers.

    push n      - push the number 'n' onto the stack.
    dup         - Duplicate the top item on the stack.
    copy n      - Copy the 'n'th item on the stack onto the top of the stack.
    swap        - Swap the top two items on the stack.
    pop         - Discard the top item on the stack.
    slide n     - Slide 'n' items off the stack, keeping the top item.

### Arithmetic

    add         - Addition
    sub         - Subtraction
    mul         - Multiplication
    div         - Integer Division
    mod         - Modulo

### Heap Access

    store       - Store
    load        - Retrieve

### Flow Control

Applicable operands follow mostly the same syntax as HaPyLi identifiers. For
example, `label abc` or `call hello_world`.

    label x     - Mark a location in the program.
    call  x     - Call the subroutine.
    jump  x     - Jump unconditionally to a label.
    jz    x     - Jump to a label if the top of the stack is zero.
    jn    x     - Jump to a label if the top of the stack is negative.
    ret         - End a subroutine and transfer control back to the caller.
    end         - End the program.

### IO

    pc          - Print the character at the top of the stack.
    pn          - Print the number at the top of the stack.
    rc          - Read a character and place it in the location given by the top of the stack.
    rn          - Read a number and place it in the location given by the top of the stack.

### Annotated Example

Here's the Whitespace tutorial's example of a program which counts from 1 to 10,
printing the current value as it goes - translated to HaPyLi + Whitespace
Assembler.

[File: count.hpl](./count.hpl)

```hapyli
asm main () =
(
    push    1           ; Put a 1 on the stack.
    label   begin       ; Set a Label at this point.
    dup                 ; Duplicate the top stack item.
    pn                  ; Output the current value.
    push    '\n'        ; Put a newline character on the stack...
    pc                  ; ...and output the newline.
    push    1           ; Put a 1 on the stack.
    add                 ; Addition. This increments our current value.
    dup                 ; Duplicate that value so we can test it.
    push    11          ; Push 11 onto the stack.
    sub                 ; Subtraction. So if we've reached the end, we have a
                        ; zero on the stack.
    jz      the_end     ; If we have a zero, jump to the end.
    jump    begin       ; Jump to the start.
    label   the_end     ; Set the end label.

)
```

If you compare this with the example in the Whitespace tutorial, you'll notice
that the last two instructions have been removed - namely, discarding the top
item of the stack and explicitly calling `end`. In HaPyLi, all functions must
leave exactly one item on the stack as a return value and the `end` instruction
is added automatically by the compiler.

## Functions defined with Whitespace Assembler

Other than their bodies, the only difference between assembler functions and
ordinary ones is the use of the `asm` keyword. Assembler functions can call and
be called by any other function, they can be inlined, they can contain
let-forms, and they always return a value.

### Handling Parameters / Local Variables

The following example illustrates how to refer to parameters and local variables
passed to a function through Whitespace Assembler. These variables are indexed
starting from 0, in reverse order in which they're defined.

[File: asmparams.hpl](./asmparams.hpl)

```hapyli
import "stdlib/base.hpl"

asm f (a b) =
    let
        c = 2
        d = 1
    in
    (
        copy 3      ; Copy parameter 'a' and print it.
        pn
        push '\n'
        pc

        copy 2      ; Copy parameter 'b' and print it.
        pn
        push '\n'
        pc

        copy 1      ; Copy local variable 'c' and print it.
        pn
        push '\n'
        pc

        copy 0      ; Copy local variable 'd' and print it.
        pn
        push '\n'
        pc

        pop         ; Pop the arguments and local variables
        pop         ; off the top of the stack.
        pop
        pop

        push 0      ; Push the return value (just '0' in this case),
                    ; onto the top of the stack.
    )

def main () = (print-number (f 4 3))
```

Note that we don't explicitly return from `f`. The HaPyLi compiler automatically
adds the `ret` instruction to non-inlined functions and inlined functions should
never contain a `ret` anyway. Remember that 'inlined' functions are not actually
called, but rather their bodies are copied verbatim to wherever the function is
referenced. Hence, it doesn't make sense to return from an inline function.

**IMPORTANT:**

When coding in Whitespace Assembler, it is very easy to accidentally corrupt the
stack, resulting in nasty and difficult to fix bugs. Functions always pop all of
their arguments and local variables, and they push exactly one return value.

### Labels in Whitespace Assembler Functions

Never use labels in inlined functions. A label must only be defined once in an
entire application. Inlining an assembler function, however, copies its body -
including any label instructions - to wherever the function is referenced.

[File: inline_labels.hpl](./inline_labels.hpl)

```hapyli
import "stdlib/base.hpl"

asm inline f () =
(
    jump  HELLO

    label HELLO
    push 0
)


def main () =
    (do (print-number (f))
        (print-number (f)))
```

The above program won't even compile because there are two definitions of the
label `HELLO`. Remove the `inline` modifier, then this program should work.

### Whitespace Assembler Calling Conventions

All non-inlined functions in HaPyLi are marked by a label named according to the
following convention: function name, followed by `~` (tilde character), followed
by the number of parameters that the function accepts. The example below
illustrates how to use this convention to call other HaPyLi functions from
Whitespace Assembler.

[File: asmcall.hpl](./asmcall.hpl)

```hapyli
import "stdlib/base.hpl"

def f () = (print-string "Hello!\n")
def f (*msg) = (print-string *msg)

asm main () =
    let
        *msg = "Goodbye!\n"
    in
    (
                    ; f~0 accepts no arguments.
        call f~0    ; Call f~0
        pop         ; Pop the return value of f~0

                    ; f~1 accepts 1 argument.
        copy 0      ; Pass in *msg.
        call f~1    ; Call f~1
        pop         ; Pop the return value of f~1

        pop         ; Pop parameters and local variables
                    ; (just *msg in this case).

        push 0      ; Push a return value.
    )
```

Inlined functions cannot be called from Whitespace Assembler because they are
not associated with any Whitespace labels.

## The Heap Pointer

The HaPyLi compiler automatically reserves memory on the heap for global
variables and string literals. Afterward, it writes the address of the
remaining, unallocated memory at address `0` on the heap. This is used by the
HaPyLi Standard Library, especially the `alloc` and `read-string` functions, to
dynamically allocate more memory at runtime.

The only time you'll ever mess with the heap pointer is when writing a new
memory management system for HaPyLi. So, unless you really know what you're
doing and you have far too much time on your hands, you should never have to
worry about the heap pointer, ever.

## Tutorial

1. [Functions](./functions.md)
2. [Expressions](./expressions.md)
3. [Variables and the Heap](./variablesandtheheap.md)
4. [Embedding Whitespace](./embeddingwhitespace.md)

Copyright ??2010
