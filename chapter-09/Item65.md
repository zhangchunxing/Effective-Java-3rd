# 优先选择接口而非反射

核心的反射功能（`java.lang.reflect`）提供了以编程的方式访问任意类的能力。给定一个`Class`对象，你可以获取到由该`Class`实例所表示的类中的构造方法、方法与字段所对应的`Constructor`、`Method`与`Field`实例。这些对象提供了以编程的方式访问类中成员名字、字段类型与方法签名等信息的能力。

此外，你还可以通过`Constructor`、`Method`与`Field`实例以反射的形式操纵其底层的对应之物：你可以通过调用`Constructor`、`Method`与`Field`实例上的方法来构建底层类的实例、调用其方法以及访问其字段。比如说，你可以通过`Method.invoke`来调用任意类的任意对象的任意方法（会受到通常的安全限制）。反我们可以通过反射让一个类使用另外一个类，即便前者在编译时后者尚不存在亦可。不过，这种强大的能力却伴随着一些代价：

*  失去了编译期类型检查的所有好处，包括异常检查。如果程序通过反射的方式调用一个不存在或是无权访问的方法，那么除非采取了一些预防措施，否则程序会在运行期出错。
* 用于执行反射访问的代码非常笨拙和啰嗦。这些代码写起来很繁琐，读起来也费劲。
* 性能问题。反射方法调用要比通常的方法调用慢很多。究竟慢多少很难精确地给出具体数字，因为这里面涉及到很多因素。在我的机器上，使用反射的方式调用一个无输入参数、返回一个`int`值的方法要比正常调用慢11倍。

有一些复杂的应用需要用到反射。比如说代码分析工具以及依赖注入框架等。甚至这样的工具后来也逐渐不再使用反射了，因为其缺点变得越来越明显。如果对你的应用是否需要使用反射存疑，那么答案是很有可能不需要。

可以通过非常有限的形式来使用反射，这样既可以获取到反射的很多好处，又能够避免其代价。对于需要在编译期使用尚不存在的类的很多程序来说，如果存在恰当的接口或是父类，那就可以通过它来引用所需使用的类（条款64）如果情况如此，那就可以通过反射的方式创建实例，然后通过其接口或是父类来正常访问他们。

比如说，如下程序创建了一个`Set<String>`实例，其类是通过第一个命令行参数所指定的。程序会将剩余的命令行参数插入到集合中并打印出来。无论第一个参数是什么，程序都会将剩余参数去重后打印出来。不过，参数的打印顺序取决于第一个参数所指定的类名。如果指定的是`java.util.HashSet`，那么打印顺序就是随机的；如果指定的是`java.util.TreeSet`，那就会按照字母表顺序打印，因为`TreeSet`中的元素是有序的：

```java
// Reflective instantiation with interface access
public static void main(String[] args) {
    // Translate the class name into a Class object
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) // Unchecked cast!
        Class.forName(args[0]);
    } catch (ClassNotFoundException e) {
        fatalError("Class not found.");
    }
    // Get the constructor
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("No parameterless constructor");
    }
    // Instantiate the set
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("Constructor not accessible");
    } catch (InstantiationException e) {
        fatalError("Class not instantiable.");
    } catch (InvocationTargetException e) {
        fatalError("Constructor threw " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Class doesn't implement Set");
    }
    // Exercise the set
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}
```

虽然这段程序仅仅只是个玩具而已，但它所演示的技术却是相当强大的。该玩具程序可以轻松转换为一个通用的集合测试器，通过操纵一个或是多个实例以及检查他们是否遵循`Set`契约来验证指定的`Set`实现。与之类似，它还可以轻松转换为通用的集合性能分析工具。实际上，这项技术非常强大，足以用于实现一个功能全面的服务提供者框架（条款1）。一般来说，该项技术是你在反射方面所需的全部。

该示例展示了反射的两个缺点。首先，示例会在运行期生成6种不同的异常，如果没有用到反射实例，那么所有这些异常都应该是编译期错误（出于好玩的目的，你可以通过传递恰当的命令行参数来生成每一种异常）。第2个缺点是，该程序使用了25行冗长的代码来根据名字生成类的实例，而构造方法调用一行就可以做到。可以通过捕获`ReflectiveOperationException`来减少程序的长度，这是Java 7所引入的各种反射异常的父类。这两个缺点都集中在实例化对象部分。这两个缺点都局限于实例化对象的程序部分。一旦实例化完毕，集合就与其他`Set`实例别无二致了。在真实的程序中，大部分代码都不会受到这种对反射的有限使用的影响。

如果编译该程序，你会收到一个未检查的类型转换警告。该警告是合理的，因为即便具名类不是`Set`实现，类型转换为`Class<? extends Set<String>>`也是可以成功的，在这种情况下，当实例化类时，程序会抛出`ClassCastException`。只不过在这种情况下，程序在实例化类时就会抛出`ClassCastException`。若想了解如何压制该警告，请参考条款27。

对于反射的一种合理的用法是管理一个类对运行期可能缺失的其他类、方法或是字段的依赖。如果你在编写一个包，它需要针对其他包的多个版本来运行，那么这种方式就是很有用的了。该项技术的做法是针对能够支持其运行的最小环境来编译你的包，通常是最老的版本，然后通过反射的方式访问新的类或是方法。要想做到这一点，如果想要访问的新的类或是方法在运行期不存在，那就需要采取一些恰当的动作。恰当的动作包括使用别的方式来完成相同的目标或是操纵精简的功能。

总结一下，反射是一项强大的功能，用在某些复杂的系统编程任务中，不过它有很多缺点。如果所编写的程序需要用到编译期未知的类，那就应该尽量只通过反射来实例化对象，然后使用编译期已知的接口或是父类来访问对象。