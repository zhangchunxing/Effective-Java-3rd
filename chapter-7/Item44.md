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

