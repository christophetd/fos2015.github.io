---
title: Project 3 - Simply Typed Lambda Calculus
layout: default
start: 14 Oct 2015, 00:00 (Europe/Zurich)
index: 6
---

# Project 3: Simply Typed Lambda Calculus

**Hand in:** 27 Oct 2015, 23:59 (Europe/Zurich)

**Project template:** [3-typed.zip](projects/3-typed.zip)

The goal of this exercise is to familiarize yourself with the simply typed λ-calculus; your work consists of implementing a type
checker and a reducer for simply typed λ-terms. To make it more interesting, we extend λ-calculus with boolean and
integer values, as well as let and pair constructs. This time we'll fix a call-by-value strategy, so there will be only one reducer you need to write (actually,
you can reuse some of the reducer from the previous assignment.

We first introduce the syntax for typed λ-calculus without *let* and *pairs*, which we'll add later. It might be a good
idea to start implementing this language first and later add the rest. The syntax is presented here:

    t ::= "true"                      true value
        | "false"                     false value
        | "if" t "then" t "else" t    if
        | numericLit                  integer
        | "pred" t                    predecessor
        | "succ" t                    successor
        | "iszero" t                  iszero
        | x                           variable
        | "\" x ":" T "." t           abstraction
        | t t                         application (left associative)
        | "(" t ")"

    v ::= "true"
        | "false"
        | nv                          numeric value
        | "\" x ":" T "." t           abstraction value

    nv ::= "0"                        numeric values
         | "succ" nv

Note: As in the first assignment, we add syntactic sugar for numeric literals. They are desugared to the corresponding sequence of
`succ succ ... 0`, as described before.

The only new thing in the above rules is the type annotation for lambda abstractions. We see that the variable name is followed by colon and
a type. It roughly says "this function expects an argument of type `T`". In the above rules, `T` stands for types, and here's the syntax for
types:

    T ::= "Bool"                      boolean type
        | "Nat"                       numeric type
        | T "->" T                    function types  (right associative)
        | "(" T ")"

There are three kinds of types in this language: booleans, natural numbers and function types. The type constructor "→" is
right-associative -- that is, the expression `T`<sub>`1`</sub>` → T`<sub>`2`</sub>` → T`<sub>`3`</sub> stands for
`T`<sub>`1`</sub>` → (T`<sub>`2`</sub>` → T`<sub>`3`</sub>`)` [TAPL, p.100].

## Evaluation rules

Evaluation rules for this language are presented below. You may note that they already fix the evaluation strategy to call-by-value, and
that the type of an abstraction is ignored during evaluation.

We can say that evaluation of simply typed lambda terms proceeds exactly the
same as for untyped lambda terms. The operation of stripping off type annotations is called *erasure* and it is what enabled the addition of
Java generics without modifying the virtual machine.

  **Computation Rules:**

    if true then t1 else t2 → t1

    if false then t1 else t2 → t2

    iszero 0 → true

    iszero succ NV → false

    pred 0 → 0

    pred succ NV → NV

    (λx: T. t1) v2 → [x → v2] t1

  **Congruence Rules:**

                       t1 → t1'
    ----------------------------------------------
    if t1 then t2 else t3 → if t1' then t2 else t3

           t → t'
    --------------------
    iszero t → iszero t'

         t → t'
    ----------------
    succ t → succ t'

         t → t'
    ----------------
    pred t → pred t'

       t1 → t1'
    --------------
    t1 t2 → t1' t2

       t2 → t2'
    --------------
    v1 t2 → v1 t2'

## Typing rules

The typing rules of the figure given below define a typing relation, similar to the evaluation relation. However, while evaluation is a relation between terms,
typing is a relation between terms, types and contexts.

A typing rule like the first one can be read "under context gamma, term `true`
has type `Bool`". The role of the context (or environment) is to keep around a mapping between variable names and their types. It will be
used to type free variables, when they are encountered. This is illustrated by the variable rule which can be read "under context gamma,
variable `x` has type `T` provided context gamma has a binding for variable `x` to type `T`".

    Γ ⊢ true : Bool

    Γ ⊢ false : Bool

    Γ ⊢ 0 : Nat

      Γ ⊢ t : Nat
    ----------------
    Γ ⊢ pred t : Nat

      Γ ⊢ t : Nat
    ----------------
    Γ ⊢ succ t : Nat

        Γ ⊢ t : Nat
    -------------------
    Γ ⊢ iszero t : Bool

    Γ ⊢ t1: Bool  Γ ⊢ t2 : T  Γ ⊢ t3 : T
    -------------------------------------
        Γ ⊢ if t1 then t2 else t3 : T

    x : T ∈ Γ
    ---------
    Γ ⊢ x : T

      Γ, x : T1 ⊢ t : T2
    -----------------------
    Γ ⊢ λx: T1. t : T1 →T2

    Γ ⊢ t1 : T11 → T12   Γ ⊢ t2 : T11
    ---------------------------------
           Γ ⊢ t1 t2 : T12

The purpose of this type system is to prevent "bad things" from happening. So far, the only bad thing we know is stuck terms, and this type
system prevents stuck terms. In other words, a term that can be assigned a type (it type checks) is guaranteed not to get stuck. The result
of its evaluation will be a value.

## Adding *let* via desugaring

Let us now proceed to the addition of `let`:

    t ::= ...
        | "let" x ":" T "=" t "in" t

We can define `let` in terms of the existing concepts, and this has the advantage that once the translation (also called "desugaring") is done in the parser, no addition
is necessary to the type checker or to the evaluator. Such an addition is called a derived form. The language that is accepted by our parser
is called external language and the language understood by the evaluator (and type checker) is called internal language.

_Desugaring of derived forms is very convenient from a scientific point of view (so much that it's become a de facto standard in literature). However, careless desugaring in a practical compiler invites trouble. As our experience with scalac shows, performing desugarings during parsing is not a very good idea, because it: 1) confuses tools, e.g. hyperlinking in IDEs, 2) makes prettyprinting and error messages less helpful, 3) complicates compiler extensibility. In this toy project, you'll desugar in parser, because it's really simple to implement, but we would also like you to understand the associated trade-offs._

Now, without further ado, let's see the
translation of let in terms of abstraction and application:

    let x: T = t1 in t2 => (λx:T. t2) t1

## Adding *pairs* the hard way

To add pairs, we can't do the same trick, because there are no existing language features to piggyback onto, so we'll need to extends the existing syntax, evaluation and typing rules:

    t ::= ...
        | "{" t "," t "}"
        | "fst" t
        | "snd" t

    v ::= ...
        | "{" v "," v "}"

    T ::= ...
         | T "*" T                    (right associative)

The first form creates a new pair, and the other two are called projections, and extract the first and the second element of a pair. We add
a new kind of values, pair values, and a new kind of type, for the corresponding pair type. We decide that the pair type constructor
(denoted by `*`) takes precedence over the arrow constructor, so `Nat * Nat → Bool` is parsed as `(Nat * Nat) → Bool`, and that function application takes precedence over projections so `snd x y` is parsed as `snd (x y)`.
The new evaluation rules are presented below:

    fst {v1, v2} → v1

    snd {v1, v2} → v2

        t → t'
    --------------
    fst t → fst t'

        t → t'
    --------------
    snd t → snd t'

          t1 → t1'
    --------------------
    {t1, t2} → {t1', t2}

          t2 → t2'
    --------------------
    {v1, t2} → {v1, t2'}

The new typing rules are as follows:

    Γ ⊢ t1 : T1   Γ ⊢ t2 : T2
    --------------------------
      Γ ⊢ {t1, t2} : T1 * T2

    Γ ⊢ t : T1 * T2
    ---------------
    Γ ⊢ fst t : T1

    Γ ⊢ t : T1 * T2
    ---------------
    Γ ⊢ snd t : T2

## What you have to do

  1. Implement `Term` parser that recognizes this language, using the combinator library. The parser must produce abstract syntax trees defined in `Terms.scala`.

  1. Implement `reduce` method which performs one step of the evaluation, according to the rules of call-by-value reducer (small step). If none of the rules apply, it should throw `NoRuleApplies` exception containing corresponding irreducible term.

  1. Implement `typeof` method which given a term finds its type. In the case of failure, it should throw `TypeError` exception containing the ill-typed term. The ill-typed term is the leftmost innermost term that doesn't typecheck, i.e. that doesn't have any typing rules that apply to it.

## Development & debugging

We provide a simple runnner for your application that lets you quickly debug your current implementation. When you implement `Term`, `reduce` and `typeof` it should produce results like:

**Example 1**:

input:

    (\x:Nat->Bool. (\y:Nat.(x y))) (\x:Nat.(iszero x)) 0

output:

    typed: Bool
    (\x:Nat->Bool.(\y:Nat.x y)) (\x:Nat.iszero x) 0
    (\y:Nat.(\x:Nat.iszero x) y) 0
    (\x:Nat.iszero x) 0
    iszero 0
    true

**Example 2**:

input:

    (\x:Nat.x) true

output:

    parameter type mismatch: expected Nat, found Bool
    (\x:Nat.x) true

**Example 3**:

input:

    (\x:Nat.snd x) 1

output:

    pair type expected but Nat found
    snd x