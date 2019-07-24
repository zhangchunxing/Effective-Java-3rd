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



