---
layout: post
title: Functors
comments: True
---

*Every sufficiently good analogy is yearning to become a functor* - [John Baez](https://en.wikipedia.org/wiki/John_C._Baez)

A functor is a,
- A Typeclass in `Haskell`
- A structure-preserving mapping between two categories in `Category Theory`

### Functor is a Typeclass

Typeclasses give the power to extend capabilities of an existing type without touching the old code. 
Functors carry the capability of transitioning from one type to another. 
So an instance of `Functor` for a particular higher kinded type, can be used to utilize this capability.
This capability is abstracted in the `Functor` class as a function named `fmap`.

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

As I mentioned a `Functor` can append the power of type transition for `constructs` that hides an arbitrary type. 
These `constructs` have different semantics and solve different problems.

e.g: 
   - `Maybe` construct is used to avoid errors, or to transform a partial computations in to a full computation.
   - `List` construct is used to process a sequence of values of some type, or to convert a non-deterministic computation into a list of possible values.
   - `State` construct is used to pass around a state and accumulate mutations from a computation to another computation.
   - `IO` construct is used to perform IO related computations.
    
In all these cases `Functor` gives the capability of transitioning from `Construct` of type `a`  to a `Construct` of type `b`.
All these `Constructs` are higher kinded types which allows to abstract over a type. 

Having a separate `Functor` typeclass which abstracts this capability gives the freedom to mentally group together things that share this common structure, which is very powerful.

To be a true `Functor` and avoid any surprises, an instance of `Functor` typeclass should follow these laws.
  
 - `fmap id = id`
 - `fmap (g . f) = fmap g . fmap f`

  
### Functor is a Structure preserving mapping

A functor `F:C -> D` from a category `C` to a category `D` consists of some data that satisfies certain properties.

- An object `F(x)` in `D` for every object `x` in `C`
- A morphism `F(x)⟶F(y)` in `D` there is a morphism `x⟶y` in `C`

A functor consists of two mappings, 
- one on objects 
- one on morphisms

Therefore a functor `F: C -> D` is a pair `F = (Fob, Fmor)` where `Fob` maps the objects of `C` to objects of `D` and 
`Fmor` maps morphisms `f: x -> y` of `C` to morphisms of type `Fob x -> Fob y` in `D` and both of these mappings are represented by the same notation `F:C -> D`

An example of a functor is the `List` type constructor in a programing language which takes a set of elements and returns a `List` from those elements.

- The object part of the functor maps the set A to the set of lists over A, i.e., combinations of the form `[x1, . . . , xn]` where each `xi`
is an element of A. Normally when a set of elements is passed to the `List` type constructor, it is expected to get a `List` of all those elements. But any implementation is possible. 
- The morphism part of the functor maps a function `f: A -> B` to the function normally written as `fmap` in `Haskell` which sends a list `[x1, . . . , xn]` to `[f(x1), . . . , f(xn)]`.
In the categorical notation, the function `fmap` can be written as `List A -> List B`

- `F` follows composition, i.e. `F(g∘f) = F(g) ∘ F(f)` in `D` for all composable `g` and `f` in `C`
- `F` sends identities to identities, i.e. `F(id x) = id F(x)` for all objects `x` in `C`