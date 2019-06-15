# 优先选择标准的函数式接口

现在Java有了lambdas，编写api的最佳实践已经发生了很大的变化。比如说，在模板方法模式中，子类重写一个原始方法，来特化父类的行为，这是很常见的。现在的可选方法是提供一个静态工厂或构造函数，它接受一个函数对象来达到同样的效果。更普遍地是，你将编写更多的接收函数对象作为参数的构造函数和方法。选择正确的函数参数类型需要谨慎。

考虑`LinkedHashMap`。你可以把它当做缓存，通过重写它的受保护的` removeEldestEntry`方法，每次有新的`key`添加到`Map`中时，这个方法会被`put`调用。当此方法返回true时，`Map`将删除传递给该方法的最老的条目。

下面的重写允许`Map`增长到100个条目，然后在每次添加一个新键时删除最老的条目，维护最近的100个条目：

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
	return size() > 100;
}
```

这种技术工作得很好，但是使用lambdas可以做得更好。如果`LinkedHashMap`是今天编写的，它将有一个接受函数对象的静态工厂或构造方法。看着`removeEldestEntry`的方法声明，你可能认为函数对象应该接受一个`Map<K,V>`，并且返回一个布尔值。但这并不能完全做到：`removeEldestEntry`方法调用`size()`来获得`Map`中的条目数量，这样做是因为`removeEldestEntry`是`Map`上的一个实例方法。传递给构造方法的函数对象不是`Map`上的实例方法，因此无法捕获它，因为在调用其工厂或构造函数时`Map`还不存在。因此，`Map`必须将自己传递给函数对象，函数对象因此必须在输入时接收`Map`和它的最老的条目。如果你要声明这样一个函数接口，它应该是这样的：

```java
// Unnecessary functional interface; use a standard one instead.
@FunctionalInterface
interface EldestEntryRemovalFunction<K,V> {
	boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

这个接口可以很好地工作，但是你不应该使用它，因为你不需要为此声明一个新的接口。`java.util.function`包提供了大量的标准函数式接口供你使用。**如果有一个标准函数式接口可以完成这项工作，那么通常应该优先使用它，而不是专门构建一个函数式接口**。通过减少API的概念层内容，这将使你的API更容易学习，并将提供重要的互操作性好处，因为许多标准函数式接口都提供了有用的默认方法。比如，`Predicate `接口提供了方法来组合谓词。在我们的`LinkedHashMap`例子中，应该优先使用标准的` BiPredicate<Map<K,V>`，` Map.Entry<K,V>>`接口，而不是自定义的`EldestEntryRemovalFunction`接口。

`java.util.Function`包下有43个接口。你不能期望记住所有的接口，但是如果你记住了6个基础接口，你就可以在需要时派生出其余的接口。这些基础接口操作于对象的引用类型。`Opertaion`接口表示结果和参数类型是相同的函数。`Predicate`接口表示接受参数并返回布尔值的函数。`Funcation`接口表示参数和返回类型不同的函数。

`Supplier`接口表示不接受参数并返回值的函数。最后，`Consumer`表示一个函数，该函数接受一个参数，但什么也不返回，本质上是消费它的参数。6大基础函数式接口概述如下：