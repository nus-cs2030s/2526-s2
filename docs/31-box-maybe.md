# Unit 31: Box and Maybe

!!! abstract "Learning Objectives"

    After completing this unit, students should be able to:

    - explain how higher-order methods (e.g. map, filter) allow controlled manipulation of hidden state across an abstraction barrier
    - apply `Box<T>`-style abstractions to build general, reusable containers parameterized by both types and behavior
    - explain the role of `Maybe<T>` as an option type that preserves “missing value” semantics without exposing null
    - reason about how wrapping null inside `Maybe<T>` restores function purity by keeping return values within the declared codomain
    - use map and filter on `Maybe<T>` to eliminate explicit `null` checks and prevent `NullPointerException`

!!! abstract "Overview"

    In earlier units, we learned how abstraction barriers protect internal state, how immutability simplifies reasoning, and how lambda expressions allow behavior to be passed around as values. We also saw how higher-order methods such as map and filter let us transform data without exposing its representation.

    In this unit, we bring these ideas together by studying `Box<T>` and `Maybe<T>`. These classes show how a container can remain fully encapsulated while still being highly flexible, by allowing clients to operate on hidden values using functions rather than direct access. In particular, `Maybe<T>` addresses a long-standing problem in Java: how to represent missing values without breaking function purity or relying on fragile null checks.

    By the end of this unit, you will see how combining generics and lambda expressions enables us to build general, safe abstractions that scale beyond concrete data structures, and how `Maybe<T>` helps us write code that is both more expressive and less error-prone.

## Lambda as a Cross-Barrier State Manipulator

Recall that every class has an abstraction barrier between the client and the implementer.  The internal state of the class is protected and hidden.   The implementer selectively provides a set of methods to access and manipulate the internal states of instances.  This approach allows the implementer to control what the client can and cannot do to the internal states.  This is good if we want to build abstractions over specific entities such as shapes or data structures such as a stack, but it is not flexible enough to build general abstractions.

Let's consider the following class:

```Java
class Box<T> {
  private T item;
}
```

It is a box containing a single item of type `T`.  Suppose that we want to keep the `item` hidden and we want to have certain rules and maintain some semantics about the use of the `item`.  Therefore, we don't want to provide any setter or getter, so that the client cannot voilate these rules.  What are some ways we can still operate on this `item`?

The only way to operate on this hidden item is to provide methods that accept a lambda expression, apply the lambda expression on the item, and return the new box with the new value.  For instance,

```Java
class Box<T> {
  private T item;
    :

  public <U> Box<U> map(Transformer<? super T, ? extends U> transformer) {
    if (!isPresent()) {
      return empty();
    }
    return Box.ofNullable(transformer.transform(this.item));
  }
    :

  public Box<T> filter(BooleanCondition<? super T> condition) {
    if (!isPresent() || !(condition.test(this.item))) {
      return empty();
    }
    return this;
  }
    :
}
```

The method `map` takes in a lambda expression and allows us to arbitrarily apply a function to the item, while the method `filter` allows us to perform an arbitrary check on the property of the item.

Methods such as these, which accept a function as a parameter, allow the client to manipulate the data behind the abstraction barrier without knowing the internals of the object.  Here, we are treating lambda expressions as "manipulators" that we can pass in behind the abstraction barrier and modify the internals arbitrarily for us, while the container or the box tries to maintain the semantics for us.

## Maybe

Let's now look at `Box<T>` in a slightly different light.  Let's rename it to `Maybe<T>`.  `Maybe<T>` is an _option type_, a common abstraction in programming languages (`java.util.Optional` in Java, `option` in Scala, `Maybe` in Haskell, `Nullable<T>` in C#, etc) that is a wrapper around a value that is either there or is `null`.  The `Maybe<T>` abstraction allows us to write code without about the possibility that our value is missing.  When we call `map` on a value that is missing, nothing happens.

Recall that we wish to write a program that is as close to pure mathematical functions as possible, a mathematical function always has a well-defined domain and codomain.  If we have a method that looks like this:
```Java
Counter c = bank.findCounter();
```

Then `findCounter` is mapping from the domain of banks to counters.  However, if we implement `findCounter` such that it returns `null` if no counter is available, then `findCounter` is not a function anymore.  The return value `null` is not a counter, as we cannot do things that we can normally do on counters to it.  So `findCounter` now maps to a value outside its codomain!  This violation of the purity of function adds complications to our code, as we now have to specifically filter out `null` value, and is a common source of bugs.

One way to fix this is to have a special counter (say, `class NullCounter extends Counter`) that is returned whenever no counter is available.  This way, our `findCounter` remains a pure function.  But this is not a general solution.  If we adopt this solution, everywhere we return `null` in place of a non-null instance we have to create a special subclass.

Another way, that is more general, is to expand the codomain of the function to include `null`, and wrap both `null` and `Counter` under a type called `Maybe<Counter>`.  We make `findCounter` returns a `Maybe<Counter>` instead
```Java
Maybe<Counter> c = bank.findCounter();
```

With this design, `findCounter` is now a function with the domain `Bank` mapped to the codomain `Maybe<Counter>`, and it is pure.

Another way to view the `Maybe<T>` class is that it internalizes all the checks for `null` on the client's behalf.  `Maybe<T>` ensures that if `null` represents a missing value, then the semantics of this missing value is preserved throughout the chain of `map` and `filter` operations.  Within its implementation, `Maybe<T>` does the right thing when the value is missing to prevent us from encountering `NullPointerException`.  There is a check for `null` when needed, internally, within `Maybe<T>`.  This internalization removes the burden of checking for `null` on the programmer and removes the possibility of runtime crashes due to missing `null` checks.
