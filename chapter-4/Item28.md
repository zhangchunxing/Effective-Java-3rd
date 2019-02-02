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

由于这些根本差别，数组与泛型是不能混用的。比如说，**创建泛型类型数组、参数化类型数组以及类型参数数组都是不合法的**。因此，如下数组创建表达式都是不合法的：`new List<E>[]`， `new List<String>[]`， `new E[]`。所有这些都会导致编译期的泛型数组创建错误。

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

比如说，假设你要编写一个`Chooser`类，其构造方法接收一个集合，它还提供了一个方法，随机返回集合中的一个元素。根据向构造方法所传递的集合的不同，你可以将该选择器作为一个骰子、一个魔术八球，或是一个蒙特卡罗模拟的数据源。如下是不使用泛型的一个简化的实现：

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

要想使用这个类，你需要在每次调用`choose`方法时将其返回值从`Object`转换为所需的类型，如果转换错误，那么在运行期就会失败。根据条款29，我们将`Chooser`改为泛型的。变化的部分如粗体所示：

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

如果编译这个类，你会看到如下错误消息：

```java
Chooser.java:9: error: incompatible types: Object[] cannot be
converted to T[]
choiceArray = choices.toArray();
							  ^
where T is a type-variable:
  T extends Object declared in class Chooser
```

你可能觉得没什么大不了的，将`Objec`数组转换为`T`数组就行了：

```java
choiceArray = (T[]) choices.toArray();
```

这么做会消除掉错误，但却会收到一个警告：

```java
Chooser.java:9: warning: [unchecked] unchecked cast
choiceArray = (T[]) choices.toArray();
                                    ^
required: T[], found: Object[]
where T is a type-variable:
T extends Object declared in cass Chooser
```

编译器告诉你，它无法保证运行期类型转换的安全性，因为程序不知道`T`表示的类型是什么——记住，元素类型信息会在运行期擦除掉泛型。那么程序会正常运行么？是的，不过编译器却不敢保证。你可以自己保证这一点，将证据放在注释中，并使用注解来压制警告，不过最好还是从根源上消除掉警告（条款27）。

为了消除掉未检查的类型转换警告，请使用列表来代替数组。如下这个版本的`Chooser`类会正常通过编译，并且没有错误和警告信息：

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

该版本有点啰嗦，运行速度也可能稍微有点慢，不过却是值得的，因为运行期不会再抛出`ClassCastException`了。

总结一下，数组与泛型拥有非常不同的类型规则。数组是协变且具化的；泛型是逆变且类型擦除的。因此，数组提供了运行期的类型安全，而非编译期的类型安全；泛型则是完全相反的。通常，数组和泛型不能很好地混合。如果混合使用了数组与泛型，并且收到了编译期的错误或是警告，那么首先就应该考虑使用列表来代替数组。