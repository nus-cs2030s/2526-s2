# Unit 13: Overloading

!!! abstract "Learning Objectives"


    After reading this unit, students should be able to:

    - explain what method overloading is and how it differs from method overriding
    - identify when two methods are considered overloaded in Java
    - correctly define overloaded methods and constructors by varying parameter types, order, or arity
    - explain why changing only parameter names or return types does not result in overloading
    - reason about which overloaded method is selected at compile time based on argument types

!!! abstract "Overview"

    In the previous unit, we studied method overriding, which allows subclasses to replace the behavior of inherited methods at runtime. Overriding supports polymorphism, where the same method call can behave differently depending on the object’s runtime type.

    In this unit, we shift focus to method overloading, which addresses a different concern: convenience and expressiveness. Overloading allows a class to provide multiple methods with the same name that operate on different kinds of inputs, while still performing conceptually similar tasks.

    Unlike overriding, overloading is resolved entirely at compile time. Understanding this distinction is essential, as it explains both what Java allows you to overload, and why some seemingly reasonable overloads are rejected by the compiler.


## Method overloading

In the previous unit, we introduced _method overriding_ &mdash; when a subclass defines an instance method with the same _method descriptor_ as an instance method in the parent class.

In contrast, _method overloading_ occurs when a class has access to (either defined or inherited) two or more methods with the same name but a different _method signatures_[^1]. In other words, we create an overloaded method by changing the type, order, or number of parameters of the method while keeping the method name identical.

[^1]: Note that this is not the same as the _method descriptor_. You cannot overload a method by changing only the return type.

Let's consider an `add` method which allows us to add two numbers, and returns the result. What if we would like to create an `add` method to sum up three numbers?

```Java
public int add(int x, int y) {
  return x + y;
}

public int add(int x, int y, int z) {
  return x + y + z;
}
```

In the example above, the methods `add(int, int)` and `add(int, int, int)` are overloaded. They have the same name but a different number of parameters. We can see that this allows us to write methods to handle differing inputs.

Now let's consider our `Circle` class again. Our `Circle::contains(Point)` method allows us to check if a `Point` is within the radius of the current instance of the `Circle`. We would like to create a new method `Circle::contains(double, double)` which will allow us to check if an (`x`, `y`) coordinate (another valid representation of a point) is within our circle.

```Java title="Circle v0.6a with Overloaded `contains` Method" hl_lines="14-22"
class Circle {
  private Point c;
  private double r;

  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  public double getArea() {
    return Math.PI * this.r * this.r;
  }

  public boolean contains(Point p) {
    return false;
    // TODO: Left as an exercise
  }

  public boolean contains(double x, double y) {
    return false;
    // TODO: Left as an exercise
  }

  @Override
  public String toString() {
    return "{ center: " + this.c + ", radius: " + this.r + " }";
  }
}
```

In the above example, `Circle::contains(Point)` and `Circle::contains(double, double)` are overloaded methods.

Recall that overloading requires changing the order, number, and/or type of parameters and says nothing about the names of the parameters. Consider the example below, where we have two `contains` methods in which we swap parameter names.

```Java
  public boolean contains(double x, double y) {
    return false;
    // TODO: Left as an exercise
  }

  public boolean contains(double y, double x) {
    return false;
    // TODO: Left as an exercise
  }
```

Because parameter names are not part of the method signature, swapping parameter names does not produce a new method.  These two methods have the same method signature, and therefore `contains(double, double)` and `contains(double, double)` are not distinct methods.  The Java compiler will reject this code with an error indicating that the method is already defined.

As a constructor is also a method, it is possible to overload the class _constructor_ as well. As in the example below, we can see an overloaded constructor which gives us a handy way to instantiate a `Circle` object that is the unit circle.

```Java title="Circle v0.6b with Overloading Constructor" hl_lines="5-14"
class Circle {
  private Point c;
  private double r;

  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  // Overloaded constructor
  public Circle() {
    this.c = new Point(0, 0);
    this.r = 1;
  }
  
  // Other methods omitted 
}
```

```Java
// c1 points to a new Circle object with a center (1, 1) and a radius of 2
Circle c1 = new Circle(new Point(1, 1), 2);
// c2 points to a new Circle object with a center (0, 0) and a radius of 1
Circle c2 = new Circle();
```

It is also possible to overload `static` class methods in the same way as instance methods. In the next unit, we will see how Java chooses which method implementation to execute when a method is invoked.

Similar to `super`, the `this` keyword can be used to invoke another constructor.  While `super` is used to invoke the constructor in the superclass, `this` invokes an overloaded constructor in the same class. This is particularly useful as it allows us to avoid duplicating code. For example, we can modify our overloaded constructor in the `Circle` class to invoke the primary constructor instead of directly initializing the instance variables.

```Java title="Circle v0.6c with Overloaded Constructor using this()" hl_lines="12"
class Circle {
  private Point c;
  private double r;

  public Circle(Point c, double r) {
    this.c = c;
    this.r = r;
  }

  // Overloaded constructor
  public Circle() {
    this(new Point(0, 0), 1);  // call the primary constructor
  }
  
  // Other methods omitted 
}
```
