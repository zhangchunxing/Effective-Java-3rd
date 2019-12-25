# 注意字符串拼接的性能

字符串连接操作符（+）是将几个字符串组合成一个字符串的方便方法。对于生成单行输出或者构造一个小的、固定大小的对象的字符串表示形式，它是可以的，但是它不能伸缩。 重复使用字符串连接运算符来连接n个字符串需要的时间为n的平方。 这是一个不幸的结果，因为字符串是不可变的（条款17）。当两个字符串连接在一起时，它们的内容都会被复制。

例如，考虑这个方法，它通过重复连接每个条目的行来构造出账单的字符串表示形式：

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

