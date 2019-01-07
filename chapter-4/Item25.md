# 在一个源文件中只定义一个顶层类

虽然Java编译器允许您在单个源文件中定义多个顶级类，但是这样做没有任何好处，而且存在很大的风险。风险源于这样一个事实:在源文件中定义多个顶级类使得为一个类提供多个定义成为可能。使用哪个定义取决于源文件传递给编译器的顺序。说得具体点，请考虑这样一个源文件，它只包含一个`Main`类，这个`Main`类引用另外两个顶层类(`Utensil`和`Dessert`)的成员：

```java
public class Main {
    public static void main(String[] args) {
    	System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

现在假设你把`Utensil `和`Dessert `都定义在一个单独的源文件`Utensil.java `中：

```java
// Two classes defined in one file. Don't ever do this!
class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```

当然主程序会打印`pancake`。

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

如果你足够幸运通过命令`javac Main.java Dessert.java`来编译这个程序，编译会失败，而且编译器会告诉你你已经重复定义了`Utensil`和`Dessert`类。

这是因为编译器会先编译`Main.java`，并且当它看到对`Utensil`的引用（在`Dessert`引用之前）时，它会到`Utensil.java`文件里去查找这个类，然后找到了`Utensil`和`Dessert`。在这个命令行上，当编译器遇到了` Dessert.java `文件，它也会进入这个文件中，导致编译器又碰上`Utensil `和`Dessert `的定义。