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

