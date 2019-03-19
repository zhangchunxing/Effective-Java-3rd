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

