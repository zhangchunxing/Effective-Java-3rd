## 在公有类中，请使用访问器方法而非公有字段

有时，你可能会想要编写一个没有任何目的简单类，只拥有实例字段：

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