# Unit 17: Abstract Class

!!! abstract "Learning Objectives"


    After this unit, students should be able to:

    - explain why abstract classes are needed to model incomplete abstractions
    - declare and use abstract classes and abstract methods correctly in Java
    - reason about compile-time constraints enforced by abstract methods and classes
    - design class hierarchies that use abstract classes to support polymorphism safely
    - distinguish clearly between abstract and concrete classes and their roles

!!! abstract "Overview"

    In earlier units, we learned how inheritance and polymorphism allow us to write code that works at a higher level of abstraction.  For example, writing methods that operate on `Object`, or on a superclass such as `Shape`, rather than on a specific subclass like `Circle`. This allowed our programs to be more extensible and reusable.

    However, as we push abstraction further, we encounter a new problem: some classes are too abstract to be fully implemented.  In this unit, we introduce abstract classes, a language mechanism that allows us to express such incomplete abstractions explicitly. Abstract classes let us define what must be implemented by subclasses, while preventing misuse, such as instantiating objects that are conceptually incomplete. More importantly, abstract classes allow the compiler to enforce design constraints that would otherwise lead to subtle runtime bugs.

    This unit completes the abstraction story that began with inheritance and polymorphism by showing how Java helps us encode design intent directly into the type system.

## High-Level Abstraction

Recall that the concept of abstraction involves hiding away unnecessary complexity and details so that programmers do not have to be bogged down with the nitty-gritty.

When we code, we should, as much as possible, try to work with the higher-level abstraction, rather than the detailed version.  Following this principle would allow us to write code that is general and extensible, by taking full advantage of inheritance and polymorphism.

Take the following example which you have seen,
```Java title="contains v0.1 with Polymorphism"
boolean contains(Object[] array, Object obj) {
  for (Object curr : array) {
    if (curr.equals(obj)) {
      return true;
    }
  }
  return false;
}
```

The function above is very general.  We do not assume and do not need to know about the details of the items being stored or searched.  All we required is that the `equals` method compared if two objects are equal.

In contrast, someone whose mind focuses on finding a circle might write something like this:
```Java title="contains v0.3 for Circle only"
boolean contains(Circle[] array, Circle circle) {
  for (Circle curr : array) {
    if (curr.equals(circle)) {
      return true;
    }
  }
  return false;
}
```

The version above serves the purpose, but is not general enough.  The only method used is `equals`, which `Circle` inherits/overrides from `Object` so using `Circle` for this function is too constraining.  We can reuse this for any other subclasses of Circle, but not other classes.

## Abstracting Circles

Consider the following function, which finds the largest area among the circles in a given array:

```Java title="findLargest v0.1 with Circle"
double findLargest(Circle[] array) {
  double maxArea = 0;
  for (Circle curr : array) {
	double area = curr.getArea();
    if (area > maxArea) {
	  maxArea = area;
    }
  }
  return maxArea;
}
```

`findLargest` suffers from the same specificity as version 0.3 of `contains`.  It only works for `Circle` and its subclasses.  Can we make this more general?  We cannot replace `Circle` with `Object`,

```Java title="findLargest v0.2 with Object"
double findLargest(Object[] array) {
  double maxArea = 0;
  for (Object curr : array) {
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
    }
  }
  return maxArea;
}
```

since `getArea` is not defined for a generic object (e.g., what does `getArea` of a string mean?).

To allow us to apply `findLargest` to a more generic object, we have to create a new type &mdash; something more specific than `Object` that supports `getArea()`, yet more general than `Circle`.

## Shape

Let's create a new class called `Shape`, and redefine our `Circle` class as a subclass of `Shape`. We can now create other shapes, `Square`, `Rectangle`, `Triangle`, etc, and define the `getArea` method for each of them.

With the new `Shape` class, we can rewrite `findLargest` as:

```Java title="findLargest v0.3 with Shape"
double findLargest(Shape[] array) {
  double maxArea = 0;
  for (Shape curr : array) {
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
    }
  }
  return maxShape;
}
```

This version not only works for an array of `Square`, `Rectangle`, `Circle`, etc but also an array containing multiple shapes!

Let's actually write out our new `Shape` class:

```Java
class Shape {
  public double getArea() {
    // what is an area of an unknown shape?
  }
}
```

and rewrite our `Circle`:

```Java title="Circle v0.8 extending from Shape" hl_lines="6 23"
import java.lang.Math;

/**
 * A Circle object encapsulates a circle on a 2D plane.
 */
class Circle extends Shape {
  private Point c;   // the center
  private double r;  // the length of the radius

  /**
   * Create a circle centered on Point c with given radius r
   */
  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  /**
   * Return the area of the circle.
   */
  @Override
  public double getArea() {
    return Math.PI * this.r * this.r;
  }

  /**
   * Return true if the given point p is within the circle.
   */
  public boolean contains(Point p) {
    // TODO: Left as an exercise
    return false;
  }

  /**
   * Return the string representation of this circle.
   */
  @Override
  public String toString() {
    return "{ center: " + this.c + ", radius: " + this.r + " }";
  }

  /**
   * Return true if the object is the same circle (i.e., same center, same radius).
   */
  @Override
  public boolean equals(Object obj) {
    if (obj instanceof Circle) {
      Circle circle = (Circle) obj;
      return (circle.c.equals(this.c) && circle.r == this.r);
    }
  }
}
```

Notably, since our `Shape` is a highly abstract entity, it does not have any fields.  A key question is: how are we going to write `Shape::getArea()`?   We cannot compute the area of a shape unless we know what sort of shape it is.

One solution is to make `Shape::getArea()` return 0.

```Java
class Shape {
  public double getArea() {
    return 0;
  }
}
```

This design is unsafe and error-prone.  It is easy for someone to inherit from `Shape`, but forget to override `getArea()`.  If this happens, then the subclass will have an area of 0.  This leads to silent logical errors that the compiler cannot detect.

As we usually do in CS2030S, we want to exploit programming language constructs and rely on the compiler to check and catch such errors for us.  Abstract methods shift error detection from runtime to compile time. If a subclass fails to implement an abstract method, the program will not compile—preventing incomplete implementations from slipping into execution.

## Abstract Methods and Classes

This brings us to the concept of _abstract classes_.  An abstract class in Java is a class that has been made into something so general that it cannot and should not be instantiated.  Usually, this means that one or more of its instance methods cannot be implemented without further details.

The `Shape` class above makes a good abstract class since we do not have enough details to implement `Shape::getArea`.

To declare an abstract class in Java, we add the `abstract` keyword to the `class` declaration.  To make an instance method abstract, we add the keyword `abstract` when we declare the instance method.

An `abstract` instance method cannot be implemented and therefore should not have any method body.

This is how we implement `Shape` as an abstract class.

```Java
abstract class Shape {
  abstract public double getArea();
}
```

An abstract class cannot be instantiated.  Any attempt to do so, such as:
```
Shape s = new Shape();
```

would result in an error.

```
_.java:_: error: Shape is abstract; cannot be instantiated
    Shape s = new Shape();
              ^
1 error
```

Note that our simple example of `Shape` only encapsulates one abstract instance method.  An abstract class can contain multiple fields and multiple methods (including class methods).  Not all the methods have to be abstract.  As long as one of them is abstract, the class becomes abstract.

To illustrate this, consider the following implemetation of the abstract class `Shape`.

```Java
abstract class Shape {
  private int numOfAxesOfSymmetry ;

  public boolean isSymmetric() {
    return numOfAxesOfSymmetry > 0;
  }

  abstract public double getArea();
}
```

`Shape::isSymmetric()` is a concrete instance method but the class is still abstract since `Shape::getArea()` is abstract.

Note that the rule for declaring an abstract class is not symmetric.  A class with _at least one_ abstract instance method must be declared abstract.  On the other hand, an abstract class _may have no_ abstract method.

## Concrete Classes

We call a class that is not abstract as a _concrete class_.  A concrete, non-abstract, class cannot have any abstract method.  Thus, any concrete subclass of `Shape` must override `getArea()` to supply its own implementation.
