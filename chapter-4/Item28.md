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

