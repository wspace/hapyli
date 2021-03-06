# Section 2: Expressions

HaPyLi is a typeless language. Every expression will evaluate to an integer, so
it's up to the programmer to remember what data is passed to and returned by a
function.

## Numeric Literals

HaPyLi defines three kinds of numeric literals - decimal integers, hexadecimal
integers, and characters. These literals may be used in place of any expression,
including a function body.

[File: literal.hpl](./literal.hpl)

```hapyli
import "stdlib/base.hpl"

def f () = 42       ; You can declare "constant functions" too! Literals are
                    ; no different than any other expressions.

def main () =
    let
        a = 65      ; a, b, and c all equal the same value.
        b = 0x41
        c = 'A'
    in
        (do (print-number a) (print-char '\n')
            (print-number b) (print-char '\n')
            (print-number c) (print-char '\n'))
```

## String Literals

Strings are slightly different than other literals. The HaPyLi compiler converts
them into null-terminated arrays of characters and stores them in Whitespace's
heap memory. A String literal or a variable containing a string will evaluate to
its address in heap memory.

Both string and character literals support the following common character escape
codes:

- `\s` -> space
- `\t` -> tab
- `\r` -> carriage return
- `\n` -> line feed
- `\0` -> null
- `\'` -> `'`
- `\"` -> `"`

In addition, any escaped character not listed above simply translates to itself.

## Built-In Functions

So far, you've seen plenty of examples of seemingly "built-in" functions in
HaPyLi, such as `print-string`, `print-number`, or the arithmetic operators.
Actually, there is nothing special about these functions. They are defined
within HaPyLi itself, in the file `stdlib/base.hpl`. This is possible because
HaPyLi allows you to embed Whitespace commands directly in your applications.

There are only two functions that are special to the compiler: `if` and `do`.

## If Expressions

Normally, when you call a function in HaPyLi all of its arguments are evaluated
first, in order from left-to-right, and then the function itself is executed. If
all functions behaved this way, it would be impossible to control the flow of
the application. `if` is the only function which allows short
circuiting.

The syntax of `if` expressions in HaPyLi is the same as those in Lisp.

[File: if.hpl](./if.hpl)

```hapyli
import "stdlib/base.hpl"

def main () =
    (if (read-number)
        (print-string "TRUE")
        (print-string "FALSE"))
```

Since HaPyLi is typeless, the condition of the if-statement behaves much as it
would in C. The value 0 represents `false` and any non-zero value represents
`true`. Some of the functions in the HaPyLi Standard Library, however, assume
that `true` is denoted by the value 1 instead of just anything non-zero, so you
must be careful.

## Do Expressions

Sometimes you want to call several functions in order - a series of print
functions, for example. Since all function calls evaluate their arguments in
order from left-to-right, it's very easy to define a function to sequence
operations together. Such a function doesn't actually have to do anything.
Unfortunately, there's a small caveat to this approach as the example below
illustrates:

[File: execute.hpl](./execute.hpl)

```hapyli
import "stdlib/base.hpl"

def inline execute (x y) = 0        ; Perform two operations in order.
def inline execute (x y z) = 0      ; Perform three operations in order
def inline execute (w x y z) = 0    ; Perform four operations in order
                                    ; ... and so on

def main () =
    (execute (print-string "ABCDEFG\n")
             (print-string "1234567\n"))
```

We don't need to do this. HaPyLi provides the `do` function, which can accept
any number of parameters. You don't need to pollute your application with 100
`execute` definitions as above just because you want to chain together 100
operations (although I can't imagine you ever wanting to chain together more
than just a few - but at least you now can).

[File: do.hpl](./do.hpl)

```hapyli
import "stdlib/base.hpl"

def main () =
    (do (print-string "Hello, ")
        (print-string "world!\n")
        (print-string "Testing\n")
        (print-string "... and so on... \n"))
```

Furthermore, `do` expressions return the value returned by their last argument.
This can be very useful for computing and returning a result after a series of
IO operations, like in prompts.

[File: prompt.hpl](./prompt.hpl)

```hapyli
import "stdlib/base.hpl"

def prompt () =
    (do (print-string "Enter your favorite number: ")
        (read-number))

def main () =
    let
        x = (prompt)
    in
        (do (print-string "You entered: ")
            (print-number x))
```

## Tutorial

1. [Functions](./functions.md)
2. [Expressions](./expressions.md)
3. [Variables and the Heap](./variablesandtheheap.md)
4. [Embedding Whitespace](./embeddingwhitespace.md)

Copyright ??2010
