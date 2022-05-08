# 谨慎地返回Optional类型

在Java 8之前，当编写在某些情况下不能返回值的方法时，你可以采用两种方法。你可以抛异常，也可以返回`null`（假设返回类型是对象引用类型）。但这两种方法都不够完美。首先，异常应该是用在异常条件下（条款69），并且抛出异常的代价很高，因为在创建异常时会捕获整个堆栈。返回`null`没有这些缺点，但它也有本身的缺点。如果一个方法返回`null`，那么客户端必须要有特定的代码，来处理可能返回`null`的情况，除非程序员可以证明永远不会返回`null`。如果客户端没有对空返回值做判断，并将空返回值存储在了某个数据结构中。那么在将来的某个时间点，在代码中的某个与问题无关的地方，可能会产生一个`NullPointerException`。

在Java 8中，还有第三种方法去编写不能返回值的方法。`Optional`类代表一个不可变的容器，它要么包含一个非空的`T`类型的引用，要么什么也不包含。`empty`表示不包含任何内容的`Optional` 。`present`表示一个非空的`Optional`。`Optional`本质上是一个不可变的容器，最多包含一个元素。`Optional<T>`没有实现`Collection<T>`接口，但原则上可以这么做。

如果有一个方法，正常情况下会返回`T`类型对象，但在某些情况下可能无法返回值，那么可以将其返回值声明为`Optional<T>`。这样做可以让方法返回一个空结果（`empty`)，表示无法返回一个有效的结果。方法返回`Optional` 比抛出异常更灵活、更易使用，而且比返回`null`更不容易出错。

在条款30中，我们给出了根据元素的自然顺序计算集合中的最大值的方法：

```java
// Returns maximum value in collection - throws exception if empty
public static <E extends Comparable<E>> E max(Collection<E> c) {
		if (c.isEmpty())
				throw new IllegalArgumentException("Empty collection");
  
		E result = null;
  
		for (E e : c)
				if (result == null || e.compareTo(result) > 0)
						result = Objects.requireNonNull(e);
  
		return result;
}

```

如果集合是空的，那么方法会抛出异常` IllegalArgumentException`。在条款30里，我们说了有一个更好的可替代方案，就是返回`Optional<T>`。修改后的代码：

```java
// Returns maximum value in collection as an Optional<E>
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
		if (c.isEmpty())
				return Optional.empty();
 
		E result = null;
  	for (E e : c)
				if (result == null || e.compareTo(result) > 0)
						result = Objects.requireNonNull(e);
		
  	return Optional.of(result);
}
```

如你所见，直接返回了一个`Optional`对象。你要做的就是使用合适的静态工厂来创建`Optional`。在这个程序中，我们一共使用了两个`Optional`对象：利用`Optional .empty()`返回一个空的`Optional`对象，利用`Optional .of(value)`返回一个包含指定的非空值的`Optional`对象。将`null`传递给`Optional.of(value)`是一个编程错误。如果这样做，该方法会抛出`NullPointerException`。`Optional.ofNullable(value)`方法可以接收`null`。如果传递`null`，该方法会返回一个空的`Optional`对象。永远不要在返回`Optional`对象的方法中去返回一个`null`值：这样做，就破坏了使用这个工具类的目的了。

流（`Stream`）的很多终止操作都返回了`Optional`。如果我们使用流的方式来重写`max`方法，`Stream`的`max`操作会返回一个`Optional`给我们（尽管我们必须传入一个显示的比较器）：

```java
// Returns max val in collection as Optional<E> - uses stream
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
		return c.stream().max(Comparator.naturalOrder());
}
```

对于是返回`Optional`，还是返回`null`或者抛出异常，我们应该如何选择？某种意义上来说，`Optional`与受检异常非常相似。因为他们都强迫`API`的使用者面对这样一个事实：可能没有返回值。而抛出非受检异常或返回`null`会让用户忽略这种可能事件，这可能会带来可怕的后果。但是，抛出受检异常需要在客户端代码中引入额外的样板代码。

如果一个方法返回`Optional`，客户端可以选择在方法不能返回值时采取什么操作。你可以指定一个默认值：

```java
// Using an optional to provide a chosen default value
String lastWordInLexicon = max(words).orElse("No words...");
```

你也可以抛任何恰当的异常。请注意，我们传递的是异常工厂，而不是真实的异常。这样就避免了创建异常的开销，除非它真的会被抛出：

```java
// Using an optional to throw a chosen exception
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

如果你能确定`Optional`里是有值的，那么直接从`Optional`中获取该值就好。而不用去判断`Optional`是否有值。但是如果你判断错误，你的代码会抛出`NoSuchElementException`异常。

```java
// Using optional when you know there’s a return value
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

