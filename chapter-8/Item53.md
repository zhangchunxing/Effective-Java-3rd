# 谨慎使用可变参数

可变参数方法，即可变的参数数量的方法，接受0个或多个指定类型的参数。可变参数是这样工作的：在你调用方法时，根据传递的参数数量创建一个同样大小的数组，然后将参数值放到这个数组中，最后把数组传递到该方法中。

比如说，有一个可变参数的方法，接受一个整型参数的序列，并且返回它们的总和。正如你所期望的，`sum(1,2,3)`的值是6并且`sum()`是0。

```java
// Simple use of varargs
static int sum(int... args) {
	int sum = 0;
	for (int arg : args)
		sum += arg;
	return sum;
}
```

有的时候，我们需要写一个接受1个或者多个某种类型的参数，而不是0个或者多个。比如说，假如你想写一个函数计算参数列表中的最小值。如果客户端没有传递参数，那么这个函数就不好定义。你可能要检查一下数组的长度在运行时：

```java
// The WRONG way to use varargs to pass one or more arguments!
static int min(int... args) {
	if (args.length == 0)
		throw new IllegalArgumentException("Too few arguments");
    
	int min = args[0];
	for (int i = 1; i < args.length; i++)
		if (args[i] < min)
			min = args[i];
    
	return min;
}
```

这个解决方案有几个问题。最严重的情况是，如果客户端不带参数调用此方法，那么它会在运行期失败而不是编译期。另一个问题是代码变得很丑。你不得不对参数做正确性校验，并且你不能使用`for-each`循环，除非你初始化`min`为`Integer.MAX_VALUE`，但这也很不好看。

幸运的是，有一种更好的方法可以达到期望的效果。声明一个接受2个参数的方法，1个指定类型的普通参数和1个同样类型的可变参数。这个解决方案纠正了前一个解决方案的所有缺陷：

```java
// The right way to use varargs to pass one or more arguments
static int min(int firstArg, int... remainingArgs) {
	int min = firstArg;
	for (int arg : remainingArgs)
		if (arg < min)
			min = arg;
    
	return min;
}
```

从这个例子中可以看出，在需要参数数量可变的方法的情况下，可变参数是有效的。可变参数是为了`printf`设计的，它和可变参数一同添加到了平台库中，并且为了核心的反射库（条款65），可变参数被改进了。`printf`和反射都从可变参数中受益匪浅。

在性能关键的场景下，要小心使用可变参数。每次调用可变参数的方法，都会产生一个数组的分配和初始化。如果你已经从经验上确定你负担不起这个成本，但是你又需要可变参数的灵活性，那么有一种模式可以让你鱼与熊掌兼得。假设你已经确定95%的该方法的调用具有3个或更少的参数。然后声明该方法的5个重载方法，每次重载0到3个普通参数，当参数数量超过3个时使用一个可变参数的方法：

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

现在你知道，在所有参数数量超过3的调用中，只有5%的调用需要支付创建数组的成本。与大多数性能优化一样，这种技术通常不合适，但当它合适时，它是一个救星。`EnumSet`的静态工厂使用这种技术将创建枚举集合的成本降到最低。这是适当的，因为`enum`集合为位字段提供具有性能竞争力的替换是至关重要的(第36项)。

总之，当你需要定义一个可变的参数数量的方法时，可变参数会非常有用。将一些必要参数放在可变参数之前，并小心使用可变参数带来的性能影响。