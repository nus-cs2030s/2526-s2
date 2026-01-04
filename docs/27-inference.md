# Unit 27: Type Inference

!!! abstract "Learning Objectives"


    After this unit, students should be able to:

    - explain what type inference is and why Java supports it for generic methods and types
    - identify the sources of type constraints (method arguments, bounds, and target typing) used during inference
    - manually derive the inferred type arguments in common generic method calls
    - recognize situations where type inference leads to surprising or unsafe behaviour
    - diagnose and explain compilation errors caused by incompatible inference bounds

## Introduction

In earlier units, we saw how Java’s type system helps prevent many classes of run-time errors by enforcing type correctness at compile time. We also learned that generic types and wildcards allow us to write flexible and reusable code—but often at the cost of additional type annotations.

To reduce verbosity, Java allows programmers to omit some type arguments and rely on the compiler to infer them automatically. This process, known as type inference, attempts to determine which type arguments would make the program type-correct.

While type inference can make code shorter and easier to read, it is not merely a convenience feature. It follows precise rules based on subtyping, bounds, and target types—and these rules can sometimes lead to results that surprise even experienced programmers.

In this unit, we study how Java infers type arguments for generic methods and types, how these inferences are derived from constraints, and why understanding the inference process is essential for writing safe and predictable generic code.

## Diamond Operator

One example of type inference is the diamond operator `<>` when we `new` an instance of a generic type:
```Java
Pair<String,Integer> p = new Pair<>();
```

Java can infer that `p` should be an instance of `Pair<String,Integer>` since the compile-time type of `p` is `Pair<String,Integer>`.  The line above is equivalent to:
```Java
Pair<String,Integer> p = new Pair<String,Integer>();
```

## Type Inferencing

We have been invoking 
```Java title="contains v0.7 (with wild cards)"
class A {
  // version 0.7 (with wild cards sequence)
  public static <S> boolean contains(Seq<? extends S> seq, S obj) {
    for (int i = 0; i < seq.getLength(); i++) {
      S curr = seq.get(i);
      if (curr.equals(obj)) {
        return true;
      }
    }
    return false;
  }
}
```

by explicitly passing in the type argument `Shape` (also called _type witness_ in the context of type inference).
```Java
     A.<Shape>contains(circleSeq, shape);
```

We could remove the type argument `<Shape>` so that we can call `contains` just like a non-generic method:
```Java
     A.contains(circleSeq, shape);
```

and Java could still infer that `S` should be `Shape`.  The type inference process looks for all possible types that match.  In this example, the type of the two arguments must match.  Let's consider each individually first:

- An object of type `Shape` is passed as an argument to the parameter `obj`.  So `S` might be `Shape` or, if widening type conversion has occurred, one of the other supertypes of `Shape`. Therefore, `Shape <: S <: Object`.
- A `Seq<Circle>` has been passed into `Seq<? extends S>`.  A widening type conversion occurred here, so we need to find all possible `S` such that `Seq<Circle>` <: `Seq<? extends S>`.  This is true only if `S` is `Circle`, or another supertype of `Circle`. Therefore, `Circle <: S <: Object`.

Solving for these two constraints on `S`, we get the following:
```
Shape <: S <: Object 
```
 
Therefore, `S` could be `Shape` or one of its supertypes: `GetAreable` and `Object`.   We choose the lower bound, so `S` is inferred to be `Shape`.

Type inference can have unexpected consequences.  Let's consider an [older version of `contains` that we wrote](23-generics.md):

```Java title="contains v0.4 (with generics)"
class A {
    public static <T> boolean contains(T[] array, T obj) {
      for (T curr : array) {
        if (curr.equals(obj)) {
          return true;
        }
      }
      return false;
    }
}
```

Recall that we want to prevent nonsensical calls where we are searching for an integer in an array of strings.
```Java
String[] strArray = new String[] { "hello", "world" };
A.<String>contains(strArray, 123); // type mismatch error
```

But, if we write:
```Java
A.contains(strArray, 123); // ok!  (huh?)
```

The code compiles!  Let's go through the type inference steps to understand what happened.  Again, we have two parameters:

- `strArray` has the type `String[]` and is passed to `T[]`.  So `T` must be `String` or its superclass `Object` (i.e. `String <: T <: Object`).  The latter is possible since Java array is covariant.
- `123` is passed as type `T`.  The value is treated as `Integer` and, therefore, `T` must be either `Integer`,  or its superclasses `Number`, and `Object` (i.e. `Integer <: T <: Object`). 

Solving for these two constraints:
```
T <: Object
```
Therefore `T` can only have the type `Object`, so Java infers `T` to be `Object`.  The code above is equivalent to:

```Java
A.<Object>contains(strArray, 123);
```

And our version 0.4 of `contains` actually is quite fragile and does not work as intended.  We were bitten _again_ by the fact that the Java array is covariant.

Type inference does not guarantee that the inferred type matches the programmer's intention. When multiple types satisfy the constraints, Java chooses the most general one that satisfies all bounds, even if that makes the method semantically meaningless.  Explicit type witnesses override inference and can be used to document intent or avoid surprising inferences. However, they do not bypass type checking, only inference.

## Target Typing

The example above performs type inference on the parameters of the generic methods.  Type inference can involve the type of the expression as well.  This is known as _target typing_.  Take the following upgraded version of `findLargest`:

```Java title="findLargest v0.6 (with Seq&lt;T&gt;)"
public static <T extends GetAreable> T findLargest(Seq<? extends T> seq) {
  double maxArea = 0;
  T maxObj = null;
  for (int i = 0; i < seq.getLength(); i++) {
    T curr = seq.get(i);
    double area = curr.getArea();
    if (area > maxArea) {
      maxArea = area;
      maxObj = curr;
    }
  }
  return maxObj;
}
```

and we call
```Java
Shape o = A.findLargest(new Seq<Circle>(0));
```

We have a few more constraints to check:

- Due to target typing, the return type of `T` must be a subtype of `Shape` (i.e. `T <: Shape`)
- Due to the bound of the type parameter, `T` must be a subtype of `GetAreable` (i.e. `T <: GetAreable`)
- `Seq<Circle>` must be a subtype of `Seq<? extends T>`, so `T` must be a supertype of `Circle` (i.e. `Circle <: T <: Object`)

Solving for all three of these constraints:
```
Circle <: T <: Shape
```

The lower bound is `Circle`, so the call above is equivalent to:
```Java
Shape o = A.<Circle>findLargest(new Seq<Circle>(0));
```

## Further Type Inference Examples

We now return to our `Circle` and `ColoredCircle` classes and the `GetAreable` interface. Recall that `Circle` implements `GetAreable` and `ColoredCircle` inherits from `Circle`.

Now lets consider the following method signature of a generic method `foo`:

```Java
public <T extends Circle> T foo(Seq<? extends T> seq)
```

Then we consider the following code excerpt:

```Java
ColoredCircle c = foo(new Seq<GetAreable>());
```

What does the java compiler infer `T` to be? Let's look at all of the constraints on `T`.

- First, the return type of `foo` must be a subtype of `ColoredCircle`, therefore `T <: ColoredCircle`.

- `T` is also a bounded type parameter, therefore `T <: Circle`.

- Our method argument is of type `Seq<GetAreable>` and must be a subtype of `Seq<? extends T>`, so `T` must be a supertype of `GetAreable` (i.e. `GetAreable <: T <: Object`).

We can see that there is no solution to our contraints, `T` can not be both a subtype of `ColoredCircle` and a supertype of `GetAreable` and therefore the Java compiler can not find a type `T`. The Java compiler will throw an error stating the inference variable `T` has incompatible bounds.

Lets consider, one final example using the following method signature of a generic method `bar`:

```Java
public <T extends Circle> T bar(Seq<? super T> seq)
```

Then we consider the following code excerpt:

```Java
GetAreable c = bar(new Seq<Circle>());
```

What does the java compiler infer `T` to be? Again, lets look at all of the constraints on `T`.

- We can say that the return type of `bar` must be a subtype of `GetAreable`, therefore `T <: GetAreable`.

- Our method argument is of type `Seq<Circle>` and must be a subtype of `Seq<? super T>`, so `T` must be a subtype of `Circle` (i.e. `T <: Circle`).

Solving for these two constraints:
```
T <: Circle
```

Whilst `ColoredCircle` is also a subtype of `Circle` it is not included in the above statement and therefore the compiler does not consider this class during type inference. Indeed, the compiler cannot be aware[^1] of all subtypes of `Circle` and there could be more than one subtype. Therefore `T` can only have the type `Circle`, so Java infers `T` to be `Circle`. 

 [^1]: Due to evolving specifications of software, at the time of compilation, a subtype may not have even been conceived of or written yet!

## Rules for Type Inference

We now summarize the steps for type inference. First, we figure out all of the type constraints on our type parameters, and then we solve these constraints. If no type can satisfy all the constraints, Java will fail to compile. If in resolving the type constraints for a given type parameter `T` we are left with:

- `Type1 <: T <: Type2`, then `T` is inferred as `Type1`
- `Type1 <: T`[^2], then `T` is inferred as `Type1`
- `T <: Type2`, then `T` is inferred as `Type2`

where `Type1` and `Type2` are arbitrary types. Java prefers the lower bound when both bounds are present, as it leads to more specific types and better type safety.  If only one bound is present, Java uses that bound to infer the type.

[^2]: Note that `T <: Object` is implicit here. We can see that this case could also be written as `Type1 <: T <: Object`, and would therefore also be explained by the previous case (`Type1 <: T <: Type2`).

!!! notes "Fresh Type Variables and Captured Wildcards"

    In more complex scenarios, Java may introduce _fresh type variables_ or _capture wildcards_ during type inference to handle cases where the exact type cannot be determined directly. These mechanisms allow Java to maintain type safety while still providing flexibility in generic programming. However, these topics are beyond the scope of this unit and will be covered in more advanced discussions on Java's type system.
