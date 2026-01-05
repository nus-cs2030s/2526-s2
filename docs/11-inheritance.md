# Unit 11: Inheritance

!!! abstract "Learning Objectives"

    After completing this unit, students should be able to:

    - explain inheritance as a mechanism for modeling subtyping and the is-a relationship
    - decide when inheritance is appropriate versus composition, and justify that choice
    - use the extends and super keywords correctly to define and initialize subclasses
    - reason about compile-time types versus runtime types in the presence of inheritance
    - predict the behavior of code involving subtype polymorphism
    - determine when narrowing type conversion (casting) is permitted and when it may fail at runtime

!!! abstract "Overview"
    In earlier units, we learned how to build complex abstractions by composing objects, carefully preserving abstraction barriers and separating client and implementer responsibilities. Composition remains our default design tool—but it is not the only one.

    In this unit, we introduce inheritance, a second mechanism for extending behavior that allows one abstraction to be treated as a more specific version of another. Inheritance is not primarily about code reuse; rather, it is about modeling subtyping—when one object can safely stand in for another.

    We will examine how inheritance expresses the is-a relationship, how Java supports this via extends and super, and why careless use of inheritance can silently break program meaning. Along the way, we will sharpen an important distinction between compile-time and runtime types, a concept that underpins polymorphism in later units.

    By the end of this unit, you should not only be able to write subclasses, but also explain when you should not.

## Extension with Composition

We have seen how composition allows us to compose a new, more complex, class, out of existing classes, without breaking the abstraction barrier of existing classes.  Sometimes, however, composition is not the right approach.  Let's consider the following example.  Let's suppose that we, as a client, want to add color as a property to our `Circle`.

Without penetrating the abstraction barrier of `Circle`, we can do the following:
```Java title="ColoredCircle v0.1 (with Composition)"
class ColoredCircle {
  private Circle circle;
  private Color color;

  public ColoredCircle(Circle circle, Color color) {
    this.circle = circle;
    this.color = color;
  }
}
```

where `Color` is another abstraction representing the color of shapes.

What should we do if we want to calculate the area of our colored circle?  Suppose we already have a `ColoredCircle` instance called `coloredCircle`. We could make `circle` public and call `#!Java coloredCircle.circle.getArea()`, or we could add an accessor and call `#!Java coloredCircle.getCircle().getArea()`.  Both of these are not ideal, since they break the abstraction barrier and reveal that the `ColoredCircle` class stores a `circle` (the latter being slightly better than the first).

A better alternative is to let `ColoredCircle` provide its own `getArea()` method and _forward_ its call to `Circle`.

```Java title="ColoredCircle v0.2 (with Composition)"
class ColoredCircle {
  private Circle circle;
  private Color color;

  public ColoredCircle(Circle circle, Color color) {
    this.circle = circle;
    this.color = color;
  }

  public double getArea() {
    return circle.getArea();
  }
}
```

Then, the client to `ColoredCircle` can just call `#!Java coloredCircle.getArea()` without knowing how a colored circle is represented internally.  The drawback of this approach is that we might end up with many such boilerplate forwarding methods.

## Extension with Inheritance

Recall the concept of subtyping.  We say that $S <: T$ if any piece of code written for type $T$ also works for type $S$ without changing the code's intended behavior.

Now, think about `ColoredCircle` and `Circle`.  If someone has written a piece of code that operates on `Circle` objects.  Do we expect the same code to work on `ColoredCircle`?  In this example, yes!  A `ColoredCircle` object should behave just like a circle &mdash; we can calculate its area, calculate its circumference, check if two circles intersect, check if a point falls within the circle, etc.  The only difference, or more precisely, extension, is that it has a color, and perhaps has some methods related to this additional field.  So, `ColoredCircle` _is a subtype of_ `Circle`.

We now show how we can introduce this subtype relationship in Java, using the `extends` keyword.  We can reimplement our `ColoredCircle` class this way:

```Java title="ColoredCircle v0.3 (with Inheritance)"
class ColoredCircle extends Circle {
  private Color color;

  public ColoredCircle(Point center, double radius, Color color) {
    super(center, radius);  // call the parent's constructor
    this.color = color;
  }
}
```

We have just created a new type called `ColoredCircle` as a class that extends from `Circle`.  We call `Circle` the _parent class_ or _superclass_ of `ColoredCircle`; and `ColoredCircle` a _subclass_ of `Circle`.  Note that if a class A is a subclass of B, A $<:$ B.  The converse is not true, A $<:$ B does not imply that A is a subclass of B (e.g., `int` is not a subclass of `float`).

We also say that `ColoredCircle` _inherits_ from `Circle`, since all the public fields of `Circle` (if any) and public methods (like `getArea()`) are now accessible to `ColoredCircle`.  Just like a parent-child relationship in real life, however, anything private to the parent remains inaccessible to the child.  This privacy veil maintains the abstraction barrier of the parent from the child, and creates a bit of a tricky situation &mdash; technically a child `ColoredCircle` object has a center and a radius, but it has no access to it!

Line 6 of the code above introduces another keyword in Java: `super`.  Here, we use `super` to call the constructor of the superclass, to initialize its center and radius (since the child has no direct access to these fields that it inherited).

If a constructor does not explicitly invoke a superclass constructor, the Java compiler automatically inserts a call to the no-argument constructor of the superclass. If the super class does not have a no-argument constructor, you will get a compile-time error. `Object` does have such a constructor, so if `Object` is the only superclass, there is no problem.
The concept we have shown you is called _inheritance_ and is one of the four pillars of OOP.  We can think of inheritance as a model for the "_is a_" relationship between two entities.

With inheritance, we can call `#!Java coloredCircle.getArea()` without knowing how a colored circle is represented internally AND without forwarding methods.

## When NOT to Use Inheritance

Inheritance tends to get overused.  _In practice, we seldom use inheritance_.  Let's look at some examples of how _not_ to use inheritance, and why.

Consider the following example:

```Java
class Point {
  private double x;
  private double y;
    :
}

class Circle extends Point {
  private double radius;
    :
}

class Cylinder extends Circle {
  private double height;
    :
}
```

The difference between these implementations and the one you have seen in [Unit 9](09-composition.md) is that they use inheritance rather than composition.  

`Circle` implemented like the above would have the center coordinate inherited from the parent (so it has three fields, x, y, and radius); `Cylinder` would have the fields corresponding to a circle, which is its base and height.  In terms of modeling the properties of a circle and a cylinder, we have all the right properties in the right class.

When we start to consider methods encapsulated with each object, things start to break down. Consider a piece of code written as follows:
```Java
void foo(Circle c, Point p) {
  if (c.contains(p)) {
    // do something
  }
}
```

Since `Cylinder` is a subtype of `Point` according to the implementation above, the code above should still work also if we replace `Point` with a `Cylinder` (according to the semantics of subtyping).   But it gets weird &mdash; what is the meaning of a `Circle` (in 2D) containing a Cylinder (in 3D)?  We could come up with a convoluted meaning that explains this, but it is likely not what the original implementer of `foo` expects.

The message here is this: _Use composition to model a has-a relationship; and inheritance for an is-a relationship_.  _Make sure inheritance preserves the meaning of subtyping_.  

## Ensuring Valid Type Assignment During Runtime

During runtime, Java only allows a variable of compile-time type $T$ to hold a value of type $S$ if $S <: T$.   Otherwise, an error (to be precise, a `ClassCastException`) will be thrown at runtime.  To avoid this, the Java compiler conservatively enforces this rule at compile time.

Consider the following line of code:
```Java
Circle c = new ColoredCircle(p, 0, blue); // OK
```

Upon reading this line of code, the compiler determines that the right hand side has compile-time type `ColoredCircle`, while the left hand side has compile-time type `Circle`.  Since `ColoredCircle` is a subtype of `Circle`, the assignment is allowed.

Recall that the compile-time type of a variable is the type declared for it, while the runtime type is the type of the actual value stored in that variable at runtime.  In the example above, after the assignment occured during execution, the compile-time type of `c` is `Circle`, while its runtime type is `ColoredCircle`.

Now, consider the following line of code:
```Java
ColoredCircle c = new Circle(p, 0); // error
```

Here, the compiler sees that the right hand side has compile-time type `Circle`, while the left hand side has compile-time type `ColoredCircle`.  Since `Circle` is not a subtype of `ColoredCircle`, the assignment is rejected at compile time.

Next, consider the following code snippet:
```Java
Circle c = new ColoredCircle(p, 0, blue);
ColoredCircle cc = c;
```

Here, the first line is valid, as we have seen before.  However, the second line will be rejected by the compiler, since the compile-time type of `c` is `Circle`, while the compile-time type of `cc` is `ColoredCircle`.  Since `Circle` is not a subtype of `ColoredCircle`, the assignment is rejected at compile time.  Even though, at runtime, `c` actually holds a `ColoredCircle` object, the compiler does not (and cannot) consider that, and only checks the compile-time types of the variables and expressions, line-by-line.

## Narrowing Type Conversion

While the compiler is not able to consider the runtime types of variables, we, as human, can help it by using a type cast.  For instance, in the last example, we can be sure that `c` holds the value with runtime type of `ColoredCircle`, we can perform a type cast to help the compiler verify the assignment:
```Java
Circle c = new ColoredCircle(p, 0, blue);
ColoredCircle cc = (ColoredCircle) c;
```

Here, the type cast expression `(ColoredCircle) c` tells the compiler to treat `c` as if it has compile-time type `ColoredCircle`.  This casting is known as _narrowing type conversion_.  

During runtime, since `c` actually holds a value of runtime type `ColoredCircle`, the assignment will succeed when the code runs.

Typecasting must be used with care.  Here, we are overriding the compiler and ask it to trust us that we know what we are doing and `c` actually holds a value of type `ColoredCircle`.

Consider the example below:
```Java
Circle c = new Circle(new Point(0, 0), 1);
ColoredCircle cc = (ColoredCircle) c;
```

The variable `c` would hold a value of runtime type `Circle` after initialization.  However, the programmer is forcing the compiler to treat `c` as if it has compile-time type `ColoredCircle`.  This code compiles successfully, but the assignment would fail at runtime, throwing a `ClassCastException`. 

Note that the compiler does not blindly trust the programmer.  It still checks that the type conversion is _possible_.  In this example, since `Circle` is a supertype of `ColoredCircle`, it is possible that `c` holds a value of runtime type `ColoredCircle`.  Therefore, the compiler allows the code to compile.  If we try to cast between two unrelated types, for example:
```Java
Circle c = new Circle(new Point(0, 0), 1);
String s = (String) c; // error
```

The compiler would reject the code, since `Circle` and `String` are unrelated types, and no subtype relationship exists between them.
