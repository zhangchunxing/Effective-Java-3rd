# 在一个源文件中只定义一个顶层类

虽然Java编译器允许你在单个源文件中定义多个顶层类，但是这样做没有任何好处，而且存在很大的风险。风险源于这样一个事实：在单个源文件中定义多个顶层类使得一个类会出现多个定义。到底使用哪个定义会受到传递给编译器的源文件的顺序的影响。具体说明一下，考虑如下这个源文件，它只包含一个`Main`类，这个`Main`类引用了另外两个顶层类（`Utensil`与`Dessert`）的成员。

```java
public class Main {
    public static void main(String[] args) {
    	System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

现在，假设在一个名为`Utensil.java`的源文件中定义了`Utensil`与`Dessert`：

```java
// Two classes defined in one file. Don't ever do this!
class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```

当然了，主程序会打印出`pancake`。

现在假设你不小心创建了另一个名为`Dessert.java`的源文件，它也定义了相同的2个类：

```java
// Two classes defined in one file. Don't ever do this!
class Utensil {
	static final String NAME = "pot";
}
class Dessert {
	static final String NAME = "pie";
}
```

如果通过命令`javac Main.java Dessert.java`来编译该程序，那么编译会失败，编译器会提示说定义了多个`Utensil`与`Dessert`类。

这是因为编译器首先会编译`Main.java`，当它遇到对`Utensil`的引用时（它位于对`Dessert`的引用之前）会在`Utensil.java`中寻找该类，但却找到了`Utensil`与`Dessert`两个类。当编译器遇到了命令行中的`Dessert.java`时，它也会寻找该文件，这就导致它会遇到两个`Utensil`与`Dessert`定义。

如果通过命令`javac Main.java`或是`javac Main.java Utensil.java`编译该程序，其行为与你编写`Dessert.java`文件之前时一样，会打印出`pancake`。不过，如果通过命令`javac Dessert.java Main.java`编译该程序，那么它会打印出`potpie`。这样，程序的行为就会受到传递给编译器的源文件的顺序的影响，这显然是不可接受的。

修复这个问题很简单，只需要将顶层类（该示例中就是`Utensil`与`Dessert`）放到单独的源文件中。如果你有将多个顶层类放到单个源文件中的想法，那么请考虑使用静态成员类（条款24）作为一种变通手段。如果某些类服务于另一个类，那么将这些类置为静态成员类一般来说是个不错的做法，因为这会增强可读性，并且通过将其设为私有减少类的可访问性（条款15）。如下示例展示了使用静态成员类对上面程序所做的改造：

```java
// Static member classes instead of multiple top-level classes
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    private static class Utensil {
        static final String NAME = "pan";
    }
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

结论很明确：**永远不要将多个顶层类或接口放到单个源文件中**。遵循这一原则会确保编译期不会出现单个类拥有多个定义的情况。这又反过来保证了编译所生成的类文件以及生成的程序的行为与传递给编译器的源文件的顺序没有任何关系。