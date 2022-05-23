# 谨慎地返回Optional类型

在 Java 8之前，要编写一个在特定环境下无法返回任何值的方法时，有两种方法：要么抛出异常，要么返回`null`（假设返回类型是一个对象引用类型）。但这两种方法都不够完美。首先，异常应该用在异常条件下（条款69）。并且抛出异常的代价很高，因为在创建异常时会捕获整个堆栈。返回`null`没有这些缺点，但它也有自身的不足。如果方法返回`null`，客户端就必须包含特殊的代码来处理返回`nu11`的可能性，除非程序员能证明不可能返回`nul1`。如果客户端疏忽了，没有检查`null`返回值，并将`null`返回值保存在某个数据结构中，那么未来在与这个问题毫不相关的某处代码中，随时有可能发生`NullPointerException`异常。

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

如你所见，直接返回了一个`Optional`对象。你要做的就是使用合适的静态工厂来创建`Optional`。在这个程序中，我们一共使用了两个`Optional`对象：利用`Optional.empty()`返回一个空的`Optional`对象，利用`Optional.of(value)`返回一个包含指定的非空值的`Optional`对象。将`null`传递给`Optional.of(value)`是一个编程错误。如果这样做，该方法会抛出`NullPointerException`。`Optional.ofNullable(value)`方法可以接收`null`。如果传递`null`，该方法会返回一个空的`Optional`对象。永远不要在返回`Optional`对象的方法中去返回一个`null`值：这样做，就破坏了使用这个工具类的目的了。

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

有时候，你可能会遇到获取默认值的代价很高的情况。除非必要，否则你希望避免这种代价。对于这些情况，`Optional`提供了参数为`Supplier<T>`的方法，并仅在必要时调用它。这个方法被称为`orElseGet`，但也许它应该被称为`orElseCompute`，因为它与三个名称以`compute`开头的`Map`方法密切相关。还有几个`Optional`的方法用于处理更多特定的用例：`filter`、`map`、`flatMap`和`ifPresent`。在Java 9中，添加了另外两个方法：`or`和`ifPresentOrElse`。如果上面的基本方法不能很好的满足你的需求，请查看这些更高级的方法的文档，看看它们是否能胜任工作。

如果这些方法都不能满足你的需求，`Optional`提供了`isPresent()`方法，它可以被视为一个安全阀。如果`Optional`包含值，则返回`true`，如果为空则返回`false`。你可以使用这个方法对一个`Optional`结果执行任何你喜欢的处理，但要确保明智地使用它。`isPresent`的许多用途都可以用上面提到的一种方法来代替。最终的代码通常会更简洁、更清晰、更自然。

例如，这个代码片段，它打印一个进程的父进程的进程ID，或者如果该进程没有父进程，则输出`N/ a`。代码片段使用了在Java 9中引入的`ProcessHandle`类：

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("Parent PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));
```

上面的代码片段可以替换为下面的代码片段，它使用了`Optional`的`map`函数：

```java
System.out.println("Parent PID: " + ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"))
```

当使用`Stream`进行编程时，经常会发现自己得到的是`Stream<Optional<T>>`，其实我们想要的是一个包含了所有非空`Optional`里的元素的`Stream<T>`。如果你使用的是Java 8，下面是如何转换的方法：

```java
streamOfOptionals.filter(Optional::isPresent).map(Optional::get)
```

在Java 9中，`Optional`配备了一个`stream()`方法。这个方法是一个适配器，如果`Optional`对象包含值，它将`Optional`转换为包含这个值的`Stream`。如果`Optional`是空的，那么得到一个空的`Stream`。结合`Stream`的`flatMap`方法（条款45），该方法为上面的代码片段提供了一个简洁的替换：

```java
streamOfOptionals.flatMap(Optional::stream)
```

并不是所有返回类型都能从`Optional`中受益。**容器类型不应该被包装在`Optional`对象中。**比如：集合（collection）、映射（map）、流（stream）、数组（array）以及`Optional`。与其返回空的`Optional<List<T>>`，不如简单地返回空的`List<T> `（Item 54）。返回空的容器，客户端代码就不需要处理`Optional`对象。`ProcesHandle`类确实有`arguments`方法，它返回`Optional<String[]>`，但应该将此方法视为错误的示范，而不要去模仿它。

那什么时候你应该声明一个方法返回`Optional<T>`而不是`T`？作为一条规则，如果方法不能返回结果，并且如果没有返回结果，客户端不得不执行特殊处理，则应该声明该方法返回`Optional<T>`。也就是说，返回`Optional<T>`是有成本的。`Optional`是一个必须分配和初始化的对象，从`Optional`对象中读取值需要一个额外的间接方法。这使得`Optional`不适合在某些性能关键的场景下使用。一个特殊的方法是否属于此类，只能通过仔细的测量来确定才行（条款67）。

返回一个包含了基本包装类型的`Optional` ，比返回一个基本类型的开销更高，因为`Optional`有两级包装，不是0级。因此，类库的设计者认为必须为基本类型 `int` 、`long`和` double`供类似`Optional<T＞`的方法。这些`Optional`类型为`OptionalInt`、`OptionalLong`和`OptionalDouble`。因此， 永远不应该返回基本包装类型的`Optional`， ”小型的基本类型”（`Boolean`、` Byte`、` Character、` `Short`和` Float` ）除外。

到目前为止，我们已经讨论了返回`optional`，以及返回之后对它们的处理方法。之所以还没有讨论到其他可能的用途，是因为`optional`的大部分其他用途都还受到质疑。例如，永远不应该用`optional`作为映射值。如果这么做，有两种方式来表达一个键的逻辑缺失：要么这个键可以不出现在映射中；要么它可以存在，并映射到一个空的`optional`。这些既增加了无谓的复杂度，并极有可能造成混淆和出错。更通俗地说， **几乎永远都不适合用`optional`作为键、值，或者集合或数组中的元素**。

这里留下了一个尚未解答的问题：适合将`optional`保存在实例的字段中吗？这样做通常带来一股坏味道，如果建议使用包含`optional`的子类。不过有时候它又是有道理的。回想一下条款2中的`NutritionFacts`的例子。`NutritionFacts`实例中包含了许多不必要的字段，你不可能给这些字段中每一个可能的合并都提供一个子类。而且因为这些字段有基本类型，所以不方便直接地描述缺失。对于`NutritionFacts`，最好的`API`是每个可选字段的`get`方法返回`optional`。这样做后，这些可选字段简单地存储在对象中就有意义了。

总而言之，如果发现自己在编写的方法始终无法返回值，并且相信该方法的用户每次在调用它时都要考虑到这种可能性，那么或许就应该返回一个`optional`。但是，你也应该意识到，返回`optional`会带来实际的性能后果；对于注重性能的方法，最好是返回一个`null` ，或者抛出异常。最后，尽量不要将`optional`用作返回值以外的任何其他用途。



















