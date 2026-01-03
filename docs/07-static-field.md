# Unit 7: Class Fields

!!! abstract "Learning Objectives"

    After this unit, students should be able to:

    - distinguish clearly between instance fields and class (static) fields, both conceptually and syntactically.
    - explain when and why a field should be declared static, final, or both.
    - define and access class fields correctly using class names.
    - recognize common use cases of class fields (constants, shared configuration, precomputed values).
    - refactor code to eliminate “magic numbers” using appropriate class fields.

## Introduction

In earlier units, we treated objects as the fundamental building blocks of a program, each with its own state. However, not all values naturally belong to individual objects. Some values, such as mathematical constants or configuration parameters, are shared universally and remain the same across all instances.

This unit introduces class (static) fields, which belong to a class rather than any specific object. You will learn how Java supports shared state through the `static` keyword, how this differs from instance fields, and how class fields are commonly used to define constants and utility values in well-designed programs.

## Why Class Fields Exist

Let's revisit the following implementation of `Circle`.
```Java title="Circle v0.3"
class Circle {
  private double x;
  private double y;
  private double r;

  public Circle(double x, double y, double r) {
    this.x = x;
    this.y = y;
    this.r = r;
  }

  public double getArea() {
    return 3.141592653589793 * this.r * this.r;
  }
}
```

In the code above, we use the constant $\pi$ but hardcode it as 3.141592653589793.  Hardcoding such a magic number is considered poor coding style.  This constant can appear in more than one place. If we hardcode such a number and want to change its precision later, we would need to trace down and change every occurrence.  Every time we need to use $\pi$, we have to remember or look up what is the precision that we use.  Not only does this practice introduce more work, it is also likely to introduce bugs.  

In C, we define $\pi$ as a macro constant `M_PI`.  But how should we do this in Java?  This is where the idea that a program consists of only objects with internal states that communicate with each other can feel a bit constraining.  The constant $\pi$ is universal and does not really belong to any object (the value of $\pi$ is the same for every circle!).  

Another example is the method `sqrt()`, which computes the square root of a given number.  `sqrt` is a general function that is not associated with any object as well.  We will study such class methods in the next unit; they follow the same class-level principle as class fields.

A solution to this is to associate these _global_ values and functions with a _class_ instead of with an _object_.  For instance, Java predefines a [`java.lang.Math`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Math.html) class[^1] that is populated with constants `PI` and `E` (for Euler's number $e$), along with a long list of mathematical functions.  To associate a method or a field with a class in Java, we declare them with the `static` keyword.  We can additionally add the keyword `final` to indicate that the value of the field will not change and `public` to indicate that the field is accessible from outside the class.  In short, the combination of `public static final` modifiers is used for constant values in Java.

[^1]: The class `Math` is provided by the package `java.lang` in Java.  A package is simply a set of related classes (and interfaces, but I have not told you what is an interface yet).  The package `java.lang.Math` is automatically imported by the Java compiler.

```Java
class Math {
  :
  public static final double PI = 3.141592653589793;
  :
  :
}
```

We call `static` fields that are associated with a class as _class fields_ and fields that are associated with an object as _instance fields_.  Note that, a `static` class field needs not be `final` and it needs not be `public`.  Class fields are useful for storing pre-computed values or configuration parameters associated with a class rather than individual objects. 
Because it is associated with the class rather than an instance, a `static` field has **exactly one** shared instance during the entire execution of the program.  
They introduce implicit shared state. Any method that modifies a class field affects all objects of that class.  

Class fields are not a violation of object-oriented design. Instead, they acknowledge that some state conceptually belongs to a class as a whole, not to individual objects.  If a field’s value would be identical across all instances, it is a strong candidate for being a class field.

Caution: because class fields introduce global shared state, they can make programs harder to reason about if overused.

## Using Class Fields

A class field behaves just like a global variable and can be accessed in the code, anywhere the class can be accessed.  Since a class field is associated with a class rather than an object, we access it through its _class name_.  

```Java
public double getArea() {
  return Math.PI * this.r * this.r;
}
```

Below is an example of a non-constant class field.  We introduce a class field `circleCount` to count how many `Circle` objects have been created so far.

```Java title="Circle v0.4"
class Circle {
  private static int circleCount = 0; // class field to count circles
  private double x;
  private double y;
  private double r;

  public Circle(double x, double y, double r) {
    this.x = x;
    this.y = y;
    this.r = r;
    circleCount++; // increment the count whenever a new Circle is created
  }

  public double getArea() {
    return Math.PI * this.r * this.r;
  }

  public static int getCircleCount() {
    return circleCount; // return the total number of Circle instances created
  }
}
```

Here, there is only exactly one instance of `circleCount` regardless of how many instances of `Circle` we have created.  In fact, we need not create any instance of `Circle` at all to be able to use `circleCount`.

```Java
Circle.getCircleCount(); // returns 0 without instance of `Circle`
new Circle(0, 0, 1);
new Circle(0, 0, 1);
Circle.getCircleCount(); // returns 2
```

!!! note "Class Fields and Methods in Python"
    Note that, in Python, any variable declared within a `class` block is a class field:
    ```Python
    class Circle:
      x = 0
      y = 0
    ```

    In the above example, `x` and `y` are class fields, not instance fields.


