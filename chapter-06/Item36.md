# 使用EnumSet代替位字段

如果一个枚举类型的元素主要用在`Set`中，那么传统的方式是使用`int`枚举模式（条款34），将2的不同指数方分配给每个常量：

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

这种表示可以让你使用『按位或』操作来将不同的常量组合到一个`Set`中，它叫做**位字段**。

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

还可以通过位字段表示使用按位运算来高效执行诸如求并集与交集等集合操作。不过，位字段有着`int`枚举常量所有的缺点，甚至更多。在打印为数字时，位字段甚至要比简单的`int`枚举常量更加难以解析。并没有一种简单的方式来遍历位字段表示的所有元素。最终，你不得不需要计算出编写API时所需的最大的位数字，然后为位字段选择一种类型（通常是`int`或是`long`）。一旦选择了某个类型，你就不能在不修改API的情况下超过其宽度（32位或是64位）。

一些使用枚举而非`int`常量的程序员在需要传递常量集合时依旧会坚持使用位字段。这么做是没有道理的，因为存在更好的方式。`java.util`包提供了`EnumSet`类来高效表示单个枚举类型的值集合。该类实现了`Set`接口，提供了任何其他的`Set`实现所拥有的功能、类型安全与互操作性。如果底层的枚举类型有64个或是更少的元素（大多数都是这种情况），那么整个`EnumSet`都会表示为单个`long`类型，这样其性能就与位字段相当了。块操作（如`removeAll`与`retainAll`）是通过位运算来实现的，就像你手工操作位字段一样。不过，这样做可以让你远离手工进行位操作的丑陋与易错性：`EnumSet`帮你完成了最困难的工作。

如下代码对之前的示例进行了修改，它使用枚举与`EnumSet`代替了位字段。这段代码看起来更短小、更整洁，也更安全：

```java
// EnumSet - a modern replacement for bit fields
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
	// Any Set could be passed in, but EnumSet is clearly best
	public void applyStyles(Set<Style> styles) { ... }
}
```

如下是客户端代码，它将一个`EnumSet`实例传递给了`applyStyles`方法。`EnumSet`类提供了丰富的静态工厂来让用户更轻松地创建集合，其中一个如下代码所示：

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

注意，`applyStyles`方法接收一个`Set<Style>`而非`EnumSet<Style>`。虽然看起来所有客户端都会将`EnumSet`传递给该方法，但更好的实践则是接收接口类型而非实现类型（条款64）。这样，客户端就可以传递其他的`Set`实现了。

总结一下，由于枚举类型会用在`Set`中，因此没有理由再使用位字段来表示它。`EnumSet`类同时拥有位字段的简洁性与性能，还具备条款34所介绍的枚举类型的诸多优势。截止到Java 9，`EnumSet`的一个缺点则是我们无法创建不可变的`EnumSet`，不过在未来的版本中这个问题很可能就会被解决掉。同时，你可以通过`Collections.unmodifiableSet`来包装`EnumSet`，不过这会牺牲掉简洁性与性能。