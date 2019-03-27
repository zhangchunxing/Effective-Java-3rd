# 使用EnumSet代替位字段

如果枚举类型的元素主要在集合中使用，则传统上使用整型枚举模式（条款34），为每个常量分配不同的2次方：

```java
// Bit field enumeration constants - OBSOLETE（过时）!
public class Text {
	public static final int STYLE_BOLD = 1 << 0; // 1
	public static final int STYLE_ITALIC = 1 << 1; // 2
	public static final int STYLE_UNDERLINE = 1 << 2; // 4
	public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
	// Parameter is bitwise OR of zero or more STYLE_ constants
	public void applyStyles(int styles) { ... }
}
```

这种表示方式允许你使用按位或操作将几个常量组合成一个集合，称为**位字段**：

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

位字段表示还允许你通过使用按位运算来高效地执行集合操作，比如并集和交集。但是位子段拥有整型枚举常量的所有缺点，甚至更多。当一个位字段被打印为一个数字时，它比一个简单的整型枚举常量更难解释。没有简单的方法可以去遍历所有的由位字段表示的元素。最后，你必须在编写API时，预测所需的最大比特数，并相应地为位字段选择一种类型（通常是`int`或`long`）。一旦选择了一种类型，如果不更改API，就不能超过它的宽度（32位或64位）。

