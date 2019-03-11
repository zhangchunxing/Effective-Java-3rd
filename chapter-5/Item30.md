# 优先考虑泛型方法

类可以是泛型的，方法也可以是泛型的。操作参数化类型的静态工具方法通常是泛型的。`Collections`中的所有“算法”方法（如`binarySearch`和`sort`）都是泛型的。

编写泛型方法类似于编写泛型类型。考虑这个有缺陷的方法，它返回两个集合的并集：

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

为了修复这些警告并使得方法类型安全，现在将它的声明修改成声明一个类型参数，来表示这3个集合的元素类型（2个参数和1个返回值），并在整个方法中使用该类型参数。**类型参数列表，是用来声明类型参数的，位于方法的修饰符和它的返回值之间**。在这个例子中，类型参数列表为`<E>`，返回类型设置为`Set<E>`。类型参数的命名约定与泛型方法和泛型类型的命名约定相同（条款29，68）：

```java
// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```

至少对于简单的泛型方法，这就是全部。此方法编译时不生成任何警告，并提供了类型安全性和易用性。这里有一个简单的程序来练习这个方法。这个程序没有类型转换，并且编译时没有任何错误或者警告出现：

```java
// Simple program to exercise generic method
public static void main(String[] args) {
	Set<String> guys = Set.of("Tom", "Dick", "Harry");
	Set<String> stooges = Set.of("Larry", "Moe", "Curly");
	Set<String> aflCio = union(guys, stooges);
	System.out.println(aflCio);
}
```

当你运行这个程序时，它会打印出[Moe, Tom, Harry, Larry, Curly, Dick]。(输出中元素的顺序取决于实现)。`union`方法的一个限制是，3个集合（2个输入参数和1个返回值）的类型必须完全相同。可以通过使用**有界通配符类型（条款31）**使方法更加灵活。

有时候，你需要创建一个不可变的对象，但是要适用于许多不同的类型。因为泛型是通过擦护来实现的（条款28），所以你可以用一个单独的`Object`来代表所有需要的类型参数，但是你需要写一个静态工厂方法来重复地为每一个需要的类型参数分配一个`Object`。这种模式，叫做**通用的单例工厂**，被使用在函数对象（条款42）上，如`Collections.reverseOrder`，并且有时候对于集合来说，如`Collections.emptySet`。

假设你想编写一个恒等函数分发器。Java库已经提供了`Function.identity`，所以你没有理由自己写一个，虽然它是有启发性的。在请求一个新的恒等函数对象时创建它是浪费时间的，因为它是无状态的。如果Java的泛型被具体化，那么每个类型都需要一个恒等函数，但是由于它们被类型擦除了，所以一个泛型单例就足够了。它是这样的：

```java
// Generic singleton factory pattern
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
	return (UnaryOperator<T>) IDENTITY_FN;
}
```

将`IDENTITY_FN`强制转换为`(UnaryFunction<T>)`会生成一个未检查的强制转换警告，因为`UnaryOperator<Object>`不是针对每个`T`的`UnaryOperator<T>`。但是，恒等函数是特别的：它会返回它的参数，没有任何修改。所以，我们知道`IDENTITY_FN`转`(UnaryFunction<T>)`是类型安全的，无论`T`的值是什么。

因此，我们可以自信地抑制这个类型转换带来的未检查的类型转换异常。一旦我们这么做了，代码编译后。不会有错误和警告。

下面是一个示例程序，它使用我们的泛型单例作为`UnaryOperator<String>`和`UnaryOperator<Number>`。正常情况下，它不会包含类型转换并且编译后没有错误和警告：

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

类型参数由包含该类型参数本身的相同表达式约束，这是允许的，尽管这种情况相对较少。这就是所谓的递归类型边界。递归类型边界的一个常见用法是与`Comparable`接口连接，它定义了类型的自然顺序（条款14）。这个接口如下所示：

```java
public interface Comparable<T> {
	int compareTo(T o);
}
```

类型参数`T`定义了实现`Comparable<T>`的类型的元素可以与之进行比较的类型。实际上，几乎所有类型都只能与自己类型的元素进行比较。例如，`String`实现`Comparable<String>`， `Integer`实现`Comparable<Integer>`等等。

许多方法接受一个实现了`Comparable`接口的元素的集合，用来对其进行排序，查找，计算最大值或最小值，等等。为了实现这些，集合里的每个元素必须可以和其它元素比较，换句话说，列表中的元素必须可以相互比较。下面是如何表达这种约束：

```java
// Using a recursive type bound to express mutual comparability
public static <E extends Comparable<E>> E max(Collection<E> c);
```

这里的类型限定符`<E extends Comparable<E>>`可以读成“任意一个类型`E`都可以与自身比较”，这或多或少与相互比较的概念相对应。

下面是与前面的声明一起使用的方法。它根据元素的自然顺序计算集合中的最大值，并且编译时没有错误或警告：

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

