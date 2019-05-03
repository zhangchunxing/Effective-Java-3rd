# 优先选择lambdas而非匿名类续

以前，只有一个抽象方法的接口（或抽象类）被当做**函数类型**。他们的实例，被叫做**函数式对象**，代表了函数或行为。自从JDK 1.1在1997年发布以来，创建函数对象的主要方法就是**匿名类**（条款24）。下面是一个代码片段，使用一个匿名类来创建排序的比较函数（影响排序顺序），按长度顺序对字符串列表进行排序：

```java
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
});
```

对于需要函数对象的经典面的向对象设计模式，特别是策略模式[Gamma95]，匿名类足够用了。`Comparator`接口代表了一个用于排序的**抽象策略**；上面的匿名类就是用来给字符串排序的**具体策略**。然而，匿名类的冗长使得用Java进行函数式编程变得毫无吸引力。

在Java8中，Java形成了这样一个概念——只有1个抽象方法的接口是特殊的，应该得到特殊处理。这些接口现在被称作**函数式接口**，Java允许你使用`lambda`表达式（或简称`lambdas`）创建这些接口的实例。`Lambdas`的功能类似于匿名类，但要简洁得多。上面的使用匿名类的代码片段怎么用lambda替换。下面展示了上面的代码用lambda替换匿名类的样子。样板文件不见了，行为却更加明显了：

```java
// Lambda expression as function object (replaces anonymous class)
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

