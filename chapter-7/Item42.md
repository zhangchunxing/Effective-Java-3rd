# 优先选择lambdas而非匿名类

以前，只有一个抽象方法的接口（或抽象类）被当做**函数类型**。他们的实例，被叫做**函数式对象**，代表了函数或行为。自从JDK 1.1在1997年发布以来，创建函数对象的主要方法就是**匿名类**（条款24）。下面是一个代码片段，使用一个匿名类来创建排序的比较函数（影响排序顺序），按长度顺序对字符串列表进行排序：

```java
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
});
```

对于需要函数对象的经典面的向对象设计模式，特别是策略模式[Gamma95]，匿名类足够用了。`Comparator`接口代表了一个用于排序的**抽象策略**；上面的匿名类就是用来给字符串排序的**具体策略**。然而，匿名类的冗长使得用Java进行函数式编程变得毫无吸引力。

在Java8中，Java形成了这样一个概念——只有1个抽象方法的接口是特殊的，应该得到特殊处理。这些接口现在被称作**函数式接口**，Java允许你使用`lambda`表达式（或简称`lambdas`）创建这些接口的实例。`Lambdas`的功能类似于匿名类，但要简洁得多。上面的使用匿名类的代码片段怎么用lambda替换。下面展示了上面的代码用lambda替换匿名类的样子。样板文件不见了，行为却更加明显了：

```java
// Lambda expression as function object (replaces anonymous class)
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

注意，lambda （`Comparator<String>`）的类型、它的参数（s1和s2都是字符串）类型和其返回值（int）的类型都没有出现在代码中。编译器使用称为**类型推断**的方法从上下文中推断这些类型。在某些情况下，编译器无法确定类型，这时你必须指定类型。类型推断的规则很复杂：它们在JLS中占了整整一章节[JLS, 18]。很少有程序员能够详细地理解这些规则，但这没有关系。尽可能省略所有lambda表达式的参数的类型，除非它们的存在使程序更清晰。如果编译器生成一个错误，告诉你它不能推断lambda表达式的参数的类型，那么就去指定参数类型。有时你可能需要将返回值或整个lambda表达式强制类型转换，但这种情况很少见。

关于类型推断，应该添加一个警告。条款26告诉你不要使用原生类型，条款29告诉你选择泛型类型，条款30告诉你选择泛型方法。当你使用lambdas的时候，这条建议就更加重要。因为编译器可以获得大多数的类型信息，这些类型信息可以使编译器对泛型进行类型推断。如果你不提供这些信息，那么编译器无法进行类型推断，并且你不得不手动地在你的lambdas中指定这些类型，这会大大增加lambda的冗余性。比如，如果将变量`words`声明为原始类型`List`而不是参数化类型`List<String>`，那么上面的代码片段无法通过编译。

顺便提一下，如果使用`comparator`的构造方法来代替lambda，代码片段中的比较器可以变得更加简洁（条款14.43）：

```java
Collections.sort(words, comparingInt(String::length));
```

事实上，利用Java 8中添加到List接口的`sort`方法，这个代码片段可以变得更简洁：

```java
words.sort(comparingInt(String::length));
```

语言中添加lambdas使得在以前没有意义的地方使用函数对象变得更加实际。比如，考虑条款34中的操作枚举类型。因为每个枚举的`apply`方法需要不同的行为，所以我们使用了指定类体的常量，并覆盖了每个枚举常量中的`apply`方法。要刷新你的记忆，以下代码：

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

条款34说到枚举实例字段比指定类体的常量更可取。使用前者代替后者的方法，lambda更容易实现指定常量的行为。只需将实现每个枚举常量行为的lambda传递给它的构造函数。构造函数将lambda存储在实例字段中，`apply`方法将调用转发给lambda。最终生成的代码比原始版本更简单、更清晰：

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

注意，我们对表示枚举常量行为的lambda使用了`DoubleBinaryOperator`接口。这是`java.util.function`中众多预定义的函数式接口之一（条款44）。它表示一个函数，该函数接受两个`double`型参数并返回一个`double`型结果。

看到基于lambda的操作枚举，你可能会想常量特定的方法体已经没有用了，但是并非如此。跟方法和类不同的是，**lambdas缺少方法名字和文档**。**如果一个计算不能自我解释，或者超过几行，就不要把它放在lambda中**。对于lambda，**一行是理想的，三行是合理的最大值**。如果你违反了这条规则，就会对程序的可读性造成严重损害。如果lambda很长或者很难读，**要么找到一种方法来简化它，要么重构程序来消除它**。而且，传递给枚举构造方法的参数是在静态上下文中计算的。因此，枚举构造方法中的lambda是无法访问枚举的实例成员的。如果枚举类型的常量特定的行为是不好理解的，或不能通过少量代码行数来实现的，或要访问实例字段或方法的，那么常量特定的类体依然依然是解决问题的办法。

同样，你可能认为匿名类在lambdas时代已经过时了。但事实是，有一些匿名类可以做的事情是lambdas不能做的。Lambdas仅限于函数式接口。如果你想创建一个抽象类的实例，可以使用匿名类，但不能使用lambda。同样地，你可以使用匿名类创建具有多个抽象方法的接口实例。**最后，lambda不能获得对自身的引用**。在lambda中，this关键字引用的是外部类实例，这通常是你想要的结果。在匿名类中，this关键字引用的是匿名类实例。**如果你需要在函数式对象的内部访问函数对象，则必须使用匿名类**。