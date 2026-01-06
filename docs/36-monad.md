# Unit 36: Monad

!!! abstract "Learning Objectives"


    After this unit, students should be able to:

    - recognize common structural patterns shared by `Maybe<T>`, `Lazy<T>`, `InfiniteList<T>`, and `Loggable<T>`
    - explain the role of `of`, `map`, and `flatMap` in structuring computations with side information
    - state and apply the monad laws (left identity, right identity, associativity)
    - reason about why violating these laws leads to unintuitive or unsafe behavior
    - distinguish between functors and monads, and relate them to abstractions seen earlier in the course

!!! abstract "Overview"

    In earlier units, we introduced abstractions such as `Maybe<T>`, `Lazy<T>`, and `InfiniteList<T>` to structure computations that may fail, be deferred, or be infinite. Although these classes appear different, they all support a similar style of programming: values are wrapped together with additional information, and computations are chained using methods like map and flatMap.

    In this unit, we step back and study the common pattern behind these abstractions. We formalize what it means for such classes to be well behaved, introducing the notion of monads and the laws they must obey. Understanding these laws explains why our earlier designs work, why some seemingly small changes can break them, and how these ideas generalize beyond Java to other programming languages and paradigms.

## Generalizing `Loggable<T>`

We now have a class `Loggable<T>` with a `flatMap` method that allows us to operate on the value encapsulated inside, along with some "side information".  `Loggable<T>` follows a pattern that we have seen many times before.  We have seen this in `Maybe<T>` and `Lazy<T>`, and `InfiniteList<T>`.  Each of these classes has:

- an `of` method to initialize the value and side information.
- a `flatMap` method to update the value and side information.

Different classes above have different side information that is initialized, stored, and updated when we use the `of` and `flatMap` operations.  The class may also have other methods besides the two above.  Additionally, the methods may have different names.

| Container | Side-Information |
| --------- | ---------------- |
| `Maybe<T>` | The value might be there (i.e., `Some<T>`) or might not be there (i.e., `None`) |
| `Lazy<T>` | The value has been evaluated or not |
| `Loggable<T>` | The log describing the operations done on the value |

These classes follow certain patterns that make them well-behaved.  In particular, they behave predictably when created with `of` and chained with `flatMap`.  Such "well-behaved" classes are examples of a programming construct called _monads_.  

## Identity Element of Binary Operations

Before we examine what "well behaved" means, we first take a brief detour to look at algebraic structures in mathematics, particularly on the concept of _identity element_ of a binary operation.

Let's consider the addition operation $+$ on integers.  We know that $+$ is a binary operation that takes in two integers and produces another integer.  For any integer $x$, $0 + x = x$ and $x + 0 = x$.  The integer $0$ is called the _identity element_ of the binary operation $+$, because adding $0$ to any integer $x$ does not change the value of $x$.

Similarly, for the multiplication operation $\times$ on integers, we know that for any integer $x$, $1 \times x = x$ and $x \times 1 = x$.  The integer $1$ is the identity element of the binary operation $\times$. 

Now, consider the exponentiation operation on integers.  For any integer $x$, we have $x^1 = x$, but it is not true that $1^x = x$ (unless $x$ is 1).  So, 1 behaves like an identity element only when it is on the right side of the operation, but not on the left side.  In this case, we say that the 1 is the _right identity_ of exponentiation, but not the _left identity_.  

On the other hand, 0 is both the left and right identity of $+$; 1 is both the left and right identity of $\times$.

## Algebraic View of `of` and `flatMap`

To help with the explanation below, we will now view a monad as a pair $(x, c)$, where $x$ is the value and $c$ is the side information.  

The argument to `flatMap` is a lambda that maps $i$ to $(f(i), c_f)$, where $f(i)$ is the new value and $c_f$ is the new side information produced by applying the lambda to $i$.  The `flatMap` method applies the lambda to a target monad $(x, c)$ to produce a new monad $(f(x), c \oplus c_f)$, where $\oplus$ is an operation that combines the side information.

Notationally, we write the `flatMap` operation as:

$$(x, c) \text{ flatMap } \bigl(i \rightarrow (f(i), c_f)\bigr) = \bigl(f(x), c \oplus c_f\bigr)$$

As a concrete example, consider the `Maybe<T>` monad. Here, the side information is whether the value is present or not.  The operation $\oplus$ is defined such that if either side information indicates absence of value (`false`), the combined side information also indicates absence of value (`false`).  If both side information indicates the presence of value (`true`), the combined side information also indicates the presence of value (`true`).  In other words, $\oplus$ is the AND operation on binary values.

The `Loggable<T>` monad, on the other hand, has side information as a log string.  The operation $\oplus$ is string concatenation.

The `of` method of a monad takes a value and creates a new monad instance that encapsulates the value and initialize the side information.  We can view `of` as a function that maps from a value $x$ to a monad $(x, c_0)$, where $c_0$ is the initialized side information.

## Identity Laws

For a monad to be well-behaved, it must obey certain laws.  The first two laws are about identity.  It says that the initialized side information $c_0$ must be both the left identity and the right identity of the corresponding $\oplus$ operation on the side information.

For instance, in our `Loggable<T>`,
```Java
public static <T> Loggable<T> of(T value) {
  return new Loggable<>(value, "");
}
```

The logger is initialized with empty side information (e.g., empty string as a log message).  The empty string is the identity of string concatenation, because for any string `s`, `"" + s = s` and `s + "" = s`.

Now, let's consider the lambda that we wish to pass into `flatMap`  &mdash; such a lambda takes in a value, computes it, and wraps it in a "new" monad, together with the corresponding side information.  For instance,

```Java
Loggable<Integer> incrWithLog(int x) {
  return new Loggable<>(incr(x), "incr " + x + "; ");
}
```

What should we expect when we take a fresh new monad `Loggable.of(4)` and call `flatMap` with a function `incrWithLog`?  Since `Loggable.of(4)` is new with no operation performed on it yet, calling 
```Java
Loggable.of(4).flatMap(x -> incrWithLog(x)) 
```

should result in the same value exactly as calling `incrWithLog(4)`.  So, we expect that, after calling the above, we have a `Loggable` with a value of 5 and a log message of `"incr 4"`.

Our `of` method should not do anything extra to the value and side information &mdash; it should simply wrap the value 4 into the `Loggable`.  Our `flatMap` method should not do anything extra to the value and the side information, it should simply apply the given lambda expression to the value.

Now, suppose we take an instance of `Loggable`, called `logger`, that has already been operated on one or more times with `flatMap`, and contains some side information.  What should we expect when we call:
```Java
logger.flatMap(x -> Loggable.of(x))
```

Since `of` should initialize the side information as the identity, it should not change the given side information.  The `flatMap` above should do nothing and the expression above should be the same as `logger`.

What we have described above is called the _left identity law_ and the _right identity law_ of monads.  Using the notations earlier, 

The left identity law says:

$$(x, c_0) \text{ flatMap } \bigl(i \rightarrow (f(i), c)\bigr) = (f(x), c)$$

The right identity law says:

$$(x, c) \text{ flatMap } \bigl(i \rightarrow (i, c_0)\bigr) = (x, c)$$

The laws above hold because $c_0 \oplus c = c$ and $c \oplus c_0 = c$.

To express this in Java, let `Monad` be a type that is a monad and `monad` be an instance of it.

The left identity law says:

- `Monad.of(x).flatMap(i -> f(i))` must be the same as `f(x)`

The right identity law says:

- `monad.flatMap(i -> Monad.of(i))` must be the same as `monad`

## Maybe<T> is Partially Well-Behaved

We have seen that `Loggable<T>` obey the identity laws, with $\oplus$ being the string concatenation and the identity $c_0$ being the empty string.  `Maybe<T>` also obeys the identity laws, with $\oplus$ being the AND operation on binary values and $c_0$ being `true` (indicating presence of value).

Now consider `Maybe<T>`.  We have seen that $\oplus$ is the AND operation on binary values.  The AND operation has `true` as its identity, but not `false`.  This means that `Maybe<T>` only obeys the identity laws only when the initialized side information is `true`, i.e., when we create a `Some<T>` instance.  If we create a `None` instance, the identity laws do not hold.  

For instance, `Maybe.none().flatMap(x -> f(x))` equals to `Maybe.none()`, which is not always the same as `f(x)`.

In other words, `Maybe<T>` is a monad with respect to `Maybe::some` and `Maybe::flatMap` operations, but not with respect to `Maybe::none` and `Maybe::flatMap`.

## Associative Law

We now return to the original `incr` and `abs` functions.  To compose the functions, we can write `abs(incr(x))`, explicitly one function after another.  Or we can compose them as another function: 
```Java
int absIncr(int x) {
  return abs(incr(x));
}
```

and call it `absIncr(x)`.  The effects should be exactly the same.  It does not matter if we group the functions together into another function before applying it to a value x.

Recall that after we built our `Loggable` class, we were able to compose the functions `incr` and `abs` by chaining the `flatMap`:

```Java
Loggable.of(4)
        .flatMap(x -> incrWithLog(x))
        .flatMap(x -> absWithLog(x))
```

We should get the resulting value as `abs(incr(4))`, along with the appropriate log messages.

Another way to call `incr` and then `abs` is to write something like this:
```Java
Loggable<Integer> absIncrWithLog(int x) {
  return incrWithLog(x).flatMap(y -> absWithLog(y));
}
```

We have composed the methods `incrWithLog` and `absWithLog` and grouped them under another method.  Now, if we call:
```Java
Loggable.of(4)
    .flatMap(x -> absIncrWithLog(x))
```

The two expressions above must have exactly the same effect on the value and its log message.

This example leads us to the third law of monads: regardless of how we group those calls to `flatMap`, their behavior must be the same.  This law is called the _associative law_.  Using our notations earlier, the law says:

$$\begin{aligned}
\bigl((x, c) \text{ flatMap } (i \rightarrow (f(i), c_f))\bigr) \text{ flatMap } (j \rightarrow (g(j), c_g) =\\ 
(x, c) \text{ flatMap } (i \rightarrow \bigl((f(i), c_f) \text{ flatMap } (j \rightarrow (g(j), c_g)\bigr))
\end{aligned}
$$

If we unpack the notations above, we have the left hand side as:

$$
\begin{aligned}
\bigl((x, c) \;\text{flatMap}\; (i \to (f(i), c_f))\bigr) \;\text{flatMap}\; (j \to (g(j), c_g)) &= {}\\
(f(x), c \oplus c_f) \;\text{flatMap}\; (j \to (g(j), c_g)) &= {}\\
(g(f(x)), (c \oplus c_f) \oplus c_g) 
\end{aligned}
$$

and the right hand side as:

$$
\begin{aligned}
(x, c) \;\text{flatMap}\;
  \bigl(i \to ((f(i), c_f) \;\text{flatMap}\; (j \to (g(j), c_g)))\bigr) &= \\
  (x, c) \;\text{flatMap}\; \bigl(i \to (g(f(i)), c_f \oplus c_g)\bigr) &= \\
  (g(f(x)), c \oplus (c_f \oplus c_g))
\end{aligned}
$$

For both sides to be equivalent, the $\oplus$ operation on the side information must be associative, i.e., 

$$(c_1 \oplus c_2) \oplus c_3 = c_1 \oplus (c_2 \oplus c_3)$$

Expressing it in Java, the law is:

- `monad.flatMap(x -> f(x)).flatMap(x -> g(x))` must be the same as `monad.flatMap(x -> f(x).flatMap(y -> g(y)))`

In our examples, `Loggable<T>` is well-behaved, since string concatenation is associative.  `Maybe<T>` is also well-behaved, since the AND operation on binary values is associative.

## A Counter Example

Let's see why following the laws are important.

If our monads follow the laws above, we can safely write methods that receive a monad from others, operate on it, and return it to others.  We can also safely create a monad and pass it to the clients to operate on.  Our clients can then call our methods in any order and operate on the monads that we create, and the effect on its value and side information is as expected.

Let's try to make our `Loggable` misbehave a little.  Suppose we change our `Loggable<T>` to be as follows:

```Java hl_lines="12 17"
// version 0.3 (NOT a monad)
class Loggable<T> {
  private final T value;
  private final String log;

  private Loggable(T value, String log) {
    this.value = value;
    this.log = log;
  }

  public static <T> Loggable<T> of(T value) {
    return new Loggable<>(value, "Logging starts: ");
  }

  public <R> Loggable<R> flatMap(Transformer<? super T, ? extends Loggable<? extends R>> transformer) {
    Loggable<? extends R> logger = transformer.transform(this.value);
    return new Loggable(logger.value, logger.log + this.log + "\n");
  }

  public String toString() {
    return "value: " + this.value + ", log: " + this.log;
  }
}
```

Our `of` adds a little initialization message.  Our `flatMap` adds a little new line before appending with the given log message.  Now, our `Loggable<T>` is not that well-behaved anymore.

Suppose we have two methods `foo` and `bar`, both take in an `x` and perform a series of operations on `x`.  Both return us a `Loggable` instance on the final value and its log.

```Java
Loggable<Integer> foo(int x) {
  return Loggable.of(x)
                 .flatMap(...)
                 .flatMap(...)
                   :
  ;
}
Loggable<Integer> bar(int x) {
  return Loggable.of(x)
                 .flatMap(...)
                 .flatMap(...)
                   :
  ;
}
```

Now, we want to perform the sequence of operations done in `foo`, followed by the sequence of operations done in `bar`.  So we called:
```Java
foo(4).flatMap(x -> bar(x))
```

We will find that the string `"Logging starts"` appears twice in our logs and there is now an extra blank line in the log file!

## Functors

We will end this unit with a brief discussion on _functors_, another common abstraction in functional-style programming.  A functor is a simpler construction than a monad in that it only ensures lambdas can be applied sequentially to the value, without worrying about side information.

Recall that when we build our `Loggable<T>` abstraction, we add a `map` that only updates the value but changes nothing to the side information, i.e.,

$$(x, c) map (i \rightarrow f(i)) = (f(i), c)$$

One can think of a functor as an abstraction that supports `map`.

A functor needs to adhere to two laws:

- preserving identity: `functor.map(x -> x)` is the same as `functor`
- preserving composition: `functor.map(x -> f(x)).map(x -> g(x))` is the same as `functor.map(x -> g(f(x))`. 

Our classes from `cs2030s.fp`, `Lazy<T>`, `Maybe<T>`, and `InfiniteList<T>` are functors as well.

## Monads and Functors in Other Languages

Although we have explored monads through Java, monads are a language-independent abstraction for structuring computations that carry additional context or effects.

In purely functional languages such as Haskell, monads play a central role. Because functions in Haskell are not allowed to produce side effects directly, monads provide a disciplined way to model effects such as failures, state, and input/output.  In other languages such as Scala, monads appear in everyday programming.  Collections, futures, and other abstractions support operations analogous to `map` and `flatMap`, allowing programmers to write chained computations in a clear, imperative-looking style, while still relying on the same monadic laws underneath.

From a broader perspective, monads represent a shift in how we think about program design. Instead of scattering special cases, flags, or global state throughout our code, we localize complexity inside a well-defined abstraction. The monad laws then act as a contract, ensuring that code remains composable and refactorable.
