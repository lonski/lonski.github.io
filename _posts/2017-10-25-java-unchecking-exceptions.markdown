---
layout: post
title: "Java: Unchecking exceptions"
subtitle: 
date: 2017-10-25
categories: [programming]
---

Java provides two types of exceptions: checked and unchecked. Unchecked exception is not documented in method signature and method's user is not aware that it may throw, unless he checks its implementation or reads it in method's java doc. On the other hand checked exceptions are listed in method signature, and compiler forces the developer to handle them. 

However sometimes we don't want to handle them.  Sometimes we want to re-throw them and catch somewhere else, sometimes we may want to not catch them at all. The reason is irrelevant. Let's see how such exceptions could be unchecked without try-catch overload.

The way to uncheck a checked exception is to re-throw it as `RuntimeException` or its subclass. One of common exception is `IOException`. In the same package there is `UncheckedIOException`. A friend of mine says it is the proof that checked exceptions idea is a failure ;-). 

So you just catch it and rethrow:

```java
try{
  methodThatThrowsIoException();
}catch(IOException e){
   throw new UncheckedIOException(e);
}
```

Ah, so ugly and redundant `try-catch`. Five lines instead of single one. 

What about this one:

```java
void process(){
  List<Item> items = collectItems();
  items.forEach(this::processItem);
}
```

Cool, a nice for-each! But wait.. damn.. `processItem` throws a checked exception:

```java
void processItem(Item item) throws IOException {
  ...
}
```

You have to fall back to old fashioned for-loop:

```java
void process() throws IOException{
  List<Item> items = collectItems();
  for(Item item : items){
    processItem(item);
  }
}
```

Or some strange solutions like:

```java
void process() {
  List<Item> items = collectItems();
  items.forEach(item -> {
    try {
      processItem(item);
    } catch (IOException e) {
      throw new UncheckedIOException(e);
    }
  });
}
```

Not so nice anymore.. 

# Unchecker for the rescue

Consider a util that accepts a lambda that may throw checked exception, and then executes it and unchecks inside. The very basic example:

```java
@FunctionalInterface
interface ThrowingRunnable {
  void run() throws Exception;
}

void uncheck(ThrowingRunnable runnable) {
  try {
    runnable.run();
  } catch (Exception e) {
    throw new RuntimeException(e);
  }
}
```

And then its usage:

```java
uncheck(() -> methodThatThrows());
```

Another version, for second example above:

```java
@FunctionalInterface
public interface ThrowingConsumer<T> {
  void accept(T t) throws Exception;
}

<T> Consumer<T> uncheck(ThrowingConsumer<T> consumer) {
  return (T t) -> {
    try {
      consumer.accept(t);
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  };
}
```

And usage:

```java
void process(){
  List<Item> items = collectItems();
  items.forEach(uncheck(this::processItem));
}
```

Isn't it nicer? One can find unchecking all exceptions using `RuntimeException` a bad practice. Sure, in general it is better to use a specific exception. Above methods can be improved to support that:

```java
<T> Consumer<T> uncheck(ThrowingConsumer<T> consumer) {
  return (T t) -> {
    try {
      consumer.accept(t);
    } catch (Exception e) {
      throw createException(e, uncheckTo);
    }
  };
}

RuntimeException createException(Exception caughtException, Class<? extends RuntimeException> uncheckTo) {
  try {
    return uncheckTo.getConstructor(Exception.class).newInstance(caughtException);
  } catch (Exception failedToInstantiateRequestedException) {
    return new UncheckedException(caughtException);
  }
}
```

And then specific exception type could be added as second parameter:

```java
void process(){
  List<Item> items = collectItems();
  items.forEach(uncheck(this::processItem, SpecificException.class));
}
```

Since we don't want to swallow the causing exception, `SpecificException` should have have constructor with single parameter of type `Exception`. In above implementation it is required for proper instantiation.

# Packing it all together

I wrote a complete utility class named `Unchecker` that holds static methods to uncheck most functional interfaces. I attached it below, together with unit tests:

- [unchecker.zip (3.6KB)](/files/unchecker.zip)
