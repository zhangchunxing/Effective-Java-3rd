# 注意字符串拼接的性能

字符串连接操作符（+）是将几个字符串组合成一个字符串的方便方法。对于生成单行输出或者构造一个小的、固定大小的对象的字符串表示形式，它是可以的，但是它不能伸缩。 重复使用字符串连接运算符来拼接n个字符串需要的时间为n的平方。 这是一个不幸的结果，因为字符串是不可变的（条款17）。当两个字符串拼接在一起时，它们的内容都会被复制。

例如，考虑这个方法，它通过重复拼接每个条目的行来构造出账单的字符串表示形式：

```java
// Inappropriate use of string concatenation - Performs poorly!
public String statement() {
	String result = "";
	for (int i = 0; i < numItems(); i++)
		result += lineForItem(i); // String concatenation
    
	return result;
}
```

如果条目的数量很大，则该方法执行得非常糟糕。 为了获得可接受的性能，使用`StringBuilder`代替`String`来存储正在构建的账单。

```java
public String statement() {
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    
	for (int i = 0; i < numItems(); i++)
		b.append(lineForItem(i));
    
	return b.toString();
}
```

自Java 6以来，我们做了大量工作，就是为了提高字符串拼接的速度。 但以上两种方法在性能上的差异仍然很大 。如果`numItems`返回100，`lineForItem`返回一个80个字符的字符串，那么在我的机器上第二个方法运行的速度是第一个方法的6.5倍。由于第一种方法时间复杂度是条目数量的平方，而第二种方法是线性的，所以随着条目数量的增加，性能差异会越来越大。注意，第二个方法预先分配了一个足够大的`StringBuilder`来容纳整个结果，从而消除了自动增长的需要。

这个道理很简单：除非性能无关紧要，否则不要使用字符串连接操作符组合多个字符串。而是使用`StringBuilder`的`append`方法。或者，使用字符数组，或者一次处理一个字符串，而不是组合它们。