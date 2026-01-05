# Unit 20: Run-Time Class Mismatch

!!! abstract "Learning Objectives"

    After completing this unit, students should be able to:

    - explain why narrowing type conversion requires explicit casting in Java
    - distinguish between compile-time type checking and run-time type checking in the presence of casts
    - identify situations where a cast is syntactically valid but can fail at run time
    - reason about how abstraction (interfaces and supertypes) can lead to run-time class mismatch errors

!!! abstract "Overview"

    In earlier units, we learned how to write reusable and flexible code by programming to higher-level abstractions, such as interfaces and supertypes. This approach allows our code to work uniformly over many different concrete classes, improving extensibility and reuse.

    However, abstraction also comes with a cost. When we deliberately “forget” the concrete class of an object and treat it as a more general type, we sometimes need to recover that concrete type later. In Java, doing so requires type casting, and incorrect casts can lead to errors that only appear at run time.

    In this unit, we examine how such run-time class mismatch errors arise, why the compiler cannot always prevent them, and what responsibilities fall on the programmer when casting is used.

## Finding the Object with the Largest Area

Let's revisit our example of `findLargest` method, which takes in an array of objects that support the `getArea` method and returns the largest area among these objects.

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

The method served our purpose well, but it is NOT a very well-designed method.  Just returning the value of the largest area is not as useful as returning the _object_ with the largest area.  Once the caller has a reference of the object, the caller can call `getArea` to find the value of the largest area.

Let's write our `findLargest` method to find which object has the largest area instead.  

```Java title="findLargest v0.5 with GetAreable (Finding the Largest Object)" hl_lines="1 3 8 11"
GetAreable findLargest(GetAreable[] array) {
  double maxArea = 0;
  GetAreable maxObj = null;
  for (GetAreable curr : array) {
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
	  maxObj = curr;
    }
  }
  return maxObj;
}
```

Let's see how `findLargest` can be used:

```Java
GetAreable[] circles = new GetAreable[] {
  new Circle(new Point(1, 1), 2),
  new Circle(new Point(0, 0), 5)
};

GetAreable ga = findLargest(circles);  // ok
Circle c1 = findLargest(circles); // error
Circle c2 = (Circle) findLargest(circles); // ok
```

The return type of `findLargest` (version 0.5) is now `GetAreable`.  On Line 6 above, we assign the return object with a compile-time type of `GetAreable` to `ga`, which also has `GetAreable` as its compile-time type.  Since the variable `ga` is of type `GetAreable`, however, it is not very useful.  Recall that `GetAreable` is an interface with only one method `getArea`.  We cannot use it as a circle.

On Line 7, we try to return the return object to a variable with compile-time type `Circle`.  This line, however, causes a compile-time error.  Since `Circle` <: `GetAreable`, this is a narrowing type conversion and thus is not allowed (See [Unit 14](14-polymorphism.md)).  We will have to make an explicit cast of the result to `Circle` (on Line 8).  Only with casting, our code can compile and we get a reference with a compile-time type of `Circle`.

## Cast Carefully

Recall that, type casting, as we did in Line 8 above, is basically a way for programmers to ask the compiler to trust that the object returned by `findLargest` has a run-time type of `Circle` (or one of its subtype).
A cast does not change the object or convert it into another class. It only changes how the compiler allows the reference to be used. 

In the snippet above, we can be sure (even _prove_) that the returned object from `findLargest` must have a run-time type of `Circle` since the input variable `circles` contains only `Circle` objects.

The need to cast our returned object, however, leads to fragile code.  Since the correctness of Line 8 depends on the run-time type, the compiler cannot help us.  It is then up to the programmers to not make mistakes.

Consider the following two snippets, which will compile perfectly, but will lead to an error at run-time, when Java detects that the actual object is not an instance of the target class.

```Java
GetAreable[] circles = new GetAreable[] {
  new Circle(new Point(1, 1), 2),
  new Square(new Point(1, 1), 5)
};

Circle c2 = (Circle) findLargest(circles);
```

Or

```Java
GetAreable[] circles = new GetAreable[] {
  new Circle(new Point(1, 1), 2),
  new Circle(new Point(1, 1), 5)
};

Square sq = (Square) findLargest(circles);
```

We will see how to resolve this problem in later units, where we will show how Java’s type system (in particular, generics) allows us to express stronger guarantees so that many of these casts and the associated run-time risks can be avoided entirely.
