---
layout: post
title: "Java: exceptions in signature of overriden method"
subtitle: ""
date: 2018-07-24
categories: [programming]
---
An item from 'funny things' bag. Consider a class with method throwing unchecked exception:

```java
class Foo {
	void fizz() throws Exception {
	}
}
```

Then extend it and override the `fizz` method:

```java
class Bar extends Foo {
	@Override
	void fizz() {
	}
}

```

Notice no `Exception` in `Bar::fizz` method. It's ok. It is allowed to remove exceptions thrown by super class (but you can't add new ones). However the result is that the same method has different signatures in both classes, and this leads to funny behaviour like below:

```java
Bar bar = new Bar();
bar.fizz(); //Ok

((Foo)bar).fizz(); // Error: Unhandled exception: java.lang.Exception
```

One may get a little surprised.
