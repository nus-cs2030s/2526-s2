# Unit 18: Interface

!!! abstract "Learning Objectives"

    After completing this unit, students should be able to:

    - Explain how interfaces model capabilities (“can-do” behavior) independently of class hierarchies.
    - Use interfaces to generalize code beyond inheritance and avoid inappropriate IS-A relationships.
    - Reason about subtype–supertype relationships involving interfaces, including multiple supertypes.
    - Predict and explain compile-time and run-time behavior when casting objects to interface types.
    - Recognize the design trade-offs involved in evolving interfaces, including the motivation for default methods.

## Introduction

In the previous units, we used inheritance, especially abstract classes, to write code that is more general and reusable. By programming against a superclass rather than a concrete class, we learned how to decouple what a method needs from how that behavior is implemented.

However, inheritance has an important limitation: it models an IS-A relationship. Not every form of generalization fits naturally into a single class hierarchy.

In this unit, we take abstraction one step further. Instead of modeling what something is, we focus on what it can do. This shift allows us to write highly flexible code that works across unrelated class hierarchies—without forcing unnatural inheritance relationships.

To achieve this, we introduce interfaces: a Java abstraction that models behavior rather than identity. Interfaces will allow us to express common capabilities, reason about multiple supertypes, and write code that is both more general and more precise.

## Modeling Behavior

Let's reexamine this method again:
```Java title="findLargest v0.3 with Shape"
double findLargest(Shape[] array) {
  double maxArea = 0;
  for (Shape curr : array) {
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
    }
  }
  return maxArea;
}
```

The only requirement for this method to compile and run correctly is that the element type provides a `getArea()` method.  While `Shape` that we defined in the previous unit meets this requirement, it does not have to be.  We could pass in an array of countries or an array of HDB flats.  It would be inapprpriate to model a `Country` or a `Flat` as a subclass of `Shape`, since inheritance models the IS-A relationship.

To resolve this, we will look at an abstraction that _models what an entity can do_, possibly across different class hierarchies.

## Interface

The abstraction to do this is called an _interface_.  An interface is also a type and is declared with the keyword `interface`.

Since an interface models what an entity can do, the name usually ends with the -able suffix[^1].

Now, suppose we want to create a type that supports the` getArea()` method, be it a shape, a geographical region, or a real estate property.  Let's call it `GetAreable`:
```Java
interface GetAreable {
  public abstract double getArea();
}
```

All methods declared in an interface are `public abstract` by default.  We could also just write:
```Java
interface GetAreable {
  double getArea();
}
```

Now, for every class that we wish to be able to call `getArea()` on, we tell Java that the class `implements` that particular interface.

For instance,
```Java
abstract class Shape implements GetAreable {
  private int numOfAxesOfSymmetry;

  public boolean isSymmetric() {
    return numOfAxesOfSymmetry > 0;
  }
}
```

The `Shape` class will now have a `public abstract double getArea()` thanks to the implementation of the `GetAreable` interface.

We can have a concrete class implementing an interface too.

```Java
class Flat extends RealEstate implements GetAreable {
  private int numOfRooms;
  private String block;
  private String street;
  private int floor;
  private int unit;

  @Override
  public double getArea() {
      :
  }
}
```

For a class to implement an interface and be concrete, it has to override all abstract methods from the interface and provide an implementation to each, just like the example above.  Otherwise, the class becomes abstract.

With the `GetAreable` interface, we can now make our function `findLargest` even more general.
```Java title="findLargest v0.4 with GetAreable"
double findLargest(GetAreable[] array) {
  double maxArea = 0;
  for (GetAreable curr : array) {
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
    }
  }
  return maxArea;
}
```

Note:

- A class can only extend from one superclass, but it can implement multiple interfaces.
- An interface can extend from one or more other interfaces, but an interface cannot extend from another class.

As a rule of thumb, use an interface when you want to model a capability or behavior that can be shared across different class hierarchies. Use an abstract class when you want to model shared states or base functionality within a class hierarchy.

## Interface as Supertype

If a class $C$ implements an interface $I$, $C <: I$.   This definition implies that a type can have multiple supertypes.

In the example above, `Flat` <: `GetAreable` and `Flat` <: `RealEstate`.

## Casting using an Interface

Like any type in Java, it is also possible to cast a variable to an interface type. Let's consider an interface `I` and two classes `A` and `B`. Note that `A` does not implement `I`

```Java
interface I {
  :
}

class A {
  :
}

class B implements I {
  :
}
```

Consider the following three variables:

```Java
I i;
A a;
B b;
```

First, let's recap whether the compiler allows the following when we convert between classes with and without a subtype relationship:
```Java
i = b; // Compiles.  Widening conversion: B <: I
b = i; // Does not compile.  Narrowing conversion: I <: B
i = a; // Does not compile.  There is no A <: I relationship
```

Now, let's consider narrowing conversion from `B` to `A` through an explicit cast:
```Java
(A) b; 
```

The above does not compile, since there is no `B` $<:$ `A` relationship.  However, the following conversion from `B` to `I` works:
```Java
(I) b;  
```

since `B` $<:$ `I`.  

Finally, consider the conversion from `A` to `I`:
```Java
(I) a; 
```

Even though `A` $\not <:$ `I`, the Java compiler allows this code to compile! Why is that so? 

The Java compiler does not let us cast when it is provable that it will not work, i.e. casting between two classes that have no subtype relationship. However, for interfaces, there is the *possibility* that a subclass *could* implement the interface.  Therefore, the Java compiler trusts that the programmer knows what they are doing, and allows it to compile. Consider one such potential subclass `AI`:

```Java
class AI extends A implements I{
  :
}
```

The key takeaway is that an explicit cast tells the compiler to trust the programmer, and therefore it may not warn us or stop us from making bad decision. It is important to always be sure whenever you use an explicit typecast, otherwise the code may compile but throw a `ClassCastException` at runtime.  

Here is an example that demonstrates this:

```Java
A a = new A();
i = (I) a; // Compiles, but failed during execution
```

## Impure Interfaces

As we mentioned at the beginning of this module, it is common for software requirements, and their design, to continuously evolve.  Once an interface is exposed beyond an abstraction barrier, changing it becomes difficult.  Unlike classes, interfaces represent a contract that many independent implementations may rely on.

Suppose that, after we define that `GetAreable` interface, other developers in the team start to write classes that implement this interface.  One fine day, we realized that we need to add more methods to the `GetAreable` interface.  Perhaps we need methods `getAreaInSquareFeet()` and `getAreaInSquareMeter()` in the interface.  

If we simply add these methods as abstract methods to the interface, all existing implementing classes will immediately fail to compile unless they are updated to provide implementations. This makes interfaces particularly hard to evolve once they are in use.

This exact problem arose when Java transitioned from version 7 to version 8. The Java standard library needed to extend existing interfaces with new methods, but doing so would have broken a large amount of existing code.

To address this, Java allows interfaces to include default methods—methods that provide a concrete implementation. Implementing classes automatically inherit these methods unless they choose to override them.

```Java
interface GetAreable {
  double getArea();

  default double getAreaInSquareFeet() {
    return getArea() * 10.7639;
  }
}
```

While default methods improve backward compatibility, they blur the clean conceptual distinction between interfaces and classes. An interface may now contain both abstract method declarations and concrete method implementations.

In CS2030S, we refer to such interfaces as _impure interfaces_.
In this course, we treat interfaces conceptually as pure specifications of behavior.  You are not expected to define default methods, and we will reason about interfaces as if they contain only abstract methods.

The key takeaway is not the syntax of default methods, but the design lesson: interfaces are difficult to change once published, so they must be designed carefully and conservatively.  This restriction is one reason why we often prefer abstract classes when evolution and shared implementation are important, and interfaces when expressing minimal, stable capabilities.

[^1]: Although in recent Java releases, this is less common.
