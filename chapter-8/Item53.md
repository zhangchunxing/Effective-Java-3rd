# 谨慎使用可变参数

可变参数方法（正式的叫法是参数数量可变的方法 [JLS, 8.4.1]）可以接收零个或多个指定类型的参数。可变参数机制的工作方式是首先创建一个数组，其大小等于传递进来的参数的数量，然后将参数值放到该数组中，最后将数组传递给方法。

比如说，如下是个可变参数方法，它接收一系列的`int`参数，然后返回这些参数的和。如你所想，`sum(1, 2, 3)` 的值是6，`sum()` 的值是0：

```java
// Simple use of varargs
static int sum(int... args) {
	int sum = 0;
	for (int arg : args)
		sum += arg;
	return sum;
}
```

有时，一个方法需要接收一个或多个某类型的参数，而非零个或多个。比如说，假设你想要编写一个用于计算最小参数的函数。如果客户端没有传递参数，那么该函数就失去了意义。你应该在运行期检查数组长度：

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

该解决方案存在几个问题。最严重的一个问题就是，如果客户端调用方法时没有传递参数，那么它就会在运行期而非编译期失败。另一个问题则是该实现方式太丑陋了。你得对`args`进行显式的有效性检查，无法使用`for-each`循环，除非将`min`初始化为`Integer.MAX_VALUE`，不过这么做也不咋样。

幸好，我们可以通过一种更好的方式来实现我们的目标。让方法接收两个参数，一个是所指定类型的正常参数；另一个则是该类型的可变参数。该解决方案消除了之前做法的所有瑕疵：

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

如你所见，当你需要一个接收可变数量参数的方法时，使用可变参数就会很有效。可变参数是针对`printf`设计的，它是与可变参数一同被添加到平台中的；同时，可变参数也是针对核心反射基础设施（条款65）而设计的，后者则进行了重构。`printf`与反射都从可变参数获益无穷。

在性能关键的场景中使用可变参数需要小心一些。每次调用可变参数方法都会导致数组的分配与初始化。如果觉得无法承受这种代价，但同时又需要可变参数所带来的灵活性，那么有一种模式可以让你鱼和熊掌兼得。假设你发现对方法95%的调用都只需要3个或是更少的参数。那么，请声明该方法的5个重载版本，这些重载的方法所接收的参数数量从零个一直到3个，同时再提供一个可变参数方法，用于参数数量超过3个时所用：

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

现在，你知道只有在5%的调用中（参数数量超过3个）才需要付出创建数组的代价。就像大多数性能优化一样，该项技术通常并不恰当，不过当需要时还是可以解决问题的。

`EnumSet`的静态工厂就使用了该项技术来将枚举集合的创建成本降到了最低。这么做是恰当的，因为枚举集合要提供与位字段相当的性能指标，这很重要（条款36）。

总结一下，当需要定义接收数量可变的参数的方法时，可变参数就是非常有价值的了。在可变参数之前加上任意所需的参数，同时要意识到使用可变参数所带来的性能后果。