---
title: "A Type Lattice Approach to Binary Format Specification"
date: 2019-12-27T16:14:18+03:00
draft: true
---

# Introduction

Kaitai Struct is like JSON Schema for binary formats. But what is the [CUE](https://cuelang.org/) of binary formats?

In this post I will describe a language for declarative specification of binary protocol formats. The language allows compilation into encoding/decoding code.

```
OnionRequest2: [
    0x82 & u1,
    nonce: Nonce,
    public_key: PublicKey,
    payload: bytes[_],
    onion_return: bytes[118],
],
```

# The theory

The language for binary data specification is based on the same concept that powers the CUE language: a type lattice.

## The type lattice

The CUE theory is based on two ideas:

- Values are types and types are values
- Subtyping relations between types form a [lattice](https://en.wikipedia.org/wiki/Lattice_(order)).

Defining the subtyping relation for primitive types is staighforward:

$$\bot < \langle\text{value of u1}\rangle < \mathrm{u1} < \mathrm{int} < \mathrm{number} < \top$$

In type theory, compound data types are known as *product types*. Product types are defined with a set of *projections*, which are functions that extract a value from a data structure. For example, the type \\(T = \\{a: \mathrm{int}, b: \mathrm{string}\\}\\) is has two projections: the first one is \\(a: T \to \mathrm{int}\\) and the second one is \\(b: T \to \mathrm{string}\\).

Let's write a set of all projections of a type \\(T\\) as \\(\Pi(T)\\). Also, we consider projections that correspond to the same field name to be equivalent.

Now we can define the subtype relation for product types. A type \\(S\\) is a subtype of the \\(T\\), denoted \\(S \leq T\\), if the following holds:

$$\Pi(S) \supseteq \Pi(T)$$

$$\forall \pi \in \Pi(T): \pi(S) \leq \pi(T)$$

**Note:** if you consider a product type to be a function from names to values, you'll find out that the subtyping relation for product types is the same as the subtyping relation for functions.

The subtyping relation allows us to introduce *type intersection* and *type union* lattice operations. The type intersection of types A and B, denoted \\(A \land B\\), is the greatest lower bound with regard to subtyping. This means that \\(A \land B\\) is a subtype of both \\(A\\) and \\(B\\), and every \\(T\\) that is a subtype of both \\(A\\) and \\(B\\) is a subtype of \\(A \land B\\).

Let's \\(\times\\) be product type concatenation: \\(\\{a: T_1\\}\times\\{b: T_2\\} = \\{a: T_1, b: T_2\\}\\). We can write a handy algebraic relation with \\(\land\\):

**Proposition:** $$(\\{a: r\\}\times R)\land(\\{a: t\\}\times T) = \\{a: r \land t\\}\times (R\land T)$$

\\( \\{\pi_a\\} \cup \Pi(R) \\) and \\(\\{\pi_a\\} \cup \Pi(T) \\) are both subsets of \\( \\{\pi_a\\} \cup \Pi(R\land T) \\). The superset is minimal because \\(\Pi(R\land T) \\) is minimal and \\(\pi_a\\) is required by the first subset subset relation condition.

The type \\(r\land t\\) satisfies the second subset relation condition.

This algebraic relation allows to reduce an intersection of a composite type into intersections of primitive types. One can say that the \\(\land\\) operation is closed within the product types.


TODO

## A lattice view on data encoding and decoding

A *unit type* is a type which has exactly one value. TODO

Let \\(repr(R)\\) be a type of all values which binary representation is \\(R\\). Then one can write an equation:

$$repr(R) \land T = V$$

*Parsing* is the process of finding value \\(V\\) such that the equation holds for a given representation \\(R\\). Dually, value generation is finding \\(R\\) such that the equation holds.

## Refinement types

$$\\{x: \mathrm{number} \mid x \leqslant 42\\}$$

# Implementation

[Razbor](https://github.com/suhr/razbor)