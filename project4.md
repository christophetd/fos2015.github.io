---
title: Project 4 - STLC Extensions
layout: default
start: 28 Oct 2015, 00:00 (Europe/Zurich)
index: 7
---

# Project 4: STLC Extensions

**Hand in:** 03 Nov 2015, 23:59 (Europe/Zurich)<br/>
<span class="very-important">You only have one week to complete this assignment!</span>

**Project template:** [4-extensions.zip](projects/4-extensions.zip)

The project skeleton is the same one you started with for the last assignment. You should continue working on
your project, and the final submission should contain all extensions, both from this assignment
and from the last assignment. In this assignment, you will extend your simply typed lambda
calculus project from exercise 3 with sum types and recursive functions.

## Sum Type

Sum types are handy when one needs to handle heterogeneous collections of values. Imagine you
have a data type like a tree, where each element can be a leaf (containing a value) or an inner
node (containing two references to subtrees). A sum type is a type which draws its values from
exactly two other types. In other words, values of a sum type T1 + T2 can be either of type T1
or of type T2. We introduce now syntactic rules for sum types, together with evaluation and
typing rules.

    t ::= ...                                                terms
        | "inl" t "as" T                                     inject left
        | "inr" t "as" T                                     inject right
        | "case" t "of" "inl" x "=>" t "|" "inr" x "=>" t    case

    v ::= ...                                                values
        | "inl" v "as" T
        | "inr" v "as" T

    T ::= ...                                                types
        | T "+" T                                            sum type (right assoc.)


We create values of a sum type by injecting a value in the left or right "part". To deconstruct
such values, we use a case expression, similar to Scala’s match construct.

Sum types are right associative and `+` has the same precedence as `*`.

Evaluation rules for sum type extension:

    case (inl v0) of inl x1 => t1 | inr x2 => t2  →  [x1 → v0] t1

    case (inr v0) of inl x1 => t1 | inr x2 => t2  →  [x2 → v0] t2

                                       t   →   t'
    --------------------------------------------------------------------------------
    case t of inl x1 => t1 | inr x2 => t2  →  case t' of inl x1 => t1 | inr x2 => t2


        t  →  t'
    ----------------
    inl t  →  inl t'

        t  →  t'
    ----------------
    inr t  →  inr t'

And lastly new typing rules:

    Γ ⊢ t0: T1 + T2    Γ, x1: T1 ⊢ t1: T    Γ, x2: T2 ⊢ t2: T
    ---------------------------------------------------------
         Γ ⊢ case t0 of inl x1 => t1 | inr x2 => t2: T


              Γ ⊢ t1: T1
    -------------------------------
    Γ ⊢ inl t1 as T1 + T2 : T1 + T2

              Γ ⊢ t1: T2
    -------------------------------
    Γ ⊢ inr t1 as T1 + T2 : T1 + T2

Evaluation and typing rules are pretty straightforward, the only thing a bit out of
ordinary is the type adornment for inl and inr. This is necessary because the two
constructors can’t possibly know what is the type of the other component of the sum
type. By giving it explicitly, the type checker is able to proceed. Alternatives to
this decision will be explored in other exercises, when we know about type inference.

## The fix operator

To bring back the fun into the game, we need some way to write non terminating programs
(or at least, looping programs). Since types arrived, we were unable
to type the fixpoint operator, and indeed there’s a theorem which states that all well
typed terms in simply typed lambda calculus evaluate to a normal form. The only
alternative is to add it into the language.

    t ::= ...        terms
        | "fix" t    fixed point of t

New reduction rules:

    fix (λx: T1. t2)  →  [x → fix (λx: T1. t2)] t2

        t  →  t'
    ----------------
    fix t  →  fix t'

New typing rules:

    Γ ⊢ t1: T1 → T1
    ---------------
    Γ ⊢ fix t1: T1

In order to make the fixpoint operator easier to use, add the following derived form:

    letrec x: T1 = t1 in t2  ⇒  let x = fix (λx: T1. t1) in t2

Don’t forget that we implement let as a derived form as well, so the actual desugaring
is a bit more complex.

## What you need to do

1. Implement `Term` parser that recognizes this language, using the combinator library.
The parser must produce abstract syntax trees defined in `Terms.scala`.

1. Implement `reduce` method which performs one step of the evaluation, according to the
rules of call-by-value reducer (small step). If none of the rules apply, it should throw
`NoRuleApplies` exception containing corresponding irreducible term.

1. Implement `typeof` method which given a term finds its type. In the case of failure,
it should throw `TypeError` exception containing the ill-typed term. The ill-typed term
is the leftmost innermost term that doesn't typecheck, i.e. that doesn't have any typing
rules that apply to it.

**Hint**: re-using code from previous assignments might be a good idea.

## Development & debugging

Since the project skeleton is the same as in the previous assignment, everything from
the "Development & debugging" section in [the previous project description](project3.html)
applies to this project as well.