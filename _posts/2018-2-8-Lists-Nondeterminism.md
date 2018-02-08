---
layout: post
title: Lists and Non-determinism
comments: true
---

The term *non-deterministic computation* can have different meanings
 - This kind of non-determinism when using multiple threads of execution where the output could be determined by the order of execution at runtime.
 - The non determinism in the context of zero or more possible outputs.
 
Although Lists are used for processing data sequences, they can also be used for turning non deterministic computations into pure functions by returning lists of all possible outputs.
If a function can return different results, then it can return them all at once. Therefore a non deterministic function can be thought of as a function that returns a list of results.

e.g: `[a]` is a nondeterministic computation of an `a`. Thus the list `[1,2]`, represents the result of a computation that non deterministically return 1 or 2 or nothing

This is really useful in a non-strict language like Haskell.
In Haskell if you need just the first value then, it is possible just to take the head of the list without evaluating the tail. 
In order to get a random value, a random number generator can be used to pick an element from the list at a random index. 
This non strictness can be used to return an infinite list also.


```haskell
data List a = Nil | Cons a (List a)
```
Lets represent `List a` by `f(x)` and `Nil` by `1`. `List` is a sum type of `Nil` and `Cons`. `Cons` is a product type of `a` and `List a`.

 `f(x) = 1 + x f(x)` is recursive since `f(x)` is using itself in its definition.

Lets expand this recursion a bit
 
 `1 + x f(x)`
 
 `1 + x (1 + x f(x))` 
 
 `1 + x + x^2 f(x)`
  
 `1 + x + x^2 (1 + x f(x))`
  
 `1 + x + x^2 + x^3 f(x)`
  
 `1 + x + x^2 + x^3 (1 + x f(x))`
  
 `1 + x + x^2 + x^3 + x^4 ...`
  
`x^2`, `x^3` can be thought of as product types of `x` in the same way List `[1,2,3]` can be thought of as a product type of `1`, `2` and `3`. 

 `1 + x + x^2 + x^3 + x^4 ...` is the sum type of `1`, `x`, `x^2`, `x^3` .etc
 
 Its same as saying *a sum type of `empty list`, `single element`, `two element list`, `three element list` .etc*  
 
`List a` represents all `zero`, `one`, `two`, `three` .etc element scenarios in which any of them can be a non deterministic representation of a particular computation. 
 
In practice, areas like natural language processing, computer games use these kind of all possible scenario representations to handle non-determinism. 


