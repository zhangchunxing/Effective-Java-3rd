# 优先选择列表而非数组

数组与泛型类型主要有两个方面不同。首先，数组是**协变的**。这个听起来可怕的单词仅仅是指，如果`Sub`是`Super`的子类型，那么数组类型的`Sub[]`是数组类型的`Super[]`的一个子类型。相反地，泛型是不变的：对于任意两种不同类型的`Type1`和`Type2`， `List<Type1>`既不是`List<Type2>`的子类型，也不是`List<Type2>`的父类型[JLS, 4.10;Naftalin07, 2.5]。你可能会认为这意味着泛型是有缺陷的，但是经过论证后，有缺陷的是数组。下面这段代码是合法的：

```java
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
```
但是下面是错误的：
```java
// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

两种方式你都不能将一个`String`类型放到一个`Long`类型的容器中，但是使用数组你可以在运行期发现你的错误；使用列表，你可以在编译期就发现你的错误。当然，你最好在编译期就发现错误。

数组和泛型之间的第二个主要区别是数组是具体化的[JLS, 4.7]。这意味着数组在运行时知道并强制它们的元素类型。如前所述，如果你尝试将字符串放入一个长整型的数组中，你将得到`ArrayStoreException`。相比之下，泛型是通过擦除实现的[JLS, 4.6]。这意味着泛型只在编译时执行类型约束，并在运行时丢弃(或擦除)元素类型信息。擦除允许泛型类型与不使用泛型的遗留代码自由地互相操作(条款26)，确保在Java 5中平稳地过渡到泛型。

由于这些基本差异，数组和泛型不能很好地混合在一起。例如，创建泛型类型、参数化类型或类型参数的数组是非法的。因此，这些数组创建表达式都是不合法的：`new List<E>[]`， `new List<String>[]`， `new E[]`。所有这些都会在编译时导致泛型数组创建错误。

为什么创建泛型数组是非法的呢？因为它不是类型安全的。如果合法，则编译器在其他正确的程序中生成的强制类型转换可能在运行时失败，并带有`ClassCastException`。这将违反泛型类型系统提供的基本保证。

为了使这更具体，请考虑以下代码片段：

```java
// Why generic array creation is illegal - won't compile!
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42); // (2)
Object[] objects = stringLists; // (3)
objects[0] = intList; // (4)
String s = stringLists[0].get(0); // (5)
```

假设第1行创建了一个泛型数组，它是合法的。第2行创建并初始化一个包含单个元素的`List<Integer>`。第3行将一个元素类型为`List<String>`的数组存储到一个元素类型为`Object`的数组中，这是合法的因为数组是协变的。第4行将`List<Integer>`存储到`Object`数组的一个元素，因为泛型是通过擦除实现的，所以这样做是可以的：一个`List<Intege>`类型的实例运行时的类型就是`List`，并且` List<String>[]`的实例运行时的类型就是`List[]`，因此这样的赋值不会生成`ArrayStoreException`。现在我们陷入了麻烦。我们将一个`List<Integer>`的实例存储到了一个被声明为仅仅容纳` List<String> `实例的数组中。在第5行中，我们从这个数组的唯一列表中检索唯一元素。编译器自动将检索到的元素转换为字符串，但它是一个整数，因此我们在运行时得到`ClassCastException`。为了防止这种情况发生，第1行(它创建一个泛型数组)必须生成一个编译时错误。

`E`、`List<E>`和`List<String>`等类型在技术上称为非具体化类型[JLS, 4.7]。直观地说，非具体化类型的运行时表示包含的信息少于其编译时表示。由于擦除，唯一可具体化的参数化类型是无边界通配符类型，如`List<?>`和` Map<?,?>`(26项)。创建无边界通配符类型的数组是合法的，但很少有用。

禁止创建泛型数组可能很烦人。例如，这意味着泛型集合通常不可能返回其元素类型的数组(但是部分解决方案参见第33项)。这也意味着在结合使用有可变参数的方法(第53项)和泛型类型时，你会得到令人困惑的警告。这是因为每次调用可变参数的方法时，都会创建一个数组来保存参数。如果该数组的元素类型不可具体化，则会得到警告。`SafeVarargs`注解可用于解决此问题(项目32)。

当你在转换为数组类型时遇到泛型数组创建错误或未检查的转换警告时，最好的解决方案通常是使用集合类型`List<E>`，而不是数组类型`E[]`。你可能会牺牲一些简洁性或性能，但作为交换，你可以获得更好的类型安全性和互操作性。

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
T extends Object declared in class Chooser
```

