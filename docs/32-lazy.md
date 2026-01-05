# Unit 32: Lazy Evaluation

!!! abstract "Learning Objectives"

    After this unit, students should be able to:

    - explain the difference between eager and lazy evaluation, and how lambda expressions can be used to delay computation in Java;
    - use functional interfaces (such as `Producer<T>`) to defer evaluation and avoid unnecessary computation;
    - implement memoization using a `Lazy<T>` abstraction and reason about when such laziness is safe and beneficial.

!!! abstract "Overview"

    In earlier units, we learned how lambda expression allows us to treat behavior as data, enabling higher-order functions and more declarative code. In this unit, we build on that idea to explore *when* computations are performed.

    By default, Java evaluates expressions eagerly — values are computed immediately, even if they are never used. Lazy evaluation delays computation until the result is actually needed, allowing us to avoid unnecessary work and improve efficiency. We will see how lambda expressions, together with simple abstractions such as `Producer<T>` and `Lazy<T>`, allow us to implement laziness explicitly in Java.

## Lambda as Delayed Data

When we write a lambda expression like this:
```Java
Transformer<Integer, Integer> f = x -> x + 1;
```

we are just defining a function rather than invoke it.  This is clear because invoking the function requires an argument for `x`, and no argument is supplied when we define `f`.

Consider the following functional interfaces instead:
```Java
@FunctionalInterface
interface Producer<T> {
  T produce();
}

@FunctionalInterface
interface Task {
  void run();
}
```

These functional interfaces have a method that does not take in a parameter.  So, we would use them as follows:

```Java
i = 4;
Task print = () -> System.out.println(i);
Producer<String> toStr = () -> Integer.toString(i);
```

Keep in mind that the lambda expressions assigned to `print` and `toStr` are not executed when they are declared.  We are just saving them to be executed later.

Lambda expressions, therefore, allows us to delay the execution of code, saving them until they are needed.  This enables another powerful mechanism called _lazy evaluation_.  We can build up a sequence of complex computations, without actually executing them, until we need to.  Expressions are evaluated on demand when their values are required.

Consider the following class:

```Java
// Version 0.1 (eager evaluation)

class Logger {
  enum LogLevel { INFO, WARNING, ERROR };

  public static LogLevel currLogLevel = LogLevel.WARNING;

  static void log(LogLevel level, String msg) {
    if (level.compareTo(Logger.currLogLevel) >= 0) {
      System.out.println(" [" + level + "] " + msg);
    }
  }
}
```

The `log` method checks the log level (i.e, how serious is the message) of the message against the current log level and only prints the message if the level of the message is the same or higher.  For instance, if the current log level is `WARNING`, then

```Java
Logger.log(Logger.LogLevel.INFO, 
    "User " + System.getProperty("user.name") + " has logged in");
```

will not get printed.

However, regardless of whether the log message will be printed, the method `System.getProperty("user.name")` will be evaluated, which results in unnecessary computation.

A better design is to wrap the message `msg` within a lambda expression so that it does not get evaluated eagerly when we pass it in as a parameter.  We can wrap the message with a `Producer<String>`.  The new `lazyLog` method looks like this:

```Java
// Version 0.2 (with Producer)

class Logger {
  enum LogLevel { INFO, WARNING, ERROR };

  public static LogLevel currLogLevel = LogLevel.WARNING;

  static void lazyLog(LogLevel level, Producer<String> msg) {
    if (level.compareTo(Logger.currLogLevel) >= 0) {
	  System.out.println(" [" + level + "] " + msg.produce());
    }
  }
}
```

and is invoked like this:
```Java
Logger.lazyLog(Logger.LogLevel.INFO, 
    () -> "User " + System.getProperty("user.name") + " has logged in");
```

The method `System.getProperty("user.name")` is now lazily called, only if the message is going to be printed.

## Memoization

We have so far seen one way of being lazy, i.e., procrastinating our computation until we really need the data.  Another way of being lazy is to avoid repeated computation.  If we have computed the value of a function before, we can cache (or memoize) the value and reuse it.  Memoization is useful only if the function is pure &mdash; it always returns the same value and has no side effects.  Here, we see another important advantage of keeping our code pure and free of side effects &mdash; so that we can be lazy!

While other languages such as Scala has native support for lazy variables, Java does not.  So let's build a simple one here.  (You will build a more sophisticated one in Lab 6) 

```Java
class Lazy<T> {
  private T value;
  private boolean evaluated;
  private Producer<T> producer;

  public Lazy(Producer<T> producer) {
    evaluated = false;
    value = null;
	this.producer = producer;
  }

  public T get() {
	if (!evaluated) {
	  value = producer.produce();
	  evaluated = true;
	}
	return value;
  }
}
```

We can now rewrite our `Logger` as

```Java
// version 0.3 (with Lazy)

class Logger {
  enum LogLevel { INFO, WARNING, ERROR };

  public static LogLevel currLogLevel = LogLevel.WARNING;

  static void lazyLog(LogLevel level, Lazy<String> msg) {
    if (level.compareTo(Logger.currLogLevel) >= 0) {
	  System.out.println(" [" + level + "] " + msg.get());
    }
  }
}
```

and call it as follows:
```Java
Lazy<String> loginMessage = new Lazy(
    () -> "User " + System.getProperty("user.name") + " has logged in");
Logger.lazyLog(Logger.LogLevel.INFO, loginMessage);
```

If `loginMessage` is used in multiple places, memoization ensures that `System.getProperty("user.name")` and the string concatenation are performed only once.
