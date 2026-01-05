# Unit 8: Class Methods

!!! abstract "Learning Objectives"

    After this unit, students should be able to:

    - distinguish clearly between instance methods and class (static) methods.
    - explain why class methods cannot access instance fields or use this.
    - define and invoke class methods correctly using the class name.
    - describe the role of the main method as the entry point of a Java program.
    - identify the required modifiers, return type, and parameters of a valid main method.

!!! abstract "Overview"

    In previous units, we focused on instance methods, which operate on the state of individual objects. However, not all behavior in a program naturally belongs to a specific object. Some operations conceptually belong to the class as a whole—for example, keeping track of how many objects have been created, or providing general-purpose utility functions.

    In this unit, we introduce class methods, which are declared using the static keyword. You will learn how class methods differ from instance methods, why they cannot access instance-specific state, and how Java uses a special class method, main, as the entry point for every program.

## Static Methods

In the previous unit, we added a class field `circleCount` to our `Circle` class to keep track of how many `Circle` instances have been created, and we added a method `getCircleCount()` to return its value.  However, `getCircleCount()` was still defined as an instance method.  This means that we had to invoke it through a specific `Circle` instance, even when the field `circleCount` exists without an `Circle` instance.

Since `getCircleCount()` does not depend on any particular `Circle` instance, it is more appropriate to define it as a _class method_.  A class method is defined with the `static` modifier.  Here is an improved version of the `Circle` class that uses a class method to return the number of circles created so far.

```Java title="Circle v0.3b with Class Method" hl_lines="18"
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
    return Circle.circleCount; // return the total number of Circle instances created
  }
}
```

Similar to a class field, a class method is associated with a class, not with an instance of the class.  We can now invoke `getCircleCount()` through the class name `Circle`, without needing to create any instance of `Circle`.

```Java title="Invoking Class Method through Class Name"
Circle.getCircleCount(); // returns 0
new Circle(0, 6, 7)
new Circle(1, 1, 2);
Circle.getCircleCount(); // returns 2
```

Other examples of class methods include the methods provided in `java.lang.Math`: `sqrt`, `min`, etc.  These methods can be invoked through the `Math` class: e.g., `Math.sqrt(x)`.

As a rule of thumb, use an instance method if the behavior depends on instance fields; Use a class method if the behavior conceptually belongs to the class and does not depend on any particular object.  Overusing class methods can lead to procedural-style code and should be avoided when behavior naturally belongs to objects.

### Accessing Instance Fields from Class Methods

Just as a class field represents shared state, a class method represents shared behavior.  Recall that for static fields (i.e., class fields), we only have exactly one instance of it throughout the lifetime of the program.  More generally, a field or method with modifier `static` belongs to the class rather than the specific instance.  In other words, they can be accessed/updated (for fields, assuming proper access modifier) or invoked (for methods, assuming proper access modifier) without even instantiating the class.

As a consequence, if we have not instantiated a class, no instance of that class has been created.  The keyword `this` is meant to refer to the _current instance_, and if there is no instance, the keyword `this` is not meaningful.  Therefore, within the context of a `static` method, Java actually prevents the use of `this` from any method with the `static` modifier.

For example,
```Java
  public static int getCircleCount() {
    return this.circleCount; 
  }
```
will result in the following error.

```
Error: non-static variable this cannot be referenced from a static context
  	return this.circleCount;
               ^
```

The opposite is not true.  We can access class fields from instance methods.

## The `main` method

The most common class method you will use is probably the `main` method.

Every Java program has a class method called `main`, which serves as the entry point to the program.  To run a Java program, we need to tell the JVM the class whose `main` method should be invoked first.  In the example that we have seen,
```
java Hello
```

will invoke the `main` method defined within the class `Hello` to kick start the execution of the program.  The main method must be declared `static` because it is invoked by the JVM before any objects of the class are created.

The `main` method must be defined in the following way:
```Java
public static final void main(String[] args) {
}
```

You have learned what `public` and `static` means.  The return type `void` indicates that `main` must not return a value.  We have discussed what `final` means on a field, but are not ready to explain what `final` means on a method yet.

The `main` method takes in an array (`[]`) of strings as parameters.  These are the command-line arguments that we can pass in when invoking `java`.  [`String`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html) (or `java.lang.String`) is another class provided by the Java library that encapsulates a sequence of characters.
