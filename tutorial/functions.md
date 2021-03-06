# Section 1: Functions

## Hello World

Let's start with a simple "hello world" application to illustrate the HaPyLi
syntax.

[File: hello.hpl](./hello.hpl)

```hapyli
import "stdlib/base.hpl"

def main() = (print-string "Hello, world!")

; Program comments are denoted by the semi-colon character,
; just as in LISP.
```

All programs must define a `main` function (lower case) with zero parameters.
The body of any function consists of just a single expression. In the above
example, `print-string` is an imported function of one parameter.

Function names may contain any of the following characters:

- `~ ! @ # $ % ^ & * \ - _ + \ | : , < . > / ?`
- any digit
- any letter

## Power

The next example defines a recursive function which accepts two arguments `x`
and `y` and returns the value of `x` raised to the power of `y`. (Yes, this is a
bad example because `power` goes into an infinite loop if `y` is less than 1).
Notice that there are no commas between function parameters.

[File: power1.hpl](./power1.hpl)

```hapyli
import "stdlib/base.hpl"

def power(x y) =
    (if (== y 1)
        x
        (* x (power x (- y 1))))

def main() = (print-number (power 2 10))
```

## Local Variables / Let Forms

Any function can contain one let-form before the main body to define local
variables.

[File: power2.hpl](./power2.hpl)

```hapyli
import "stdlib/base.hpl"

def power(x y) =
    (if (== y 1)
        x
        (* x (power x (- y 1))))

def main() =
    let
        x = (read-number)
        y = (read-number)
    in
        (print-number (power x y))
```

## Function Overloading

HaPyLi does support a limited form of function overloading. Two functions are
considered unique if they have either different names or if they accept
different numbers of parameters.

[File: params.hpl](./params.hpl)

```hapyli
import "stdlib/base.hpl"

def f (a b) = (print-string "Two Parameters")

def f (a b c) = (print-string "Three Parameters")

def main() = (f 1 2 3)
```

## Inline Functions

Due to the nature of the Whitespace virtual machine, ordinary function calls are
extremely inefficient. For each `call` instruction in your program, Whitespace
will search through every instruction in the entire program to find the
appropriate label to jump to. To avoid this overhead, you may define functions
to be inlined. Calls to inlined functions do not emit a `call` instruction, but
are instead replaced by a copy of the entire function's body. The only
restriction is that inlined functions cannot be recursive.

I don't know of any method by which to determine whether a function should be
inlined or not. Inlining a function will definitely increase the compiled
program's size, so it may either increase or decrease overall performance.

[File: inline.hpl](./inline.hpl)

```hapyli
import "stdlib/base.hpl"

; Because "sum" is marked as inline, calling it
; is no different than calling '+' directly.
; They will compile exactly the same.

def inline sum (a b) = (+ a b)

def main () = (print-number (sum 1 2))
```

## Function Parameters and Return Values

Arguments to a function are always passed by value and every function returns
exactly one value. Furthermore, a function's parameters and local variables are
read-only. The values stored within them cannot be changed during the function's
execution. These limitations can be overcome, however, by defining global
variables or allocating arrays on heap memory and passing/returning pointers to
these arrays. These techniques are discussed later.

## Tutorial

1. [Functions](./functions.md)
2. [Expressions](./expressions.md)
3. [Variables and the Heap](./variablesandtheheap.md)
4. [Embedding Whitespace](./embeddingwhitespace.md)

Copyright ??2010
