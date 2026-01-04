# Unit 21: Variance

!!! abstract "Learning Objectives"

    After completing this unit, students should be able to:

    - explain what variance means and distinguish between covariant, contravariant, and invariant type relationships
    - reason about subtype relationships involving complex types (e.g., arrays) based on the subtype relationships of their component types
    - explain why Java arrays are covariant and how this design choice can lead to run-time type errors despite successful compilation
    - predict when Java’s type system can and cannot prevent unsafe operations involving arrays

## Introduction

In earlier units, we learned how Java establishes subtype relationships between classes and interfaces through inheritance and implementation. These relationships allow polymorphism: a variable of a supertype can refer to an object of a subtype.

However, once we start working with collections of objects, such as arrays, the situation becomes more subtle. Even if one type is a subtype of another, it is no longer obvious whether an array of the first type should be considered a subtype of an array of the second.

This unit introduces the notion of variance, which describes how subtype relationships between component types affect subtype relationships between complex types. We will see that Java makes arrays covariant, a decision that enables flexibility but also introduces a form of unsoundness that only manifests at run time.

Understanding this trade-off is essential for appreciating both Java’s type system design and the motivation behind safer alternatives introduced later in the course.


## Arrays of Reference Types

Both the methods `findLargest` and `contains` take in an array of reference types as parameters:

```Java title="findLargest v0.5 with GetAreable (Finding the Largest Object)"
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

What kinds of arrays can we pass into these methods?  Let's try this:
```Java
Object[] objArray = new Object[] { Integer.valueOf(1), Integer.valueOf(2) };
Integer[] intArray = new Integer[] { Integer.valueOf(1), Integer.valueOf(2) };

contains(objArray, Integer.valueOf(1)); // ok
contains(intArray, Integer.valueOf(1)); // ok
```

Line 4 is not surprising since the type for `objArray` matches that of the parameter `array`.  Line 5, however, shows that it is possible to assign a reference to an object with run-time type `Integer[]` to a variable with compile-time type `Object[]`.

## Variance of Types

So far, we have established the subtype relationship between classes and interfaces based on inheritance and implementation.  The subtype relationship between _complex types_ such as arrays, however, is not so trivial.  Let's look at some definitions.

The _variance of types_ refers to how the subtype relationship between complex types relates to the subtype relationship between components.

Let $C(S)$ correspond to some complex type based on type $S$.  An array of type $S$ is an example of a complex type.

We say a complex type is:

- _covariant_ if $S <: T$ implies $C(S) <: C(T)$
- _contravariant_ if $S <: T$ implies $C(T) <: C(S)$
- _invariant_ if it is neither covariant nor contravariant.

## Java Array is Covariant

 Arrays of reference types are covariant in Java[^1].  This means that, if $S <: T$, then $S[] <: T[]$.  

[^1]: Arrays of primitive types are invariant.

For example, because `Integer` <: `Object`, we have `Integer[]` <: `Object[]` and we can do the following:

```Java
Integer[] intArray;
Object[] objArray;
objArray = intArray; // ok
```

By making array covariant, however, Java opens up the possibility of run-time errors, even without typecasting!

Consider the following code:
```Java
Integer[] intArray = new Integer[2] {
  Integer.valueOf(10), Integer.valueOf(20)
};
Object[] objArray;
objArray = intArray;
objArray[0] = "Hello!"; // <- compiles!
```

On Line 5 above, we set `objArray` (with a compile-time type of `Object[]`) to refer to an object with a run-time type of `Integer[]`.  This is allowed since the array is covariant.

On Line 6, we try to put a `String` object into the `Object` array.  Since `String` <: `Object`, the compiler allows this.  The compiler does not realize that at run-time, the `Object` array will refer to an array of `Integer`.  

So we now have a perfectly compilable code, that will crash on us with an `ArrayStoreException` when it executes Line 6 &mdash; only then would Java realize that we are trying to stuff a string into an array of integers!

This is an example of a type system rule that is unsafe.  In other words, covariance of arrays breaks the guarantee that “well-typed programs do not go wrong.” Java compensates by inserting run-time checks, shifting some type safety from compile time to run time.

Since the array type is an essential part of the Java language, this rule cannot be changed without ruining existing code.  We will see later that Java’s generic types (such as List<T>) avoid this pitfall by not being covariant by default, trading flexibility for stronger compile-time guarantees.

