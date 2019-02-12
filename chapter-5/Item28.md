# 优先选择列表而非数组

数组在两个方面与泛型类型存在着不同。首先，数组是**协变的**。这个词语看起来很高深，但实际上却很简单，它表示如果`Sub`是`Super`的子类型，那么数组类型`Sub[]`就是`Super[]`的子类型。与之相反，泛型是逆变的：对于任意两个不同的类型`Type1`与`Type2`来说，`List<Type1>`既不是`List<Type2>`的子类型，也不是其父类型[JLS, 4.10; Naftalin07, 2.5]。你可能觉得这意味着泛型是有缺陷的，不过有缺陷的却是数组。如下代码片段是合法的：

```java
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
```
但如下代码片段却是非法的：
```java
// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

无论哪种方式，你都不能将`String`放到`Long`容器中，不过对于数组来说，你是在运行期发现的错误；而对于列表来说，你是在编译期发现的错误。当然了，你更愿意在编译期发现错误。

数组与泛型之间的第二个主要差别在于数组是具化的[JLS, 4.7]。这意味着数组在运行期是知道其元素类型的，并且会强制施加类型限制。如前所述，如果试图将`String`放到`Long`数组中，那就会抛出`ArrayStoreException`异常。与之相反，泛型是通过类型擦除实现的[JLS,4.6]。这意味着泛型只会在编译期施加类型限制，在运行期则会丢弃（或是叫做擦除）其元素类型。类型擦除使得泛型类型能够与没有使用泛型的遗留代码互操作（条款26），这确保了Java 5中向泛型的平滑迁移。

由于这些根本差别，数组与泛型是不能混用的。比如说，创建泛型类型数组、参数化类型数组以及类型参数数组都是不合法的。因此，如下数组创建表达式都是不合法的：`new List<E>[]`， `new List<String>[]`， `new E[]`。所有这些都会导致编译期的泛型数组创建错误。

为什么说创建泛型数组是不合法的呢？因为它不是类型安全的。如果合法，那么由编译器在正确的程序中所生成的类型转换就有可能在运行期出现失败，抛出`ClassCastException`异常。这违背了泛型类型系统所提供的基本保证。

具体来说说，考虑如下代码片段：

```java
// Why generic array creation is illegal - won't compile!
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42);              // (2)
Object[] objects = stringLists;                   // (3)
objects[0] = intList;                             // (4)
String s = stringLists[0].get(0);                 // (5)
```

假设第1行代码（创建了一个泛型数组）是合法的。第2行代码创建并初始化了一个包含单个元素的`List<Integer>`。第3行代码将`List<String>`数组存储到了一个`Object`数组变量中，这是合法的，因为数组是协变的。第4行代码将`List<Integer>`存储到`Object`数组中唯一的元素中，这么做是可以的，因为泛型是通过类型擦除实现的：`List<Integer>`实例的运行期类型就是`List`而已，`List<String>[]`实例的运行期类型就是`List[]`，这样该赋值并不会导致`ArrayStoreException`。现在，我们遇到了麻烦。我们将一个`List<Integer>`实例存储到一个数组中，而该数组的声明是仅能持有`List<String>`实例。在第5行代码中，我们从该数组中的唯一一个列表中获取到其中的唯一一个元素。编译器会自动将获取到的元素转换为`String`，不过它却是个`Integer`，这样运行期就抛出了`ClassCastException`。为了防止这种情况发生，第1行代码（即创建泛型数组这一行）就必须要生成一个编译期错误。

从技术角度来说，诸如`E`、`List<E>`及`List<String>`这样的类型叫做非具化类型[JLS, 4.7]。直观上来说，非具化类型指的是运行期所包含的信息要少于编译期所包含的信息。由于类型擦除的原因，可以具化的唯一参数化类型就是无边界的通配符类型，如`List<?>`与`Map<?, ?>`（条款26）。创建无边界通配符类型的数组是合法的，不过这样做其实没什么用处。

禁止创建泛型数组挺恼人的。比如说，这意味着我们通常无法让泛型集合返回一个其元素类型的数组（不过，请参考条款33了解如何部分地解决这个问题）。此外，在同时使用可变参数方法（条款53）与泛型类型时，你会遇到令人困惑的警告信息。这是因为，每次调用可变参数方法时，系统都会创建一个数组来持有可变参数。如果该数组的元素类型并非具化的，那就会遇到警告。`SafeVarargs`注解可用来解决这个问题（条款32）。

当遇到泛型数组创建错误或是在转换为数组类型时出现未检查的类型转换警告时，最佳的解决方案通常是使用集合类型`List<E>`来代替数组类型`E[]`。你可能会因此失去一些简洁性或是性能，不过得到的却是更好的类型安全性与互操作性。

例如，假设你想编写一个`Chooser`类，该类的构造函数接受一个集合，而单个方法随机返回所选择的集合的一个元素。根据传递给构造函数的集合，你可以将选择器用作游戏骰子、魔术8球或蒙特卡洛模拟的数据源。下面是一个没有泛型的简单实现：

```java
// Chooser - a class badly in need of generics!
public class Chooser {
    private final Object[] choiceArray;
        public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
	}
}
```

使用这个类，每次你调用`choose`方法，你都不得不把方法的返回值从`Object`类型转成声明的类型，并且如果你得到了一个错误的类型，那么类型转换会在运行期间失败。考虑到条款29的建议，我们试图修改`Chooser`，使其具有通用性。变化以粗体显示：

```java
// A first cut at making Chooser generic - won't compile
public class Chooser<T> {
    private final T[] choiceArray;
    public Chooser(Collection<T> choices) {
    choiceArray = choices.toArray();
}
// choose method unchanged
}
```

如果你尝试编译这个类，你会得到这样的错误信息：

```java
Chooser.java:9: error: incompatible types: Object[] cannot be
converted to T[]
choiceArray = choices.toArray();
							  ^
where T is a type-variable:
  T extends Object declared in class Chooser
```

你会说，这不是大问题，没什么大不了的，你会说，我把对象数组转换成`T`数组：

```java
choiceArray = (T[]) choices.toArray();
```

这样做消除了错误，但你会得到一个警告：

```java
Chooser.java:9: warning: [unchecked] unchecked cast
choiceArray = (T[]) choices.toArray();
                                    ^
required: T[], found: Object[]
where T is a type-variable:
T extends Object declared in cass Chooser
```

编译器告诉你，它不能保证在运行时强制转换的安全性，因为程序不知道类型`T`代表什么——记住，元素类型信息在运行时从泛型中擦除。这个计划行得通吗？是的，但是编译器不能证明它。你可以向自己证明这一点，在注释中添加说明，并使用注解压制警告，但是最好消除导致警告的原因(条款27)。若要消除未检查的强制转换警告，请使用列表而不是数组。这是`Chooser`类的一个版本，编译时没有错误或警告：

```java
// List-based Chooser - typesafe
public class Chooser<T> {
    private final List<T> choiceList;
    public Chooser(Collection<T> choices) {
    	choiceList = new ArrayList<>(choices);
    }
    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
	}
}
```

这个版本稍微冗长一些，可能稍微慢一些，但是为了让你安心，在运行时不会得到`ClassCastException`是值得的。

总之，数组和泛型有非常不同的类型规则。数组是协变的、具体化的；泛型是不变的和被擦除的。因此，数组提供运行时类型安全，而不是编译时类型安全，泛型则相反。通常，数组和泛型不能很好地混合。如果你发现将它们混合在一起，并得到编译时错误或警告，那么你的第一反应应该是用列表替换数组。