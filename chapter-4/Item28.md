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

