# 注意字符串拼接的性能

字符串拼接运算符（+）是将几个字符串合并成为一个的便捷方式。对于生成一个单行输出或是构建小型、固定大小对象的字符串表示来说，它很好用，不过它却不具备可伸缩性。 不断使用字符串拼接运算符来拼接n个字符串所需要的时间为n的二次方。 由于字符串是不可变的（条款17），因此这是一个很不幸的结果。在对两个字符串进行拼接时，他们的内容都会被复制一份。

比如说，考虑如下方法，它用于构造一个账单的字符串表示，做法是不断对每个条目的一行进行拼接

```java
// Inappropriate use of string concatenation - Performs poorly!
public String statement() {
	String result = "";
	for (int i = 0; i < numItems(); i++)
		result += lineForItem(i); // String concatenation
    
	return result;
}
```

如果条目数量很多，那么该方法的性能将会非常差。为了得到可接受的性能，请使用`StringBuilder`来代替`String`存储构建过程中的清单：

```java
public String statement() {
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    
	for (int i = 0; i < numItems(); i++)
		b.append(lineForItem(i));
    
	return b.toString();
}
```

从Java 6开始，人们投入了巨大的精力让字符串拼接的速度变得更快一些，不过这两个方法之间在性能上的差异依旧很大：如果`numItems`返回100，`lineForItem`返回一个80个字符的字符串，那么在我自己的机器上，第2个方法的运行速度要比第1个快6.5倍之多。由于第1个方法在时间上是条目数量的二次方，而第2个方法是线性的，因此当条目数量量增加时，二者之间在性能上的差异将会变得越来越明显。注意到第2个方法预先分配了一个足以容纳下整个结果的`StringBuilder`，这就不需要对`StringBuilder`进行自动扩容了。即便使用了默认大小的`StringBuilder`，其速度也要比第1个方法快5.5倍。

道理很简单：不要使用字符串拼接运算符来组合多个字符串，除非性能问题不重要。请使用`StringBuilder`的`append`方法来替代。此外，还可以使用字符数组，或是一次处理一个字符串而不是将其组合起来。