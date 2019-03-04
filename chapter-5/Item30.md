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

