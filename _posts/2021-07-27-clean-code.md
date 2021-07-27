---
layout: post
title: "Applying Clean Code"
author: Bharathi Ramana Joshi
categories: programming, tips
tags: [programming]
image:
---

I've recently been reading Uncle Bob's *Clean Code* (my
[notes](https://github.com/iambrj/notes/blob/master/clean-code.md)), and found
myself applying some of the guidelines in the book to the Scheme compiler I've
been hacking on (source code
[here](https://github.com/iambrj/imin/blob/master/compiler.rkt)).  Specifically,
I had this chunk of code that during register allocation translates a color to a
memory location (I use [Graph-coloring
allocation](https://en.wikipedia.org/wiki/Register_allocation#Graph-coloring_allocation)):

```racket
(define (var->mem var->color color->reg)
  (let* ([maxcolor (apply max (dict-keys color->reg))]
         [spilled-vars (filter (lambda (v)
                                 (and (Var? v)
                                      (> (dict-ref var->color v)
                                         maxcolor)))
                               (dict-keys var->color))]
         [spilled-vecs (filter (lambda (v)
                                 (and (Vector? v)
                                      (> (dict-ref var->color v)
                                         maxcolor)))
                               (dict-keys var->color))]
         [var->stk (foldr (lambda (v i a)
                            `((,v . ,i) . ,a))
                          '()
                          spilled-vars
                          (range 1 (add1 (length spilled-vars))))]
         [vec->stk (foldr (lambda (v i a)
                            `((,v . ,i) . ,a))
                          '()
                          spilled-vecs
                          (range 1 (add1 (length spilled-vecs))))])
    (values (dict-map var->color (lambda (v c)
                                   `(,v . ,(if (Vector? v)
                                             (if (member v spilled-vecs)
                                               (Deref 'r15 (* 8 (dict-ref vec->stk v)))
                                               (dict-ref color->reg c))
                                             (if (member v spilled-vars)
                                               (Deref 'rbp (* 8 (dict-ref var->stk v)))
                                               (dict-ref color->reg c))))))
            (length (dict-values var->stk))
            (length (dict-values vec->stk)))))
```

And it's too ugly. Specifically,

1. It's too long: a function should be, in the worst of cases, 20 lines.
2. It duplicates code for the way variables are handled and vectors are handled,
   which brings me to my next point;
3. The variable names are misleading: it's unclear how a `var` is different from
   a `vec`. Of course, I wrote the code so I know `var` is a non-vector typed
   variable and `vec` is a vector typed variable. But the names I've chosen do
   not reflect this, leaving the reader rather confused.
4. It lacks *implicity*: i.e. the chunk of code implies very little, if anything
   at all, about the context in which it exists.
5. It works at different levels of abstraction. This is also indicated by the
   multiple indents used.
6. A reason that cannot be inferred from just the above chunk, it takes an
   extra argument that can be avoided (`color->reg`). This is constructed from a
   global constant just before calling `var->mem`, so we might as well move the
   construction into `var->mem`.
7. It has magic constants : it's unclear what `r11` or `rbp` are there for.

So let's refactor this chunk of code into *clean code*. Let's first start with
the most easily resolvable issue, the last one:

```racket
(define shadow-stk-reg 'r15)
(define stk-reg 'rbp)
...
(define (var->mem var->color color->reg)
...
                                   `(,v . ,(if (Vector? v)
                                             (if (member v spilled-vecs)
                                               (Deref shadow-stk-reg (* 8 (dict-ref vec->stk v)))
                                               (dict-ref color->reg c))
                                             (if (member v spilled-vars)
                                               (Deref stk-reg (* 8 (dict-ref var->stk v)))
                                               (dict-ref color->reg c))))))
...
```

This immediately increases the readability of the code --- readers familiar with
garbage collection techniques can infer that there are two stacks being used,
one for stuff allocated on the stack (`stk-reg`) and one for stuff allocated on
the heap (`shadow-stk-reg`). This also increases the *implicity* of the code: it
tells us that the compiler is doing garbage collection in the first place!

Now, let's move on to the next most easiest to solve issue: 6. We can fix this
by adding another binding into the `let*` that constructs
   `color->reg`:

```racket
(define (var->mem var->color color->reg)
  (let* ([color->reg (map (lambda (r)
                            (cons (index-of usable-registers r) r))
                          usable-registers)]
...
```

There! Now we've decreased the cognitive burden of remembering what the second
argument to `var->mem` should be to whoever wants to use it.

Now let's try to tackle the code duplication (actually, the solution I came up
with for this also fixes 5 to an extent). Firstly we notice that the function is
working at multiple levels of abstractions: it is manipulating lists, which are
significantly lower level of detail. This hints at a possible abstraction: hide
away all the list manipulation code into another function, and call that
function from `var->mem`. Let's first deal with calculating spilled variables:

```racket
(define (spill maxcolor var->color)
  (filter (lambda (v)
            (> (dict-ref var->color v) maxcolor))
          (dict-keys var->color)))
```

Now this is all well and good, but this still doesn't help with eliminating
duplication. For that, we employ yet another function to build a single
dictionary mapping variables (both vector and non-vector typed) to their stack
locations (using the spilled variable list we get from `spill`):

```racket
(define (spill->stk spills)
  (let ([vec-spills (filter Vector? spills)]
        [nonvec-spills (filter Var? spills)])
    (foldr (lambda (v i a)
           (let ([stk (if (Var? v) stk-reg shadow-stk-reg)]
                 [offset (if (Var? v)
                           (index-of nonvec-spills v)
                           (index-of vec-spills v))])
            `((,v . ,(Deref stk (* 8 offset))) . ,a)))
         '()
         spills
         (range 1 (add1 (length spills))))))
```

We can also eliminate some more duplication by yet another helper:

```racket
(define (stack-size pred spills)
  (length (filter pred spills)))
```

Now we rewrite `var->mem` using the above auxiliary functions:

```racket
(define (var->mem var->color)
  (let* ([color->reg (map (lambda (r)
                            (cons (index-of usable-registers r) r))
                          usable-registers)]
         [maxcolor (apply max (dict-keys color->reg))]
         [spills (spill maxcolor var->color)]
         [spilled->stk (spill->stk spills)])
    (values (dict-map var->color (lambda (v c)
                                   `(,v . ,(if (member v spills)
                                             (dict-ref spilled->stk v)
                                             (dict-ref color->reg c)))))
            (stack-size Var? spills)
            (stack-size Vector? spills))))
```

This looks so much more beautiful and readable! Interestingly, even after all
this refactoring the total number of lines occupied by the entire chunk of code
(with auxiliary functions) increases by *just 2 lines*. The final piece of code
is:

```racket
(define (var->mem var->color)
  (let* ([color->reg (map (lambda (r)
                            (cons (index-of usable-registers r) r))
                          usable-registers)]
         [maxcolor (apply max (dict-keys color->reg))]
         [spills (spill maxcolor var->color)]
         [spilled->stk (spill->stk spills)])
    (values (dict-map var->color (lambda (v c)
                                   `(,v . ,(if (member v spills)
                                             (dict-ref spilled->stk v)
                                             (dict-ref color->reg c)))))
            (stack-size Var? spills)
            (stack-size Vector? spills))))

(define (spill maxcolor var->color)
  (filter (lambda (v)
            (> (dict-ref var->color v) maxcolor))
          (dict-keys var->color)))

(define (spill->stk spills)
  (let ([vec-spills (filter Vector? spills)]
        [nonvec-spills (filter Var? spills)])
    (foldr (lambda (v i a)
           (let ([stk (if (Var? v) stk-reg shadow-stk-reg)]
                 [offset (if (Var? v)
                           (index-of nonvec-spills v)
                           (index-of vec-spills v))])
            `((,v . ,(Deref stk (* 8 offset))) . ,a)))
         '()
         spills
         (range 1 (add1 (length spills))))))

(define (stack-size pred spills)
  (length (filter pred spills)))
```

I've only read the first 4 chapters of the book, yet it was immensely helpful as
can be seen above. Needless to say, I eagerly look forward to working through
the rest of the book.
