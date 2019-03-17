# 优先考虑泛型方法

就像类可以是泛型的一样，方法同样也可以是泛型的。用于操纵参数化类型的静态辅助方法通常都是泛型的。`Collections`中所有与『算法』相关的方法（比如说`binarySearch`与`sort`）都是泛型的。

编写泛型方法类似于编写泛型类型。考虑如下这个有缺陷的方法，它返回了两个集合的并集：

```java
// Uses raw types - unacceptable! (Item 26)
public static Set union(Set s1, Set s2) {
	Set result = new HashSet(s1);
	result.addAll(s2);
	return result;
}
```

这个方法可以编译，但是有2个警告：

```java
Union.java:5: warning: [unchecked] unchecked call to
HashSet(Collection<? extends E>) as a member of raw type HashSet
Set result = new HashSet(s1);
             ^
Union.java:6: warning: [unchecked] unchecked call to
addAll(Collection<? extends E>) as a member of raw type Set
result.addAll(s2);
             ^
```

为了消除这些警告并确保方法类型安全，要对其声明进行修改，我们会声明一个类型参数，表示3个集合（两个参数和一个返回值）中的元素类型，并在整个方法中使用这个类型参数。**类型参数列表（声明了类型参数）位于方法修饰符及其返回类型之间**。在该示例中，类型参数列表是`<E>`，返回类型是`Set<E>`。对于泛型方法与泛型类型来说，其类型参数的命名约定是一致的（条款29与68）：

```java
// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```

至少对于简单的泛型方法来说，这些就够了。该方法编译时不会产生任何警告，并且会提供类型安全性与易用性。下面是个简单的程序，用来执行该方法。这个程序中没有出现类型转换，并且编译时也没有任何错误与警告：

```java
// Simple program to exercise generic method
public static void main(String[] args) {
	Set<String> guys = Set.of("Tom", "Dick", "Harry");
	Set<String> stooges = Set.of("Larry", "Moe", "Curly");
	Set<String> aflCio = union(guys, stooges);
	System.out.println(aflCio);
}
```

当运行程序时，它会打印出[Moe, Tom, Harry, Larry, Curly, Dick]（输出中的元素顺序与实现相关）。这个`union`方法的一个限制在于，所有3个集合（两个输入参数和一个返回值）的类型都必须是完全一样的。可以通过**有界限的通配符类型（**条款31）让方法变得更加灵活一些。

有时，你想要创建一个不可变的对象，但却又能应用到很多不同的类型上。由于泛型是通过类型擦除实现的（条款28），因此可以使用单个对象来应对所有的类型参数化需求，不过你需要编写一个静态工厂方法为每个所需的类型参数化重复使用该对象。这种模式叫做泛型单例工厂，它用于函数对象上（条款42），如`Collections.reverseOrder`；有时也会用在集合上，如`Collections.emptySet`。

假设你想要编写一个恒等函数分配器。Java库提供了`Function.identity `， 因此没理由再去编写自己的了（条款59），不过自己做一下还是能学到一些东西的。每次请求都创建一个新的恒等函数对象是很浪费的，因为它是无状态的。如果Java泛型是具化的，那么每个类型就都需要一个恒等函数了，不过由于类型擦除的原因，因此一个泛型单例就足够了。它是这样的：

```java
// Generic singleton factory pattern
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
	return (UnaryOperator<T>) IDENTITY_FN;
}
```

由`IDENTITY_FN`到（`UnaryFunction<T>`）的转换会生成一个未检查的类型转换警告，因为对于每个`T`来说，`UnaryOperator<Object>`并非`UnaryOperator<T>`。不过，恒等函数有其特殊性：它无修改地返回了其参数，这样我们就知道无论`T`的值是什么，将其作为`UnaryFunction<T>`是类型安全的。

因此，我们可以自信地压制该类型转换所生成的未检查转换警告。接下来，代码就可以顺利通过编译，没有任何错误和警告。

如下这个示例程序将我们的泛型单例用作了`UnaryOperator<String>`与`UnaryOperator<Number>`。与往常一样，它没有进行类型转换，编译时也不会出现错误和警告：

```java
// Sample program to exercise generic singleton
public static void main(String[] args) {
	String[] strings = { "jute", "hemp", "nylon" };
	UnaryOperator<String> sameString = identityFunction();
	for (String s : strings)
		System.out.println(sameString.apply(s));
    
	Number[] numbers = { 1, 2.0, 3L };
	UnaryOperator<Number> sameNumber = identityFunction();
	for (Number n : numbers)
		System.out.println(sameNumber.apply(n));
}
```

虽然很少遇到，不过一个类型参数受到包含该类型参数本身的表达式的制约也是允许的。这叫做递归的类型约束。递归的类型约束经常用在`Comparable`接口上，它定义了一个类型的自然排序（条款14）。该接口代码如下所示：

```java
public interface Comparable<T> {
	int compareTo(T o);
}
```

类型参数`T`定义了一个类型，实现了`Comparable<T>`的类型的元素可以与其进行比较。实际上，几乎所有的类型只能与自身类型的元素进行比较。比如说，`String`实现了`Comparable<String>`，`Integer`实现了`Comparable<Integer>`，诸如此类。

很多方法都会接收一个实现了`Comparable`的元素的集合，对其进行排序、搜索，计算其最小值或是最大值等等。要想完成这些任务，集合中的每个元素都要能与其他元素进行比较；换言之，列表中的元素要能相互进行比较。如下代码表达了这样的约束：

```java
// Using a recursive type bound to express mutual comparability
public static <E extends Comparable<E>> E max(Collection<E> c);
```

类型边界`<E extends Comparable<E>>`可以理解为『能与自身进行比较的任何类型E』，这多少与相互比较的概念类似。

如下方法遵循了之前的声明。它根据集合中元素的自然顺序计算出最大值，该方法可以顺利通过编译，并且没有任何错误或是警告：

```java
// Returns max value in a collection - uses recursive type bound
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

注意到如果列表为空，该方法会抛出`IllegalArgumentException`。更好的方式则是返回一个`Optional<E>`（条款55）。

递归的类型边界可以变得更加复杂一些，不过幸好，很少会出现这种情况。如果理解了这种用法、其通配符变量（条款31）以及模拟的自我类型用法（条款2），那么你就能在实际开发中处理好大多数递归的类型边界问题。

总结一下，类似于泛型类型，泛型方法要比那些要求客户端对输入参数以及返回值进行显式类型转换的方法更加安全且易于使用。就像类型一样，你应该确保方法使用时无需进行类型转换，通常这需要使其成为泛型的才可以。此外，你应该对需要进行类型转换的既有方法进行泛型化。这会让新用户用起来更爽，并且不会破坏既有的客户端（条款26）。