---
layout: post
title: "Java: retrying execution"
subtitle:
date: 2017-11-10
categories: [programming]
---

A common problem is to retry execution when there are, for example connection problems. There is very nice library called [awaitility](https://github.com/awaitility/awaitility). However if use case is simple then simpler solution could be used. For example below reusable method:

```java
/**
 * Executes given action. When an exception occurs it tries to execute it again defined amount of times.
 *
 * @param supplier  - an action to be executed
 * @param tryCount  - maximum amount of executions. When set to 0 it will be repeated infinitely.
 * @param sleepTime - time between tries
 */
public static <T> T exec(ThrowingSupplier<T> supplier, int tryCount, int sleepTime) throws Exception {
    int tries = 0;
    while (true) {
        try {
            return supplier.get();
        } catch (Exception e) {
            if (++tries == tryCount) {
                throw e;
            }
            Thread.sleep(sleepTime);
        }
    }
}
```

So having a method `wyszybypszy()` that does some stuff:

```java
exec(() -> wyszybypszy(), 5, 5000);
```

Let me show you two more use cases: retrying on specific exception only and retrying with different argument set.

# Retrying only on specific exception

In some cases we could want to retry only in case of one specific exception, for example our method `wyszybypszy` throws `ConnectionTimeoutException`. Adding one additional argument `exceptionToRecover` and type check in catch clause:

```java
public static <T> T exec(
        ThrowingSupplier<T> supplier,
        Class exceptionToRecover,
        int tryCount,
        int sleepTime) throws Exception {
    int tries = 0;
    while (true) {
        try {
            return supplier.get();
        } catch (Exception e) {
            if (!exceptionToRecover.isInstance(e) || ++tries == tryCount) {
                throw e;
            }
            Thread.sleep(sleepTime);
        }
    }
}
```

And then we retry only when connection timeouts:

```java
exec(() -> foo(), ConnectionTimeoutException.class, 5, 5000);
```

# Retrying with different arguments

Another use case - we have given list of hosts. If connection to one host fails then pick next host and retry connection. 

```java
/**
 * Executes given function with first argument from argumentSequence list. When an exception is
 * thrown, given function is executed with next argument from argumentSequence list. If end of argumentSequence list
 * is reached and an exception is thrown then it will not be recovered.
 *
 * @param function           to be executed
 * @param argumentSequence   list of argument sets for function
 */
public static <T, R> R exec(List<T> argumentSequence, ThrowingFunction<T, R> function) throws Exception {
    for (int i = 0; i < argumentSequence.size(); ++i) {
        try {
            return function.apply(argumentSequence.get(i));
        } catch (Exception e) {
            if (i == argumentSequence.size() - 1) {
                throw e;
            }
        }
    }
    return null;
}
```

Assuming having method that is establishing the connection:

```java
Connection connect(String host){
    ...
}
```

Our retryer can be used like this:

```java
exec(Arrays.asList("host1", "host2", host -> connect(host));
```
