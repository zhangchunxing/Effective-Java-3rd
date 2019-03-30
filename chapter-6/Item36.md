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

有一些程序员更喜欢使用枚举相对于整型常量，但是当需要传递常量集合时，他们仍然坚持使用位字段。这里没有理由这样做，因为存在更好的选择。`java.util`包提供了`EnumSet`类来有效地表示单一的枚举类型的值集合。这个类实现了`Set`接口，并提供了与任何其他`Set`实现相同的所有丰富性、类型安全性和互操作性。但在内部，每个`EnumSet`都表示为一个位向量。如果底层枚举类型有64个或更少的元素（大多数元素都有），则整个`EnumSet`用一个`long`，所以它的性能可以跟一个位字段相媲美。批量操作（如`removeAll`和`retainAll`）是使用位算法实现的，就像手动处理位字段一样。但是你不会遇到手动对位操作带来的难看和潜在错误：因为`EnumSet`为你完成了繁重的工作。

下面是修改前一个示例去使用枚举和枚举集合代替位字段时的样子。它更简便、更清晰、更安全：

```java
// EnumSet - a modern replacement for bit fields
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
	// Any Set could be passed in, but EnumSet is clearly best
	public void applyStyles(Set<Style> styles) { ... }
}
```

下面是一个客户端代码，它将`EnumSet`实例传递给`applyStyles`方法。`EnumSet`类提供了一组丰富的静态工厂，可以方便地创建集合，其中一个例子如下：

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

注意，`applyStyles`方法采用`Set<Style>`，而不是`EnumSet<Style>`。虽然似乎所有客户端都可以将`EnumSet`传递给该方法，但通常最好的做法是接受接口类型而不是实现类型（条款64）。因为这种做法允许特殊的客户端传递一些其它`Set`实现的可能性。

总结一下，**如果枚举类型只是用在集合中，那就不要要用位字段表示**。`EnumSet`类结合了位字段的简洁性和性能，以及条款34中描述的枚举类型的所有优点。`EnumSet`一个真正缺点是，在`Java 9`中，它无法创建不可变的`EnumSet`，但是在即将发布的版本中，这一点可能会得到纠正。在此期间，你可以用` Collections.unmodifiableSet`来包装`EnumSet`，但简洁性和性能将受到影响。

