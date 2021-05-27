---
layout: post
title: "How I learnt miniKanren"
author: Bharathi Ramana Joshi
categories: pl
tags: [miniKanren]
image:
---

I've spent quite a lot of time in the last semester working with
[miniKanren](http://minikanren.org/) in an attempt to implement at least a
sizeable subset of it metacircularly. The result of these efforts is
metaMicroKanren (mmKanren for short), which is
[microKanren](http://webyrd.net/scheme-2013/papers/HemannMuKanren2013.pdf) with
two extra forms for recursive lexical bindings (since piggybacking onto scheme's
`letrec` is not possible when the host language is miniKanren). This post goes
over what I feel are the resources that helped me gain enough of an
understanding to achieve this.

1. *The Reasoned Schemer, 2nd Edition* : this covers not only all the basics of
   writing programs in miniKanren, but also sheds light on good practises and
   idioms for relational programming. This is *the* starting point for anyone
   wanting to learn relational programming in miniKanren. Also, I specifically
   say 2nd edition, because I initially read the 1st edition but later found out
   that much had changed since the 1st edition and had to go through parts of
   2nd edition again. Please don't make the same mistake I did, just read the
   2nd edition to begin with!
2. [miniKanren, live and untagged: quine generation via relational
   interpreters](https://doi.org/10.1145/2661103.2661105) : this demonstrates
   how to implement an interpreter for a subset of Scheme in miniKanren. Do not
   be deceived by how small the size of the final interpreter is! There are
   extremely subtle and highly non-trivial choices made, to truly appreciate
   which one must try to rewrite the interpreter without referring to the paper.
3. [A unified approach to solving seven programming
   problems](https://doi.org/10.1145/3110252) : this is yet another paper
   demonstrating advanced techniques of relational programming using miniKanren.
   In particular, the 7th challenge, whose solution uses a Scheme interpreter
   inside a Scheme interpreter in miniKanren (i.e. a tower of interpreters
   interpreting one another) is quite taxing to understand, let alone solve.
4. [$\mu$Kanren : A Minimal Functional Core for Relational
   Programming](http://webyrd.net/scheme-2013/papers/HemannMuKanren2013.pdf) :
   the previous resources show how to use miniKanren, but this one shows how to
   implement a subset of miniKanren (while managing to capture the core essence
   of how it works). Once again, do not be deceived by how small the interpreter
   is - just "39" lines of Scheme. I say 39 in quotes, because the authors of
   this paper do not consider an important component (reification to be
   specific) to be part of $\mu$Kanren, rather they say it is part of the user
   interface. However, without this component the entire $\mu$Kanren system is
   quite unsuable, so I'm not sure if I agree with such a philosophy.
   Furthermore, just those 39 lines of Scheme code came out to be around ~300
   lines of dense miniKanren code when I implemented mmKanren, so it is quite
   important to understand why/what each of those 39 lines are the way they
   are/doing to gain a meaningful understanding of $\mu$Kanren, and miniKanren
   itself at large. I had to write down all of $\mu$Kanren onto an A4 size paper
   and contemplate on it for hours together to understand it enough to able to
   implement it metacircularly.

One more paper I had to understand for implementing mmKanren, but isn't
necessarily helpful to learn miniKanren is [Definitional Interpreters for
Higher-Order Programming
Languages](https://link.springer.com/article/10.1023/A:1010027404223). This
paper demonstrates how to implement a higher-order language using a first-order
langauge (e.g. implementing Scheme in C). The technique used to achieve this is
named, quite aptly, "defunctionalization". This was important to understand
because Scheme uses higher-order functions in a variety of ways to implement
$\mu$Kanren.  However, since miniKanren lacks procedures altogether,
defunctionalization must be applied for each particular kind of use of
procedures.

All in all, mmKanren taught me a ton about implementing interpreters, relational
programming and doing research in programming languages. Of course, it was super
fun!
