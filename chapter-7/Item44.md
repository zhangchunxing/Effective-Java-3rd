# 优先选择标准的函数式接口

既然Java已经拥有了lambdas，因此编写APIs的最佳实践也发生了极大的变化。比如说，模板方法模式（子类重写父类中的原始方法来特化其父类的行为）就变得不再那么有吸引力了。现代化方式则是提供一个静态工厂或是构造方法，接收一个函数对象来实现相同的效果。更为通用的，你可以编写多个构造方法与方法，接收函数对象作为参数。选择正确的函数式参数类型则需要加倍小心。

考虑`LinkedHashMap`。你可以把它当做缓存，你可以通过重写其受保护的`removeEldestEntry`方法将该类用作缓存，`removeEldestEntry`方法会在每次将新的键添加到map中时被put方法所调用。当该方法返回true时，map就会将传递给该方法的最老的条目删除。

如下重写允许map一直增加到100个条目，接下来每次在添加新的键时就会将最老的条目删除，维护100个最近的条目：

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
	return size() > 100;
}
```

这项技术没什么问题，不过可以通过lambdas变得更好一些。如果LinkedHashMap是现在编写的，那么它就应该有一个静态工厂或是构造方法，接收一个函数对象。看看`removeEldestEntry`的声明，你可能会觉得函数对象应该接收一个`Map.Entry<K,V>`并返回boolean，不过事实却并非如此：`removeEldestEntry`方法调用size()来获得map中条目的数量，这样做之所以可行是因为`removeEldestEntry`是map中的一个实例方法。你传递给构造方法的函数对象并非map中的实例方法，无法对其进行捕获，因为在工厂或是构造方法被调用时，map尚不存在。因此，map必须将自身传递给函数对象，函数对象需要在其输入中接收map以及最老的条目。如果声明这样一个函数式接口，应该如下所示：

```java
// Unnecessary functional interface; use a standard one instead.
@FunctionalInterface
interface EldestEntryRemovalFunction<K,V> {
	boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

该接口使用起来没什么问题，不过你不应该用它，因为无需为了这个目的而声明一个新接口。`java.util.function`包提供了大量标准的函数式接口供你使用。**如果某个标准的函数式接口可以满足要求，那么你就应该使用它而不是新建函数式接口**。这会使得API更加易于学习，减少了学习成本，并且提供了更好的互操作性，因为很多标准的函数式接口都提供了有用的默认方法比如说，`Predicate`接口就提供了用于组合谓词的方法。对于我们这个`LinkedHashMap`示例来说，我们应该优先使用标准的`BiPredicate<Map<K,V>`,`Map.Entry<K,V>>`接口而非自定义的`EldestEntryRemovalFunction`接口。

`java.util.Function`包下有43个接口。你不能期望记住所有的接口，但是如果你记住了6个基础接口，你就可以在需要时派生出其余的接口。这些基础接口操作于对象的引用类型。`Opertaion`接口表示结果和参数类型是相同的函数。`Predicate`接口表示接受参数并返回布尔值的函数。`Funcation`接口表示参数和返回类型不同的函数。

`Supplier`接口表示不接受参数并返回值的函数。最后，`Consumer`表示一个函数，该函数接受一个参数，但什么也不返回，本质上是消费它的参数。6大基础函数式接口概述如下：

| 接口              | 函数式签名          | 例子                |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T,R>     | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

这6个基础接口中每一个都有3个变形，用于操作原生类型：`int`，`long`和`double`。变形的接口名字就是给基础接口的名字前加上原生类型的名称而得到的。比如，接受`int`类型的断言就叫`IntPredicate`，还有接受2个`long`型并且返回1个`long`型的二元操作叫做`LongBinaryOperator`。这些变种类型都没有参数化，除了`Function`变种，它的返回类型参数化了。比如，`LongFunction<int[]>`接受一个`long`型，返回一个`int[]`。

`Function`接口还有9中额外的变形，供结果类型是私有的时候使用。源类型和结果类型总是不同，因为从一个类型到自身的接口叫`UnaryOperator`。如果源类型和结果类型都是基本类型，那就给`Function`加上`Src To Result`前缀。比如，` LongToIntFunction `（6个变形）。如果源是基本类型，而结果是一个对象引用，那就给`Function`加上` <Src> ToObj `前缀，比如，`DoubleToObjFunction`（3个变形）。

其中有3个基础函数式接口的2个参数版本是有意义的：`BiPredicate<T,U>` , ` BiFunction<T,U,R>` 和 `BiConsumer<T,U>`。还有返回3个相关的原生类型的`BiFunction `变形：` ToIntBiFunction<T,U>`，`ToLongBiFunction<T,U>` `和ToDoubleBiFunction<T,U>`。`Consumer`的2个参数变形，接受一个对象引用和一个原生类型：`ObjDoubleConsumer<T> `，`ObjIntConsumer<T> `和` ObjLongConsumer<T> `。总共有9种2个参数版本的基础接口。

最后，还有`BooleanSupplier`接口。它是返回布尔值的`Supplier`的变体。除了`Predicate`及其4种变体形式支持布尔类型返回值，这是标准函数式接口名中唯一明确提到布尔类型的地方。`BooleanSupplier`接口和前面段落中描述的42个接口构成了所有43个标准函数式接口。诚然，这是一件难以接受的事情，而且并不完全正交。另一方面，你需要的大部分函数式接口都已经为你写好，并且它们的名称非常规则，所以当你需要时，你应该不会遇到太多麻烦。

大多数标准函数式接口的存在只是为了支持基本类型。不要试图使用包装类型的基础函数式接口来代替基本类型的函数式接口。虽然它可以工作，但是它违反了条款61的建议，“优先使用基本类型，而不要用包装类型”。在批量操作中使用包装类型的性能后果可能是致命的。

现在你知道了，你通常应该使用标准的函数式接口，而不是自己编写一个接口。但是什么时候你应该自己写呢?当然，如果标准接口中没有你需要的接口，那么你需要编写一个自己的接口。比如，你需要一个接受3个参数的谓词，或者抛出一个受检异常的谓词。但是有时候，即使你需要的接口和标准接口中的一个结构上是完全一致，你也应该编写自己的函数接口。

考虑我们的老朋友`Comparator<T>`，它在结构上与`ToIntBiFunction<T,T>`接口完全相同。即使后者接口在将前者添加到库中时已经存在，使用它也是错误的。有几个原因使`Comparator`值得拥有自己的接口。首先，每次在API中使用它时，它的名字就是优秀的文档，所以它使用得很多。第二，`Comparator`接口对有效实例的构成有严格的要求，有效实例包含它的通用契约。要实现该接口，你要保证遵守它的契约。第三，该接口大量地配备了有用的默认方法用来转换和组其它比较器。

如果你需要与`Comparator`共享一个或多个以下特性的函数式接口，你应该认真考虑编写一个专用的函数式接口，而不是使用标准接口：
 - 它将被广泛使用，并且可以从描述性名称中获益。

 - 它与之有很强的联系。

 - 它将受益于自定义默认方法。

如果你选择编写自己的函数式接口，请记住它是一个接口，因此在设计时应该非常小心（条款21）。

注意，`EldestEntryRemovalFunction`接口被`@FunctionalInterface`注解标记了。这种注解类型在本质上类似于`@Override`。这就是程序员意图的声明，有3个目的：它告诉类及其文档的读者，这个接口是为了启用lambdas而设计的；它让你保持诚实，因为接口不会编译，除非它有一个抽象的方法；它还可以防止维护人员在接口发展过程中意外地向接口添加抽象方法。始终使用`@FunctionalInterface`注解来注释你的函数式接口。

关于api中函数式接口的使用，值得关心的最后一点。不要提供在相同的参数位置接受不同函数式接口的多重载方法，这样做可能会在客户端中产生模棱两可的情形。这不仅仅是一个理论问题。`ExecutorService`的`submit`方法可以既可以接受`Callable<T>`参数，又可以接受`Runnable`参数。所以在编写客户端代码的时候，需要一个强制类型转换来表明正确的重载（条款52）。避免此问题的最简单方法就是不去写在相同参数位置上接收不同函数式接口的重载。这是条款52“谨慎地使用重载”中建议的一个特例。

总之，既然Java已经有了lambdas，那么在设计api时必须考虑到lambdas。在输入时接受函数式接口类型，并在输出时返回它们。通常，最好使用`java.util.function`提供的标准接口。但是要注意相对少见的情况，在这种情况下，你最好编写自己的函数接口。

