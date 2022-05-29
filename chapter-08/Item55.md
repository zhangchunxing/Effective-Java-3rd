# 谨慎返回Optional类型

在 Java 8之前，要编写一个在特定环境下无法返回任何值的方法时，有两种方法：要么抛出异常，要么返回`null`（假设返回类型是一个对象引用类型）。但这两种方法都不够完美。首先，异常应该用在异常条件下（条款69）。并且抛出异常的代价很高，因为在创建异常时会捕获整个堆栈。返回`null`没有这些缺点，但它也有自身的不足。如果方法返回`null`，客户端就必须包含特殊的代码来处理返回`nu11`的可能性，除非程序员能证明不可能返回`nul1`。如果客户端疏忽了，没有检查`null`返回值，并将`null`返回值保存在某个数据结构中，那么未来在与这个问题毫不相关的某处代码中，随时有可能发生`NullPointerException`异常。

在Java 中，还有第三种方法可以编写不能返回值的方法。`Optional<T＞`类代表的是一个不可变的容器，它可以存放单个非`null`引用，或者什么内容都没有。不包含任何内容的`optional`称为空（empty），非空的`optional`中的值称作存在（present）。`optional`本质上是一个不可变的集合，最多只能存放一个元素。`Optioal<T＞`没有实现`Collection<T>`接口，但原则上是可以的。

理论上能返回`T`的方法，在某些特定的条件下也可能无法返回，可以改为声明返回`Optional<T>`。这样做可以让方法返回一个空结果（`empty`)，表示无法返回一个有效的结果。方法返回`Optional` 比抛异常更灵活、更易使用，而且相对于返回`null`的方法更不容易出错。

在条款30中展示过下面这个方法，根据元素的自然顺序，计算集合中的最大值：

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

如果指定的集合为空，这个方法就会抛出`IllegalArgumentExceptio`。在条款30中说过，更好的替代方法是返回`Optional<E>`。下面就是返回修改之后的代码：

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

如上所示，返回`optional`是很简单的事。只要用适当的静态工厂创建`optional`即可。在这个程序中，我们使用了两个`optional`：`Optional.empty()`返回一个空的`optional`，`Optional.of(value)`返回一个包含了指定非`null`值的`optional`。将`null`传入`Optional.of(value)`是一个编程错误。如果这样做，该方法会抛出`NullPointerException`。`Optional.ofNullable(value)`方法接受可能为`null`的值，当传入`null`值时就返回一个空的`optional`。**永远不要在返回`Optional`对象的方法中去返回一个`null`值**：因为它彻底违背了`optional`的本意。

`Stream`的许多终止操作都返回`optional`如果重新用`stream` 编写`max` 方法，让`stream`的`max`操作替我们完成产生`optional`的工作（虽然还是需要传人一个显式的比较器）：

```java
// Returns max val in collection as Optional<E> - uses stream
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
		return c.stream().max(Comparator.naturalOrder());
}
```

那么，如何选择是返回`optional`，还是返回`null`，或是抛出异常呢？**`Optional`本质上与受检异常（条款71）相类似 **，因为它们强迫`API`的用户面对没有返回值的现实。抛出非受检异常，或者返回`null`，都允许用户忽略这种可能性，从而可能带来灾难性的后果。但是，抛出受检异常需要在客户端添加额外的样板代码。

如果方法返回`optional`，客户端必须做出选择：如果该方法不能返回值时应该采取什么动作 你可以指定一个默认值：

```java
// Using an optional to provide a chosen default value
String lastWordInLexicon = max(words).orElse("No words...");
```

你也可以抛出任何恰当的异常。注意此处传入的是一个异常工厂，而不是真实的异常。这避免了创建异常的开销，除非它真正抛出异常：

```java
// Using an optional to throw a chosen exception
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

如果你能够证明`optional`为非空，就不必指定如果`optional`为空要采取什么动作，直接从`optional`获得值即可；但是如果你判断错误，你的代码会抛出`NoSuchElementException`异常。

```java
// Using optional when you know there’s a return value
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

有时候，获取默认值的开销很高。除非十分必要，否则还是希望能够避免这一开销的。对于这类情况，`Optional`提供了一个带有`Supplier<T>`的方法，只有在必要的时候才调用它。这个方法被称为`orElseGet`，但也许它应该被称为`orElseCompute`，因为它与三个名称以`compute`开头的`Map`方法密切相关。还有几个`Optional`的方法用来处理更加特殊用例的情况：`filter`、`map`、`flatMap`和`ifPresent`。Java 9中，又新增了两个方法：`or`和`ifPresentOrElse`。如果上面的基本方法不能很好地满足你的需求，可以查看文档寻找更高级的方法，看看它们是否能胜任工作。

万一这些方法都无法满足需求， `Optional`还提供了`isPresent()`方法，它可以被当作是一个安全阀。当`optional`中包含一个值时，它返回`true`；当`optional`空时，返回`false`。该方法可用于对`optional`结果执行任意的处理，但要确保正确使用。`isPresent`的许多用法都可以用上述任意一种方法取代。这样得到的代码一般会更加简短、清晰，也更符合习惯用法。

例如，以下代码片段用于打印出一个进程的父进程ID ，当该进程没有父进程时打印`N/A`。 这里使用了在 Java9中引人的`ProcessHandle`类：

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("Parent PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));
```

上述代码片段可以用以下的代码代替，这里使用了`Optional`的`map`函数：

```java
System.out.println("Parent PID: " + ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"))
```

当用`Stream`编程时，经常会遇到`Stream<Optional<T>>`，其实我们想要的是包含了非空`Optional`中所有元素的`Stream<T>`。如果你使用的是Java 8，可以像这样弥补差距：

```java
streamOfOptionals.filter(Optional::isPresent).map(Optional::get)
```

在Java 9中，`Optional`还配有一个`stream()`方法。这个方法是一个适配器，如果`Optional`对象有值，它将`Optional`转换为包含这个值的`Stream`。如果`Optional`是空的，那么得到一个空的`Stream`。结合`Stream`的`flatMap`方法（条款45），可以简洁地取代上述代码片段，如下：

```java
streamOfOptionals.flatMap(Optional::stream)
```

但是，并非所有的返回类型都受益于`optional`的处理方法。**容器类型不应该被包装在`Optional`对象中。**比如：集合（collection）、映射（map）、流（stream）、数组（array）以及`Optional`。不要返回空的`Optional<List<T＞＞`，而应该只返回一个空的`List<T＞`（Item 54）。直接返回空的容器，客户端代码就不需要处理`Optional`对象。`ProcesHandle`类确实有`arguments`方法，它返回`Optional<String[]>`，但应该将此方法视为错误的示范，而不要去模仿它。

那什么时候你应该声明一个方法返回`Optional<T>`而不是`T`？作为一条规则，如果方法不能返回结果，并且如果没有返回结果，客户端不得不执行特殊处理，则应该声明该方法返回`Optional<T>`。也就是说，返回`Optional<T>`是有成本的。`Optional`是一个必须分配和初始化的对象，从`Optional`对象中读取值需要一个额外的间接方法。这使得`Optional`不适合在某些性能关键的场景下使用。一个特殊的方法是否属于此类，只能通过仔细的测量来确定才行（条款67）。

返回一个包含了基本包装类型的`Optional` ，比返回一个基本类型的开销更高，因为`Optional`有两级包装，不是0级。因此，类库的设计者认为必须为基本类型 `int` 、`long`和` double`供类似`Optional<T＞`的方法。这些`Optional`类型为`OptionalInt`、`OptionalLong`和`OptionalDouble`。因此， 永远不应该返回基本包装类型的`Optional`， ”小型的基本类型”（`Boolean`、` Byte`、` Character、` `Short`和` Float` ）除外。

到目前为止，我们已经讨论了返回`optional`，以及返回之后对它们的处理方法。之所以还没有讨论到其他可能的用途，是因为`optional`的大部分其他用途都还受到质疑。例如，永远不应该用`optional`作为映射值。如果这么做，有两种方式来表达一个键的逻辑缺失：要么这个键可以不出现在映射中；要么它可以存在，并映射到一个空的`optional`。这些既增加了无谓的复杂度，并极有可能造成混淆和出错。更通俗地说， **几乎永远都不适合用`optional`作为键、值，或者集合或数组中的元素**。

这里留下了一个尚未解答的问题：适合将`optional`保存在实例的字段中吗？这样做通常带来一股坏味道，如果建议使用包含`optional`的子类。不过有时候它又是有道理的。回想一下条款2中的`NutritionFacts`的例子。`NutritionFacts`实例中包含了许多不必要的字段，你不可能给这些字段中每一个可能的合并都提供一个子类。而且因为这些字段有基本类型，所以不方便直接地描述缺失。对于`NutritionFacts`，最好的`API`是每个可选字段的`get`方法返回`optional`。这样做后，这些可选字段简单地存储在对象中就有意义了。

总而言之，如果发现自己在编写的方法始终无法返回值，并且相信该方法的用户每次在调用它时都要考虑到这种可能性，那么或许就应该返回一个`optional`。但是，你也应该意识到，返回`optional`会带来实际的性能后果；对于注重性能的方法，最好是返回一个`null` ，或者抛出异常。最后，尽量不要将`optional`用作返回值以外的任何其他用途。



















