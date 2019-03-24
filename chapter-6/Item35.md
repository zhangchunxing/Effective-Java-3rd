# 使用实例字段而非序数

很多枚举都会自然关联到一个`int`值。所有枚举都有一个`ordinal`方法，它会返回每个枚举常量在其类型中的数字位置。你可能会想通过`ordinal`获取到一个关联的int值：

```java
// Abuse of ordinal to derive an associated value - DON'T DO THIS
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET,
	SEXTET, SEPTET, OCTET, NONET, DECTET;
    
	public int numberOfMusicians() { return ordinal() + 1; }
}
```

虽然这个枚举可以正常使用，但对于维护来说却是个梦魇。如果常量重新进行了排序，那么`numberOfMusicians`方法就会出问题。如果想要再添加一个与之前已经使用过的`int`值相关联的枚举常量，那就悲催了。比如说，添加一个针对双四重奏的常量应该是挺好的事情，它就像八重奏一样，包含了8个乐师，但我们却没法做到这一点。

此外，如果想要添加针对某个`int`值的常量，那就必须得为所有中间的`int`值添加常量。比如说，假设你想要添加一个表示三个四重奏的常量，它包含了12个乐师。不过，并没有标准的术语来表示包含11个乐师的组合，因此你就得为无符号`int`值（11）添加一个假的常量。这么做顶多很丑陋。但如果有很多`int`值都用不上，那就是不切实际了。

幸好，这个问题有一个简单的解决之道。**永远不要通过`ordinal`获取与枚举所关联的值；请将其存储到实例字段中**：

```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
	SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
	NONET(9), DECTET(10), TRIPLE_QUARTET(12);
    
	private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size; }
	public int numberOfMusicians() { return numberOfMusicians; }
}
```

枚举规范中关于`ordinal`是这样说的：『大多数程序员都用不到这个方法。设计它的目的是给那些基于枚举的通用目的的数据结构所用，比如说`EnumSet`与`EnumMap`』。除非你所编写的是这样的代码，否则最好不要碰`ordinal`方法。
