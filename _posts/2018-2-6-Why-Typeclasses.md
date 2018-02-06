---
layout: post
title: Why Type Classes?
comments: true
---

The concept of Type class is one solution for the expression problem which is a major challenge for programing languages even today.
Lets explore what this problem means.

### The Expression Problem

According to [Wikipedia](https://en.wikipedia.org/wiki/Expression_problem),

```text
The goal is to define a datatype by cases, where one can add 
new cases to the datatype and new functions over the datatype, 
without recompiling existing code, and while retaining static 
type safety (e.g., no casts).

```
Lets boil this down. 

We can consider computer programs as a combination of `data` and `operations`. If we want to add new `data` or `operations` we should not touch the existing code.
This is the challenge expressed by the Expression problem.

One of the major distinctions between Functional Programing and Object Oriented Programing is, 
in functional programing, data and related operations are completely separated and in OOP data encapsulates the related operations. 

In Haskell, the basic programing style is to define your ADTs(Algebraic Data Types) at the beginning of the program. 

```haskell
module Data where

data Expression = Constant Int | Add Expression Expression

evaluate :: Expression -> Int
evaluate (Constant i) = i
evaluate (Add e1 e2) = result e1 + result e2 
```

`Constant` and `Add` are data constructors for type `Expression`


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

class Expression extends Expression {
    private Expression left, right;
    
    // getters and setters
    // ...
    
    public int evaluate() {
        return left.evaluate() + right.evaluate();
    } 
} 
```
`Constant` and `Add` are separate implementations of interface `Expression`

Lets assume that a pretty printer has to be added for type `Expression`.

```haskell

module PrettyPrinter where

import Data

prettyPrint :: Expression -> String
prettyPrint (Constant i) = show i
prettyPrint (Add l r) = prettyPrint l ++ " + " ++ prettyPrint r
```
It seems it is easy to add a new operation like `prettyPrint`. Let's try the same in Java. 

```java

interface Expression {
    int evaluate;
    String prettyPrint;
}

class Constant extends Expression {
    private int value; 
    
    // getters and setters
    // ..
    
    public int evaluate() {
        return this.value;
    }
    
    public String prettyPrint() {
        return this.value.toString();
    }
}

class Expression extends Expression {
    private Expression left, right;
    
    // getters and setters
    // ...
    
    public int evaluate() {
        return left.evaluate() + right.evaluate();
    } 
    
    public String prettyPrint() {
        return left.toString() + " + " + right.toString();
    }
} 
```

It is not easy to add a new operation like `prettyPrint`, because all the implementations of `Expression` has to be updated with the new `operation`.

Lets try to add a new data type for `Expression`

```haskell
    
module Data where

data Expression = Constant Int 
                | Add Expression Expression 
                | Neg Int

evaluate :: Expression -> Int
evaluate (Constant i) = i
evaluate (Add e1 e2) = evaluate e1 + evaluate e2 
evaluate (Neg x) = 0 - evaluate x
```
```haskell

module PrettyPrinter where

import Data

prettyPrint :: Expression -> String
prettyPrint (Constant i) = show i
prettyPrint (Add l r) = prettyPrint l 
                        ++ " + " ++ prettyPrint r
prettyPrint (Neg x) = " -(" ++ prettyPrint x ++ ")"
```

In Haskell, its not easy to add a new data type since all the functions has to be updated with the new value. 

```java
    
public class Neg extends Expression {
    private Expression operand;
    @Override
    public String prettyPrint() {
        return "-(" + operand.prettyPrint() + ")";
    }
}
```
In Java, a new type can be added without touching the existing code. 

Therefore, in a functional programing language like haskell, its easy to get `data` extensibility but not `operator` extensibility. In an OOP language like Java,
its easy to get `data` extensibility but not `operator` extensibility. 

### How about Visitor Pattern?

Visitor Patterns doesn't exactly solve the problem but it's worth trying.

```java

interface Visitor<R> {
    R visit(Constant that);
    R visit(Add that);
}

public abstract class Expression {
    public abstract <R> R accep(Visitor<R> v);
}

public class PrettyPrinter implements Visitor<String> {
    public String visit(Constant that) {
        return that.info.toString();
    }
    
    public String visit(Add that) {
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

public class Add extends Expression {
    private Expression left, right;
    
    @Override
    public <R> R accept(Visitor<R> v) {
        return v.visit(this);
    }
}
```

The `Visitor` contains a snapshot of all the data variants. Because there's a `visit` implementation for each of the data variant.
If a new data variant is added, then the `Visitor` interface has to be changed and new implementation for that variant should be added.
Therefore this is not a neat solution.

### Type Classes to the Rescue

Lets breakdown the data declarations.

```haskell

data Constant = Constant Int
data Add l r = Add l r  
```
Define a generic type class `Expression` and add instances for all data types.

```haskell

-- type class
class Expression x

instance Expression Constant

instance (Expression l, Expression r) => Expression (Add l r)
    
```
Evaluation functionality is defined in a separate type class.

```haskell

-- type class that contains evaluate operation
class Expression x => Evaluate x where
  evaluate :: x -> Int

instance Evaluate Constant where
  evaluate (Constant i) = i
  
instance (Expression l, Expression r) => 
                           Evaluate (Add l r) where
  evaluate (Add l r) = evaluate l + evaluate r
      
```
Lets try to extend data by adding `Neg`

```haskell

data Neg x = Neg x

instance Expression x => Expression (Neg x)

instance Expression x => Evaluate (Neg x) where
  evaluate (Neg x) = 0 - evaluate x
      
```
Lets try to extend operations by adding `prettyPrint`

```haskell
class Expression x => PrettyPrint x where
  prettyPrint :: x -> string
  
instance PrettyPrint Constant where
  prettyPrint (Constant i) = show i
  
instance (Expression l, Expression r) => 
              PrettyPrint (Expression (Add l r)) where
  prettyPrint (Add l r) = prettyPrint l 
                          ++ " + " ++ prettyPrint r

instance Expression x => PrettyPrint (Neg x) where
  prettyPrint (Neg x) = " -(" ++ prettyPrint x ++ ")"
```

That's it. 
We have extended both `data` and `operators` without touching old code.





