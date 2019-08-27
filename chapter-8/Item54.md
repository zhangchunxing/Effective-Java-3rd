# 返回空集合或是数组而非null

像下面这样的方法是经常可见的：

```java
// Returns null to indicate an empty collection. Don’t do this!
private final List<Cheese> cheesesInStock = ...;
/**
* @return a list containing all of the cheeses in the shop,
* or null if no cheeses are available for purchase.
*/
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

没有理由对没有奶酪可供购买的情况进行特殊处理。在客户端需要额外的代码去处理可能返回`null`的情况，比如：

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
	System.out.println("Jolly good, just the thing.");
```

几乎每次使用返回`null`来代替空集合或数组的方法时，这种啰嗦的写法都是无法避免的。这很容易出错，因为编写客户端的程序员可能会忘记对返回`null`的特殊情况进行处理。这样的错误可能会被忽略多年，因为这样的方法通常返回一个或多个对象。而且，返回`null`代替空容器，会使返回容器的方法实现复杂化。

有时有人认为，返回一个`null`比空集合或数组更可取，因为它避免了分配空容器的开销。这个论点有两点是不成立的。首先，在这个级别上担心性能是不明智的，除非度量表明所讨论的容器分配的确是导致性能问题的直接原因（条款67）。第二，返回空集合和数组而不分配它们是可能的。下面是返回可能为空的集合的典型代码。通常，这就是你所需要的：

```java
//The right way to return a possibly empty collection
public List<Cheese> getCheeses() {
	return new ArrayList<>(cheesesInStock);
}
```

假设你有证据表明分配空集合会损害性能，那么你可以通过重复返回相同的不可变空集合来避免分配，因为不可变对象可以自由共享（条款17）。已经有代码来做这件事，去使用` Collections.emptyList`方法。如果你要返回一个`set`，你可以使用`Collections.emptySet`；如果你要返回一个`map`，你可以使用`Collections.emptyMap`。但是请记住，这是一个优化，很少需要它。如果你认为你需要它，测量一下前后的表现，确保它确实有帮助：

```java
// Optimization - avoids allocating empty collections
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

