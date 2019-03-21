# 使用实例字段而非序数

大多数枚举很自然地与一个`int`值关联。所有枚举都有一个`ordinal`方法，它返回每个枚举常量在其类型中的数值位置。你可能想从序数中获得相关的`int`值：

```java
// Abuse of ordinal to derive an associated value - DON'T DO THIS
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET,
	SEXTET, SEPTET, OCTET, NONET, DECTET;
    
	public int numberOfMusicians() { return ordinal() + 1; }
}
```

尽管这个枚举可以正常使用，但是维护它将是一个噩梦。如果常量被重新排序，`numberOfMusicians `方法就会崩溃。如果你想再添加一个常量，并关联一个你已经使用过的`int`值，这是不可能的。例如，为双四重奏添加一个常量可能很好，它像八重奏一样，由八个音乐家组成，但是没有办法做到这一点。

此外，如果不为所有中间的`int`值添加常量，就不能为`int`值添加常量。例如，假设你想添加一个表示三元四重奏（由12位音乐家组成）的常量。对于由11个音乐家组成的合奏没有标准术语，因此你必须为未使用的int值（11）添加一个虚拟常量。充其量，这是丑陋的。如果许多int值未被使用，这是不切实际的。

幸运的是，这些问题有一个简单的解决方案。**永远不要从枚举的序号派生与枚举相关的值；而是将它存储在一个实例字段中**：

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

`Enum`规范对`ordinal`有这样的描述：“大多数程序员是不会使用这种方法。它被设计用于基于枚举的通用数据结构，如`EnumSet`和`EnumMap`。“除非使用这个字符编写代码，否则最好完全避免使用序号方法。

