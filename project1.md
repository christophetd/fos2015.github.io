---
title: Project 1 - The NB Language
layout: default
start: 16 Sep 2015, 00:00
index: 4
---

# Project 1: The NB Language

**Hand in:** 29 Sep 2015, 23:59 UTC

**Project template:** [1-arithmetic.zip](projects/1-arithmetic.zip)

The cryptic acronym stands for Numbers and Booleans and comes from the course book.
This simple language is defined in Chapter 3 of the the TAPL book.

    t  ::= "true"                   terms
         | "false"
         | "if" t "then" t "else" t
         | numericLiteral
         | "succ" t
         | "pred" t
         | "iszero" t

    v  ::= "true"                   values
         | "false"
         | nv

    nv ::= 0                        numeric values
         | "succ" nv

This language has three syntactic forms: terms, which is the most general form, and two
kinds of values: numeric values, and boolean values. We have extended the syntax by
allowing numeric literals. They are syntactic sugar and have to be transformed during
parsing to their equivalent value `succ succ .. 0`. The language is completely defined by
the production `t`, for terms. Values are a subset of terms, and for simplicity they are
defined using a BNF notation, but they need not be parsed as such.

## Small step semantics

The evaluation rules are as follows.

             if true then t1 else t2 → t1

             if false then t1 else t2 → t2

                  isZero zero → true

                isZero succ nv → false

                    pred zero → zero

                   pred succ nv → nv

                        t1 → t1'
     ——————————————————————————————————————————————
     if t1 then t2 else t3 → if t1' then t2 else t3

                         t → t'
                  ————————————————————
                  isZero t → isZero t'

                         t → t'
                    ————————————————
                    pred t → pred t'

                         t → t'
                    ————————————————
                    succ t → succ t'

## Big step semantics

The other style of operational semantics commonly in use is called big step sematics.
Instead of defining evaluation in terms of a single step reduction, it formulates the
notion of a term that evaluates to a final value, written "t ⇓ v". Here is how the big
step evaluation rules would look for this language:


               v ⇓ v            (B-VALUE)

      t1 ⇓ true     t2 ⇓ v2
    ——————————————————————————  (B-IFTRUE)
    if t1 then t2 else t3 ⇓ v2

      t1 ⇓ false     t3 ⇓ v3
    ——————————————————————————  (B-IFFALSE)
    if t1 then t2 else t3 ⇓ v3

             t1 ⇓ nv1
        ——————————————————      (B-SUCC)
        succ t1 ⇓ succ nv1

            t1 ⇓ 0
          ———————————           (B-PREDZERO)
          pred t1 ⇓ 0

         t1 ⇓ succ nv1
         —————————————          (B-PREDSUCC)
         pred t1 ⇓ nv1

            t1 ⇓ 0
        ————————————————        (B-ISZEROZERO)
        iszero t1 ⇓ true

         t1 ⇓ succ nv1
       —————————————————        (B-ISZEROSUCC)
       iszero t1 ⇓ false

## What you have to do

1. Implement `term` parser that recognizes this language, using the combinator library.
   The parser must produce abstract syntax trees defined in `Terms.scala`.

1. Implement `reduce` method which performs one step of the evaluation, according to the rules
   of the small step semantics. If none of the rules apply it should throw `NoReductionPossible`
   exception containing corresponding irreducible term.

1. Implement `eval` method which implements a big step evaluator (one which evaluates a term
   down to a value, or it gets stuck when no rule applies). This method should implement
   the big step semantics defined above, and not call reduce. If evaluation is not possible
   it should throw `TermIsStuck` exception containing corresponding stuck term.

All of these are given with stubbed `???` body. `???` is used to indicated unimplemented
parts of code in Scala.

## Development & debugging

We provide a simple runnner for your application that lets you quickly debug your current
implementation. When you implement `term`, `reduce` and `eval` implementations it should
produce results like:

**Example 1**:

input:

    if iszero pred pred 2 then if iszero 0 then true else false else false

output:

    If(IsZero(Pred(Pred(Succ(Succ(Zero))))),If(IsZero(Zero),True,False),False)
    If(IsZero(Pred(Succ(Zero))),If(IsZero(Zero),True,False),False)
    If(IsZero(Zero),If(IsZero(Zero),True,False),False)
    If(True,If(IsZero(Zero),True,False),False)
    If(IsZero(Zero),True,False)
    If(True,True,False)
    True
    Big step: True

**Example 2**:

input:

    pred succ succ succ false

output:

    Pred(Succ(Succ(Succ(False))))
    Big step: Stuck term: Succ(False)
