---
layout: post
title: Why Type Classes?
comments: true
---

The concept of Type class is one solution for the expression problem which is a major challenge for programing languages even today.
Lets explore what this problem means.

### The Expression Problem

According to [Wikipedia](https://en.wikipedia.org/wiki/Expression_problem),

```
The goal is to define a datatype by cases, where one can 
add new cases to the datatype and new functions over the 
datatype, without recompiling existing code, and while 
retaining static type safety (e.g., no casts).
```
Lets boil this down. 

We can consider computer programs as a combination of `data` and `operations`. If we want to add new `data` or `operations` we should not touch the existing code.
This is the challenge expressed by the Expression problem.

One of the major distinctions between Functional Programing and Object Oriented Programing is, 
in functional programing, data and related operations are completely separated and in OOP data encapsulates the related operations. 

In Haskell, the basic programing style is to define your ADTs(Algebraic Data Types) at the beginning of the program. 

```haskell
module Data where

data Expression = Constant Int | Plus Expression Expression

evaluate :: Expression -> Int
evaluate (Constant i) = i
evaluate (Plus e1 e2) = result e1 + result e2 
```

`Constant` and `Plus` are data constructors for type `Expression`


```java

interface Expression {
    int evaluate;
}

class Constant extends Expression {
    private int value; 
    
    // getters and setters
    // ..
    
    public int evaluate() {
        return this.value;
    }
}

class Plus extends Expression {
    private Expression left, right;
    
    // getters and setters
    // ...
    
    public int evaluate() {
        return left.evaluate() + right.evaluate();
    } 
} 
```
`Constant` and `Plus` are separate implementations of interface `Expression`

Lets assume that a pretty printer has to be added for type `Expression`.

```haskell

module PrettyPrinter where

import Data

stringify :: Expression -> String
stringify (Constant i) = show i
stringify (Plus l r) = stringify l ++ " + " ++ stringify r
```
It seems it is easy to add a new operation like `stringify`. Let's try the same in Java. 

```java

interface Expression {
    int evaluate;
    String stringify;
}

class Constant extends Expression {
    private int value; 
    
    // ..
    
    public int evaluate() {
        return this.value;
    }
    
    public String stringify() {
        return this.value.toString();
    }
}

class Plus extends Expression {
    private Expression left, right;
    
    // ...
    
    public int evaluate() {
        return left.evaluate() + right.evaluate();
    } 
    
    public String stringify() {
        return left.stringify() + " + " + right.stringify();
    }
} 
```

It is not easy to add a new operation like `stringify`, because all the implementations of `Expression` has to be updated with the new `operation`.

Lets try to add a new data type for `Expression`

```haskell
    
module Data where

data Expression = Constant Int 
                | Plus Expression Expression 
                | Minus Expression Expression 

evaluate :: Expression -> Int
evaluate (Constant i) = i
evaluate (Plus e1 e2) = evaluate e1 + evaluate e2 
evaluate (Minus e1 e1) = evaluate e1 - evaluate e2
```
```haskell

module PrettyPrinter where

import Data

stringify :: Expression -> String
stringify (Constant i) = show i
stringify (Plus l r) = stringify l 
                        ++ " + " ++ stringify r
stringify (Minus e1 e1) = stringify l 
                           ++ " - " ++ stringify r
```

In Haskell, its not easy to add a new data type since all the functions has to be updated with the new value. 

```java
    
public class Minus extends Expression {
    private Expression left;
    private Expression right;
    
    @Override
    public String stringify() {
        return left.stringify() + " - " + right.stringify();
    }
}
```
In Java, a new type can be added without touching the existing code. 

Therefore, in a functional programing language like haskell, its easy to get `data` extensibility but not `operator` extensibility. In an OOP language like Java,
its easy to get `data` extensibility but not `operator` extensibility. 

### Can we use Visitor Pattern?

Visitor Patterns does not exactly solve the problem but it's worth checking out.

```java

interface Visitor<R> {
    R visit(Constant that);
    R visit(Plus that);
}

public abstract class Expression {
    public abstract <R> R accept(Visitor<R> v);
}

public class Stringify implements Visitor<String> {
    public String visit(Constant that) {
        return that.info.toString;
    }
    
    public String visit(Plus that) {
        return that.left.accept(this) + 
                " + " + that.right.accept(this);
    }
}

public class Constant extends Expression {
    private int info;
    
    @Override
    public <R> R accept(Visitor<R> v) {
        return v.visit(this);
    } 
}

public class Plus extends Expression {
    private Expression left, right;
    
    @Override
    public <R> R accept(Visitor<R> v) {
        return v.visit(this);
    }
}
```

The `Visitor` contains a trace of all the data variants. Because there's a `visit` implementation for each of the data variants.
If a new data variant is added, then the `Visitor` interface has to be changed and new implementation for that variant should be added.
Therefore this is not a neat solution.

### Type Classes to the Rescue

Lets breakdown the data declarations.

```haskell

data Constant = Constant Int
data Plus l r = Plus l r  
```
Define a generic type class `Expression` and add instances for all data types.

```haskell

-- type class
class Expression x

instance Expression Constant

instance (Expression l, Expression r) => Expression (Plus l r)
    
```
Evaluation functionality is defined in a separate type class.

```haskell

-- type class that contains evaluate operation
class Expression x => Evaluate x where
  evaluate :: x -> Int

instance Evaluate Constant where
  evaluate (Constant i) = i
  
instance (Expression l, Expression r) => 
                           Evaluate (Plus l r) where
  evaluate (Plus l r) = evaluate l + evaluate r
      
```
Lets try to extend data by adding `Minus`

```haskell

data Minus l r = Minus l r

instance (Expression l, Expression r) => Expression (Minus l r)

instance (Expression l, Expression r) => Evaluate (Minus l r) where
  evaluate (Minus l r) = evaluate l - evaluate r
      
```
Lets try to extend operations by adding `stringify`

```haskell
class Expression x => PrettyPrint x where
  stringify :: x -> string
  
instance PrettyPrint Constant where
  stringify (Constant i) = show i
  
instance (Expression l, Expression r) => 
              PrettyPrint (Expression (Plus l r)) where
  stringify (Plus l r) = stringify l 
                          ++ " + " ++ stringify r

instance (Expression l, Expression r) => 
              PrettyPrint (Expression (Minus l r)) where
  stringify (Minus l r) = stringify l 
                          ++ " - " ++ stringify r
```

That's it. 
We have extended both `data` and `operators` without touching old code.





