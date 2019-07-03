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

`java.util.function`中有43个接口。我们不能要求你将他们都记住，不过如果记住6个基本的接口，那么在需要时就可以扩展了。这些基本的接口是对对象引用类型进行操作的。`Operator`接口表示结果与参数类型都一样的函数。`Predicate`接口表示接收一个参数并返回boolean的函数。`Function`接口表示参数与返回类型不同的函数。`Supplier`接口表示不接收参数并返回（或是『提供』）一个值的函数。最后，`Consumer`表示接收一个参数并且不返回结果的函数，我们称之为消费掉其参数。这6个基本的函数式接口如下表所示：

| 接口              | 函数式签名          | 例子                |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T,R>     | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

对于这6个基本的接口来说，每一个都有3个变种，分别用于操纵原生类型`int`，`long`和`double`。其名字衍生自这些基本的接口，只不过每一个都在前面加上了一个原生类型。比如说，接收一个`int`的`predicate`叫做`IntPredicate`，接收两个`long`值并返回一个`long`的二元操作叫做`LongBinaryOperator`。这些变种都不是参数化的，除了`Function`变种，它是通过返回类型进行参数化的。比如说，`LongFunction<int[]>`接收一个`long`并返回一个int[]。

`Function`接口还有另外9个变种，用在结果类型为原生类型的场景下。其源与结果类型总是不同的，因为从一种类型得到相同类型的函数是`UnaryOperator`。如果源与结果类型都是原生类型，那就在`Function`前加上`SrcToResult`，比如说`LongToIntFunction`。比如，` LongToIntFunction `（6个变形）。如果源是个原生类型，结果是个对象引用，那就在`Function`前加上`<Src>ToObj`，比如说`DoubleToObjFunction`。

有3个基本的函数式接口还存在两参数版本，他们是`BiPredicate<T,U>`、`BiFunction<T,U,R>`与`BiConsumer<T,U>`。还有返回3个相关的原生类型的`BiFunction`变种，他们是`ToIntBiFunction<T,U>`、`ToLongBiFunction<T,U>`与`ToDoubleBiFunction<T,U>`。有两参数版本的`Consumer`，接收一个对象引用和一个原生类型：`ObjDoubleConsumer<T>`、`ObjIntConsumer<T>`与`ObjLongConsumer<T>`。总的来说，这些基本接口有9个两参数版本。

最后，有一个`BooleanSupplier`接口，它是`Supplier`的变种，返回`boolean`值。这是在所有标准的函数式接口名当中，唯一一个显式使用了boolean类型的；不过，boolean返回值是通过Predicate及其4个变种来得到支持的。`BooleanSupplier`接口及上一段提及的42个接口共同构成了全部43个标准的函数式接口。坦白地说，确实有些多，不过并不太可怕。另一方面，你所需要的大多数函数式接口已经是现成的了，他们的名字很直观，需要时很容易就能找到。

大多数标准的函数式接口存在的唯一目的在于为原生类型提供支持。请不要使用原生类型包装类的基本函数式接口，而应该使用原生类型的函数式接口。虽然它可以使用，不过却违背了条款61的建议：『优先选择原生类型而非包装类型』。对于大批量操作来说，使用包装类型的性能影响是非常可观的。

现在，你知道应该优先选择标准的函数式接口而不是编写自己的了。不过，何时应该编写自己的呢？当然了，如果标准的函数式接口无法满足所需，那就需要编写自己的了；比如说，如果需要一个接收3个参数的谓词，或是能够抛出受检查异常的谓词。不过，有时即便既有的标准函数式接口能够满足所需，你还是需要编写自己的函数式接口。

考虑一下我们的老朋友`Comparator<T>`，它在结构上等价于`ToIntBiFunction<T,T>`接口。虽然在向库中添加`ToIntBiFunction<T,T>`时，`Comparator<T>`就已经存在了，但使用`ToIntBiFunction<T,T>`却并非一个正确之选。有几个理由会让我们继续选择使用`Comparator`接口。首先，每次在API中使用时，其名字都提供了很好的文档，它用得也很多。其次，`Comparator`接口对什么能够构成有效的实例拥有很强的约束，这形成了其契约。通过实现该接口，你可以确保遵守其契约。第三，该接口拥有一些有用的默认方法来转换和组合比较器。

如果你所需要的函数式接口与`Comparator`相比存在如下几个共性，那么你就应该考虑编写自己的而不是使用标准的了：
 - 使用广泛，能够受益于一个描述性好的名字。

 - 拥有与之相关的一个很强的契约。

 - 能够从自定义的默认方法获益。

如果决定编写自己的函数式接口，请记住它是个接口，因此在设计时需要非常小心（条款21）。

注意到`EldestEntryRemovalFunction`接口被标记为了`@FunctionalInterface`注解。该注解类型本质上类似于`@Override`。它表明了程序员的意图，有3个目的：它告诉类与文档的读者，接口被设计为用于lambda之目的；它会让你保持诚实，因为除非接口中只有一个抽象方法，否则将无法编译通过；防止接口维护者在接口演化过程中不小心添加了其他抽象方法。请始终对你的函数式接口使用`@FunctionalInterface`注解。

关于在APIs中使用函数式接口还有最后一点要进行说明。请不要提供在相同的参数位置接收不同的函数式接口的重载方法，这样会导致客户端使用起来很混乱。这并非仅仅是个理论上的问题。`ExecutorService`的`submit`方法可以接收一个`Callable<T>`或是`Runnable`，我们可以编写一个客户端程序，进行强制类型转换来标识正确的重载（条款52）。不过，规避这一问题最简单的方式则是不要编写在相同的参数位置接收不同的函数式接口的重载方法。这是条款52所给出建议的一个特例。

总结一下，既然Java已经拥有了lambdas，因此在设计APIs时就需要考虑到lambdas。在输入中接收函数式接口类型，在输出中返回函数式接口类型。一般来说，最好使用`java.util.function`中所提供的标准接口；不过值得注意的是，有时我们还是需要编写自己的函数式接口。



