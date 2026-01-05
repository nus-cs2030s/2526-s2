# Unit 15: Method Invocation

!!! abstract "Learning Objectives"

    After this unit, the student should be able to:

    - explain the two-step method invocation process in Java (compile-time method selection and runtime method dispatch)
    - predict which method implementation is executed for a given invocation, and explain why class methods are statically bound

!!! abstract "Overview"

    In earlier units, we learned how overloading, overriding, and polymorphism allow a single method call to behave differently depending on the object it is invoked on. This enables flexible and extensible program designs, but it also raises an important question in a statically typed language like Java: If many decisions are made at compile time, how does Java support dynamic behavior at run time?

    This unit answers that question by explaining how Java resolves method invocations. We will see that method invocation is a two-step process: one step performed during compilation, and another during execution. Each step uses different information and determines different aspects of the method call.

    Understanding this process explains why overloading is resolved at compile time, why overriding supports dynamic binding, and why class methods do not participate in dynamic binding. By the end of this unit, method invocation will no longer seem magical, but a precise and predictable mechanism.

## How does Dynamic Binding work?

We have seen that, with the power of dynamic binding and polymorphism, we can write succinct, future-proof code.  Recall that example below, where the magic happens in Line 4.  The method invocation `curr.equals(obj)` will call the corresponding implementation of the `equals` method depending on the runtime type of `curr`.

```Java title="contains v0.1 with Polymorphism" hl_lines="3"
boolean contains(Object[] array, Object obj) {
  for (Object curr : array) {
    if (curr.equals(obj)) {
      return true;
    }
  }
  return false;
}
```

How does dynamic binding work?  To be more precise, when the method `equals` is invoked on the target `curr`, how does Java decide which method implementation is this invocation bound to?  While we have alluded to the fact that the runtime type of the target `curr` plays a role, this is not the entire story.  Recall that we may have multiple versions of `equals` due to overloading.  So, Java also needs to decide, among the overloaded `equals`, which version of `equals` this particular invocation is bound to.

To do so, Java splits the problem of method invocation into two separate questions:

- Which method signature is being called? (decided at compile time)
- Which class provides the implementation? (decided at run time)

These two decisions are made independently, using different types of information.

## During Compile Time

During compilation, Java determines the method descriptor of the method invoked, using the compile-time type of the target.

For example, in the line
```Java
boolean same = curr.equals(obj)
```
above, the target `curr` has the compile-time type `Object`.

Let's generalize the compile-time type of the target to $C$.  To determine the method descriptor, the compiler searches for all methods in $C$ (including inherited methods) that can be correctly invoked on the given argument and the given return type.  Methods inherited from $C$'s supertypes are included in the search.

In the example above, we look at the class `Object`, and there is only one method called `equals`.  The method can be correctly invoked with one argument of type `Object` and returns a `boolean`.  Therefore, the method descriptor `boolean equals(Object)` is chosen.

What if more than one methods can correctly accept the argument?  In this case, we choose the _most specific_ one.  Intuitively, a method $M$ is more specific than method $N$ if the arguments to $M$ can be passed to $N$ without compilation error.  You can also see it as preferring the ethod whose parameter types are the "closest" to the argument's compile-time type.  

For example, let's say a class `Circle` implements:

```Java
boolean equals(Circle c) { .. }

@Override
boolean equals(Object c) { .. }
```

Then, `equals(Circle)` is more specific than `equals(Object)`.  Every `Circle` is an `Object`, but not every `Object` is a `Circle`. Let's try to understand this using our definition of "more specific" above.

Consider the second part of the definition, _"if the arguments to $M$ can be passed to $N$ without compilation error"_. We need to find which arguments can be accepted by the methods we wish to compare.  In the case of `equals(Circle)`, it can accept an argument of compile-time type `Circle` (and all its subclasses), but not an argument of compile-time type `Object`.  On the other hand, `equals(Object)` can accept an argument of compile-time type `Object` and all its subclasses, including `Circle`.

Therefore, since all arguments to `equals(Circle)` can be passed to `equals(Object)` without compilation error, we say `equals(Circle)` is more specific than `equals(Object)`.

There is also the possibility that when comparing two methods, none of the two methods is more specific than the other.  For instance, given `S1` <: `T` and `S2` <: `T`, `foo(S1)` is not more specific than `foo(S2)` and `foo(S2)` is not more specific than `foo(S1)`.  If the Java compiler fails to determine a single most specific method, it will throw a compilation error.

Otherwise, once the Java compiler determines the most specific method, it stores the method's descriptor (return type and signature) in the generated bytecode.  This chosen descriptor will never change during runtime, only the _class_ that provides the implementation may change.

In the example above, the method descriptor `boolean equals(Object)` will be stored in the generated binaries.  Note that it does not include information about the class that implements this method.  The class from which this method implementation will be taken is determined in Step 2 during run time.

## During Run Time

During execution, when a method is invoked, the method descriptor from Step 1 is first retrieved.  Then, the runtime type of the target is determined.  Let the runtime type of the target be $R$.  Java then looks for an accessible method with the matching descriptor in $R$.  If no such method is found, the search will continue up the class hierarchy, first to the parent class of $R$, then to the grand-parent class of $R$, and so on, until we reach the root `Object`.  The first method implementation with a matching method descriptor found will be the one executed.

For example, let's consider the invocation in the highlighted line below again:

```Java title="v0.1 without Polymorphism" hl_lines="3"
boolean contains(Object[] array, Object obj) {
  for (Object curr : array) {
    if (curr.equals(obj)) {
      return true;
    }
  }
  return false;
}
```

Let's say that `curr` points to a `Circle` object during runtime.  Suppose that the `Circle` class does not override the method `equals` in `Object`.  As a result, Java cannot find a matching method descriptor `boolean equals(Object)` in the method `Circle`.  It then looks for the method in the parent of `Circle`, which is the class `Object`.  It finds the method `boolean Object::equals(Object)` with a matching descriptor.  Thus, the method `boolean Object::equals(Object)` is executed.

Now, suppose that `Circle` overrides the method `boolean Object::equals(Object)` with its own `boolean Circle::equals(Object)` method.  Since Java starts searching from the class `Circle`, it finds the method `boolean Circle::equals(Object)` that matches the descriptor.  In this case, `curr.target(obj)` will invoke the method `boolean Circle::equals(Object)` instead.

This search works because Java guarantees that a method that overrides another must have the compatible method descriptor, ensuring that the runtime lookup is type-safe.

## Invocation of Class Methods

The description above applies to instance methods.  Class methods, on the other hand, do not support dynamic binding.  The method to invoke is resolved statically during compile time and is fixed at compie time.  The same process in Step 1 is taken, but during run time, Java will always invoke the method defined in the compile-time type of the target, ignore the runtime type.
