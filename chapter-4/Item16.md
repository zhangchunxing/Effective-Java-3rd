## 在公有类中，请使用访问器方法而非公有字段

有时，你可能会编写一些没什么功能的类，它没有别的目的，只是用来将一些实例字段放在一起而已：

```java
// Degenerate classes like this should not be public!
class Point {
    public double x;
    public double y;
}
```

因为这些类的数据字段是直接访问的，所以这些类没有提供封装的好处(第15条)。所以，你不改变API就不能改变表现层，你不能保证一切不变，而且当一个字段被访问时，你不能做附加动作。坚持面向对象开发的程序员感觉这样的类是厌恶的，并且应给被提供了私有字段和公共访问方法（setter）的类替代，并且对于可变类来说，就是mutators (setters)：

```java
// Encapsulation of data by accessor methods and mutators
class Point {
    private double x;
    private double y;
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    public double getX() { return x; }
    public double getY() { return y; }
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

当然，支持者们认为如果变成公共类，这样做是正确的：如果类可以在它的包外被访问，那么提供可访问的方法来保持改变类内部形式的灵活性。如果一个公共类暴露其数据字段，那么改变其表示形式的所有希望都将落空，因为客户机代码可以广泛分发。

然而，如果一个类是包私有的或者只一个私有的内部类，那么暴露他的数据字段本来就没有什么错——假设它们能够很好地描述类提供的抽象。无论是在类定义中还是在使用它的客户端代码中，这种方式都比提供访问方法的方式要看起来更简洁点。虽然客户端代码与类的内部表现绑定在一起，但这段代码仅限于包含该类的包下。如果想要对表示形式进行更改，您可以在不接触包外部任何代码的情况下进行更改。对于私有内部类，更改的范围进一步限制在封闭类中。

Java平台库中的几个类违反了公共类不应该直接暴露字段的建议。这其中包括了`java.awt`包下的`Point`和`Dimension`类。这些类不应被效仿，而应被视为警告。正如在第67条中所述，暴露`Dimension`类内部，这一决定导致了严重的性能问题，至今仍然存在。

虽然公共类直接露出字段从来都不是一个好主意，但是如果字段是不可变的，那么就不会造成什么危害。你无法在不改变其API的情况下，改变类的表现形式，而且当读取一个字段时，你无法做一些辅助性的动作，但是你可以强制使用不变量。例如，这个类保证每个实例代表一个有效的时间：

```java
// Public class with exposed immutable fields - questionable
public final class Time {
    
	private static final int HOURS_PER_DAY = 24;
	private static final int MINUTES_PER_HOUR = 60;
	public final int hour;
	public final int minute;
    
	public Time(int hour, int minute) {
		if (hour < 0 || hour >= HOURS_PER_DAY)
			throw new IllegalArgumentException("Hour: " + hour);
		if (minute < 0 || minute >= MINUTES_PER_HOUR)
			throw new IllegalArgumentException("Min: " + minute);
		this.hour = hour;
		this.minute = minute;
	}
	... // Remainder omitted
        
}
```

总之，公共类不应该暴露出它的可变字段。公共类暴露不可变字段，这样做没什么危害，但仍然是有问题的。然而，对于包私有或私有嵌套类来说，有时候需要露出字段，无论是可变的还是不可变的。