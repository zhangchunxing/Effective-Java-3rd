# 优先选择接口而非反射

核心反射工具`java.lang.reflect`，提供对任意类的编程访问。给定一个`Class`对象，你可以获得`Constructor`、`Method`和`Field`类型的实例，分别表示这个`Class`对象代表的类的构造方法，方法和字段。这些对象提供对类的成员名、字段类型、方法签名等的编程式访问。

而且， `Constructor`,  `Method` 和 `Field` 实例可以让你通过反射来操作他们底层的对应物。你可以通过调用`Constructor`、`Method`和`Field`实例上的方法来构造对应类的实例对象，调用其方法和访问其成员变量。例如，`Method.invoke`允许你在任何类的任何对象上调用任何方法（受通常的安全约束）。反射允许一个类使用另一个类，即使在编译前者时后者并不存在。然而，这种能力是有代价的：

* 你将失去编译期类型检查的好处，包括异常检查。如果一个程序尝试通过反射来调用一个不存在的或者不可访问的方法，那么它会在运行期失败除非你提前采取了特殊的预防措施。
* 通过反射来访问的代码显得笨拙和冗余。这种代码写起来很繁琐，可读性又差。
* 性能受损。通过反射的方法调用法比正常的方法调用要慢许多。由于有许多因素在起作用，所以很难确切地说慢了多少。在我的机器上，通过反射去调用一个无参的方法并返回一个`int`值。比正常调用要慢11倍。

有一些复杂的应用程序需要反射。比如，代码分析工具和依赖注入框架。即使是这类工具也正在摆脱反射，随着它的缺点变得越来越明显。如果你对你的应用程序是否需使用反射存有疑虑，那么可能不需要。

仅在一些非常有限的形式中使用反射，你是可以获得反射的许多好处的，同时花费很少的代价。比如说，有些程序它使用的类是在编译期无法获得的，那么你可以在编译期通过一个合适的接口和超类来引用这个类（条款64）。如果是这种情况，你就可以通过反射来创建这个实例，并通过它们的接口或超类正常地访问它们。

比如，这是一个创建`Set<String>`实例的程序，它的具体类型由第一个命令行参数指定。程序会将剩下的命令行参数插入到这根`Set`中并打印它。除了第一个参数，程序会把剩余的参数打印出来，并做去重。但是，打印这些参数的顺序取决于第一个参数中指定的类型。如果你指定`java.util.HashSet`，那么它们会被随机打印出来；如果你指定`java.util.TreeSet`，那么它们是按字母顺序打印的，因为`TreeSet`中的元素是有序的：

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

虽然这个程序只是一个玩具，但它演示的技术非常强大。玩具程序可以很容易地转换成一个通用的集合测试器，通过积极地操作一个或多个实例并检查它们是否遵守`Set`契约来验证指定的`Set`实现。类似地，它可以转变成一个通用的集合性能分析工具。实际上，该技术已经足够强大，可以实现成熟的服务提供者框架（条款1）。

这个例子展示了反射的两个缺点。首先，这个示例在可能生成6个不同运行期异常，如果没有使用反射实例化，所有这些异常都是编译期错误。(为了好玩，你可以通过传入适当的命令行参数来让程序生成6个异常中的每一个）。第二个缺点是，从类名生成类实例需要25行冗长的代码，而构造函数调用只需一行即可。通过捕获`ReflectiveOperationException `（Java 7中引入的各种反射异常的超类），可以减少程序的长度。这两个缺点都局限于实例化对象的程序部分。一旦实例化，该集合就与任何其他集合实例难以区分。在实际的程序中，通过限制反射的使用，大量的代码可以不受这些影响。

