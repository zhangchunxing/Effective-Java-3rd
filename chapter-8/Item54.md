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

我们没什么理由对无奶酪可售的情况进行特殊处理。这么做需要在客户端使用额外的代码来处理可能的`null`返回值，比如说：

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
	System.out.println("Jolly good, just the thing.");
```

对于返回`null`而非空集合或是数组的几乎每个方法来说都需要这种处理。这很容易出错，因为编写客户端的程序员可能忘记编写特殊处理代码来处理`null`返回值这种情况。这种错误可能经年累月也不会为人所发现，因为这样的方法通常会返回一个或是多个对象。此外，返回`null`而非空容器的做法会使得返回容器的方法实现变得更加复杂。

有时，有些人会认为`null`返回值要比空集合或是数组更加适合，因为它避免了分配空容器的开销。这个观点在两方面是不成立的。首先，在这个层面上担心性能是没必要的，除非有度量结果表明容器分配真的是性能问题的罪魁祸首（条款67）。其次，我们可以返回空集合与数组而不对其进行分配。如下代码返回了一个有可能为空的集合。通常，这就是你需要的全部：

```java
//The right way to return a possibly empty collection
public List<Cheese> getCheeses() {
	return new ArrayList<>(cheesesInStock);
}
```

即便有证据表明分配空集合会对性能产生影响，你也可以通过重复返回同一个不变的空集合来避免分配问题，因为不变对象是可以自由共享的（条款17）。如下代码通过`Collections.emptyList`方法就做到了这一点。如果返回的是`Set`，那就可以使用`Collections.emptySet`；如果返回的是`Map`，则可以使用`Collections.emptyMap`。不过请记住，这只是一种优化而已，很少会被使用。如果需要，那么请先度量性能，从而确保这么做真的有用：

```java
// Optimization - avoids allocating empty collections
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

数组的情况与集合一样。永远不要用`null`来代替长度为0的数组作为返回值。通常情况下，你只需返回正确长度的数组即可，这个长度可能为0。注意到我们将长度为0的数组传递给了`toArray`方法来表示所需的返回类型，即`Cheese[]`：

```java
//The right way to return a possibly empty array
public Cheese[] getCheeses() {
	return cheesesInStock.toArray(new Cheese[0]);
}
```

如果觉得分配长度为0的数组会对性能产生影响，那就可以重复返回相同的长度为0的数组，因为所有长度为0的数组都是不可变的：

```java
// Optimization - avoids allocating empty arrays
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

在这个优化版本中，我们向每个`toArray`调用都传递了相同的空数组，当`cheesesInStock`为空时，该数组会从`getCheeses`中返回。不要预先分配传递给`toArray`的数组以期达到改进性能之目的。研究表明，这么做是徒劳的 。

```java
// Don’t do this - preallocating the array harms performance!
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

总结一下，永远不要用`null`来代替空数组或是集合作为返回值。这会使得你的API变得难以使用且易于出错，同时也不会带来任何性能上的好处。