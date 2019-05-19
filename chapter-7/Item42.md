# 优先选择lambdas而非匿名类

从历史上来看，只拥有单个抽象方法的接口（或是不常见到的抽象类）被用作函数类型。他们的实例，被叫做**函数对象**，表示函数或是动作。由于JDK 1.1是在1997年发布的，因此创建函数对象的主要方式就**是匿名类**（条款24）。如下代码片段用来根据长度对字符串列表进行排序，它使用匿名类（指定了排序顺序）来创建sort的比较函数：

```java
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
});
```

匿名类完全可以胜任需要函数对象的经典面向对象设计模式，特别是策略模式[Gamma95]。`Comparator`接口表示排序的**抽象策略**；上面的匿名类是用于排序字符串的**具体策略**。然而，匿名类的冗长使得用Java进行函数式编程变得毫无吸引力。

在Java 8中，语言正式对此进行了规范化，带有单个抽象方法的接口是特殊的，需要特殊对待。这种接口被称作**函数式接口**，Java允许你使用`lambda`表达式（或简称`lambdas`）创建这种接口的实例。从函数角度来看，Lambdas类似于匿名类，不过更加简洁。如下代码片段将上面的匿名类替换为了lambda。这里不再有样板代码，但行为却更加明确：

```java
// Lambda expression as function object (replaces anonymous class)
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

注意，lambda （`Comparator<String>`）的类型、它的参数（s1和s2都是`String`）类型和其返回值（int）的类型都没有出现在代码中。编编译器可以通过上下文，使用一种叫做**类型推断**的过程来推断出这些类型。在某些情况下，编译器无法确定这些类型，这时你必须指定类型。类型推断的规则很复杂：JLS用了整整一章的篇幅来介绍[JLS, 18]。很少有程序员能够理解这些规则的细节，不过这也没什么问题。如果类型的存在无法让程序变得更加整洁，那就请省略掉所有lambda参数的类型。如果编译器报错了，告诉你无法推断出lambda参数的类型，那就请指定。有时，你需要对返回值或是整个lambda表达式进行类型转换，不过这种情况很少出现。

关于类型推断这里还要强调一点。条款26指出，不要使用原生类型；条款29指出，要优先使用泛型类型；条款30指出，要优先使用泛型方法。在使用lambdas时，这些建议变得更加重要，因为编译器会从泛型当中获取到大多数类型信息以执行类型推断。如果没有提供该信息，那么编译器就无法进行类型推断，你得在lambdas中手工指定类型，这会导致代码变得异常啰嗦。就拿上面的代码片段来说吧，如果将变量`words`的类型声明为原生类型`List`而非参数化类型`List<String>`，程序将无法编译通过。

此外，如果使用比较器构建方法（条款14与43）来代替lambdas，那么上述代码片段中的比较器还可以变得更加简洁一些：

```java
Collections.sort(words, comparingInt(String::length));
```

事实上，上述代码片段还可以变得更简短一些，方式是使用Java 8向`List`接口中所添加的`sort`方法：

```java
words.sort(comparingInt(String::length));
```

语言中增加的lambdas使得我们可以在需要的地方使用函数对象，而之前则是行不通的。比如说，考虑条款34中的Operation枚举类型。由于每个枚举的`apply`方法都拥有不同的行为，因此我们使用了常量特定的类体并在每个枚举常量中重写了`apply`方法。为了帮助大家回忆起来，这里贴出之前的代码：

```java
// Enum type with constant-specific class bodies & data (Item 34)
public enum Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; }
	},
    
	MINUS("-") {
		public double apply(double x, double y) { return x - y; }
	},
	
    TIMES("*") {
		public double apply(double x, double y) { return x * y; }
	},
    
	DIVIDE("/") {	
		public double apply(double x, double y) { return x / y; }
	};
    
	private final String symbol;
    
	Operation(String symbol) { this.symbol = symbol; }
    
	@Override
	public String toString() { return symbol; }
    
	public abstract double apply(double x, double y);
}
```

条款34指出，我们应该优先使用枚举实例字段而非常量特定的类体。Lambdas通过前者而非后者使得实现常量特定的行为变得很轻松。仅仅需要将实现每个枚举常量行为的lambda传递给其构造方法即可。该构造方法将lambda存储为一个实例字段，`apply`方法会将调用转发给lambda。代码要比之前的版本更加简单和清晰：

```java
// Enum with function object fields & constant-specific behavior
public enum Operation {
	PLUS ("+", (x, y) -> x + y),
	MINUS ("-", (x, y) -> x - y),
	TIMES ("*", (x, y) -> x * y),
	DIVIDE("/", (x, y) -> x / y);
    
	private final String symbol;
	private final DoubleBinaryOperator op;
    
	Operation(String symbol, DoubleBinaryOperator op) {
		this.symbol = symbol;
		this.op = op;
	}
    
	@Override
    public String toString() {
        return symbol;
    }
    
	public double apply(double x, double y) {
		return op.applyAsDouble(x, y);
	}
}
```

注意，这里针对表示枚举常量行为的lambdas使用了`DoubleBinaryOperator`接口。这是`java.util.function`中预先定义的诸多函数式接口之一（条款44）。它表示接收两个`double`参数并返回一个`double`结果的函数。

看看这个基于lambda的`Operation`枚举，你可能觉得常量特定的方法体不会再有用武之地了，不过事实并非如此。与方法和类不同，**lambdas缺少名字与文档；如果计算无法做到自我解释，或是代码超过了一定的行数，那就不要将其放到lambda中**。对于lambda来说，**一行代码是最理想的，至多也不要超过三行**。如果违背了这个原则，那么它就会对程序的可读性带来严重的影响。如果lambda过长或是难以阅读，**请对其进行简化或是重构程序来消除它**。此外，传递给枚举构造方法的参数是在一个静态上下文中进行计算的。这样，枚举构造方法中的lambdas就无法访问到枚举中的实例成员。如果枚举类型所拥有的常量特定行为难以理解、无法在几行代码中实现，或是需要访问实例字段和方法，那么常量特定的类体依旧是我们的不二之选。

与之类似，你可能还会觉得匿名类在lambdas时代也没什么用了。大多数情况下是这样的，不过还存在一些场景，你可以通过匿名类来实现，但lambdas却做不到。Lambdas被限定到了函数式接口上。如果想要创建一个抽象类的实例，那就可以通过匿名类来实现，但却无法通过lambda实现。类似地，你可以通过匿名类创建拥有多个抽象方法的接口实例。**最后，lambda不能获得对自身的引用**。在lambda中，this关键字引用的是外层实例，通常这是你所需要的。在匿名类中，this关键字引用的是匿名类实例。**如果需要从体内访问函数对象，那就只能使用匿名类**。

Lambdas与匿名类有一个共性，即不能跨实现对其进行序列化和反序列化。因此，你不应该序列化lambda（以及匿名类实例）。如果想要序列化一个函数对象，比如说`Comparator`，那么请使用私有、静态的嵌套类实例（条款24）。

总结一下，从Java 8开始，lambdas成为了表示小函数对象的最佳方式。**除非创建的是并非函数式接口的类型实例，否则请不要再使用匿名类表示函数对象了**。此外，请记住，lambdas使得函数对象的表示变得如此轻松，它为函数式编程技术开启了大门，而这一切在之前的Java中是行不通的。