---
layout: post
title: "The mysterious Y combinator"
author: Bharathi Ramana Joshi
categories: pl
tags: [lambda-calculus]
image:
---

One of the fundamental limitations of the Lambda Calculus is that the functions
do not have names - they are *anonymous*. As a direct consequence, there is no
trivial way of defining recursive functions - functions that call themselves -
as functions do not have any names to be called with! However, recursion is a
very powerful idea - it is used in algorithms (e.g. Divide and Conquer, Dynamic
Programming, Backtracking all make use of recursion), definitions of language
grammars (and thus, to some extent, their implementations), creating data
structures (fun exercise: try to define a binary tree recursively). Therefore,
sooner or later, the need for recursion arises.

In order to implement recursion in the Lambda Calculus, we first need to
understand the concept of a fixed point. Here's a fun exercise you can try -
take a scientific calculator which is at the very least capable of calculating
the elementary trigonometric functions. Enter `cos(0)`. As expected, the
calculator outputs `1`. Now, enter `cos(1)`. It should output something around
`0.5403`. Now enter `cos(0.5403)` (tip: There is a `Ans` button on most
scientific calculators. Use that instead of typing out the answer explicitly
each time). If you keep repeating this process for around 20 times, you'll
notice that all subsequent results are around `0.7389`. On my calculator, after
around 58 iterations, this value stops changing at `0.7390851332`. On further
application, `cos(0.7390851332)` keeps giving `0.7390851332`. This is called a
"Fixed point" of the `cos` function. Generalizing, fixed points are those values
for which the result of a function application is the same value, i.e. if `f` is
any function and `x` is `f`'s fixed point, then `f(x) = x` (Fun exercise: try
finding the fixed point of the function `x^2 + 2x + 1`).

## Okay, but what does it have to do with recursion?

Before implementing recursion using fixed points, I'd like to talk about the Y
Combinator. The Y Combinator is simply a function that takes as an argument
another function and returns its fixed points. For example, if we were to pass
the Y Combinator the cosine function, we would get `0.7390851332`. If we were to
pass it `x^2`, we would get `0` (note that it could have also returned `1`,
which is also a valid fixed point. The actual value returned depends on the way
the Y combinator is implemented).

Now, say we wanted to implement the recursive version of the factorial function
as follows

```
fact(n) = 1, n == 0
     n * fact(n - 1), n > 0
```

## Named functions? Didn't you say that was exactly what we were trying to avoid?

Yes and no. What we are trying to do is implement a recursive function without
explicitly calling it. Observe that this is equivalent to using only those named
functions that do not call themselves! Thus, we can use named functions as long
as they do not call themselves.

## Using the Y combinator

We first construct another function `g`, whose fixed point is `fact`. Thus, when
we pass `g` to the Y combinator, we get back `fact`!

How to do that you ask? Define `g` as follows

```
g(f(n)) = 1, n == 0
          n * f(n - 1), n > 0
```

This looks suspiciously similar to the definition of the `fact` function -
except we don't use `g` while defining `g`, thus making it viable to implement
`g` in the Lambda Calculus! Also, observe that when `fact` is passed to `g`, we
get back `g` - thus making `fact` a fixed point of `g`! In fact this can be
extended to any recursive function (fun exercise: try defining an appropriate
`g` for the Fibonacci function).

## But how to implement the Y combinator?

So far, the post treats the Y combinator as some sort of a black box which takes
a function and returns its fixed points. To understand how to actually implement
it, consider the following definition of `s` (for self-application)

```
s x = g (x x)
```

for some function `g`, i.e. `s` takes a function `x` applies it onto itself and
then applies `g` on the result. Now observe what happens if we feed `s` to
itself

```
s s = g (s s)
```

Surprise! `s` is a fixed point of `g`. As `g` was an arbitrary function, we've
just created a way to generate fixed points for *any* function! Polishing this
further, we have

```
Y g =
    let s x =
        g (x x) in (s s)
```

which is translated into racket as

```lisp
(define (Y g)
    (let ([s (lambda (x)
                (g (x x)))])
        (s s)))
```

## A small catch

If you try to run the above racket procedure, you'll notice that it never
returns. This is because of the order in whicn racket performs
[evaluation](https://docs.racket-lang.org/reference/eval-model.html). Racket,
like most schemes, is *applicative order*, which means that the arguments to a
function are evaluated before the function is applied. Therefore, `(s s)`
evaluates to `(g (s s))`, which evaluates to `(g (g (s s)))` and so on. To get
around this, we make the assumption that the fixed point of `g` is always a
procedure of a single argument. Note that we do not lose any expressivity
because of currying. Thus, the above definition is now massaged into

```lisp
(define (Y g)
    (let ([s (lambda (x)
                (lambda (n))
                    ((g (x x)) n))])
        (s s)))
```

If we were using a lazy language like Haskell, we could have avoided all this
and defined the Y combinator as

```haskell
y g = y (g x)
```
