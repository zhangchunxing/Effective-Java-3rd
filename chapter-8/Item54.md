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

