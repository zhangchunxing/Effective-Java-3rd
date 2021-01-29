## 在公有类中，请使用访问器方法而非公有字段

有时，你可能会编写一些没什么功能的类，它没有别的目的，只是用来将一些实例字段放在一起而已：

```java
// Degenerate classes like this should not be public!
class Point {
    public double x;
    public double y;
}
```

由于这种类的数据字段可以直接访问，因此这些类并未提供封装的种种好处（条款15）。你无法在不改变API的情况下改变其表示，无法增强其不变性，在访问一个字段时，你无法执行附加的动作。立场坚定的面向对象程序员们会觉得这种类非常令人生厌，他们总是应该被拥有私有字段和公有访问器方法（getters）的类所替代，对于可变类来说，则需要使用变化的方法（setters）：

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

当然了，对于公有类来说，立场坚定者的想法是正确的：如果一个类在其包外部可访问，那么就应该提供访问器方法来保持改变类内部表示的灵活性。如果公有类公开了其数据字段，那么改变其表示的所有希望都会丧失掉，因为客户端代码到处都是。

不过，如果类是在包内访问的，或是私有的嵌套类，那么公开其数据字段就没什么问题——假设他们做了充分的工作来描述类所提供的抽象。从视觉上来看，这种方式要比访问器方法方式更加清晰，无论是在类的定义还是在使用其的客户端代码中均如此。当客户端代码被绑定到了类的内部表示上时，代码就会被限制在类所在的包级别上了。如果要改变其表示，那就只需修改类本身即可，无需触碰到包外的任何代码。对于私有嵌套类来说，修改的范围仅会延伸到其外部类而已。

Java平台库中的几个类违反了公共类不应该直接暴露字段的建议。这其中包括了`java.awt`包下的`Point`和`Dimension`类。他们都是反面典型。正如在第67条中所述，暴露`Dimension`类内部，这一决定导致了严重的性能问题，时至今日，这一问题依旧萦绕在我们身边。

虽然对于公有类来说，直接露出其字段始终不是一个好做法，不过如果字段是不可变的，那么其带来的坏处就没那么大了。你无法在不改变其API的情况下，改变类的表现形式，在读取字段时无法执行附加的动作，不过你可以增强其不变性。例如，如下类会保证每个实例都表示一个有效的时间：

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

总结一下，公有类永远都不应该公开其可变字段。对于公开不变字段的公有类来说，其危害倒是没那么大，不过这一点也是存疑的。不过，有时我们希望包内或是私有嵌套类公开其字段，无论字段可变与否。