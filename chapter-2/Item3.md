## 通过私有的构造方法或者一个枚举类型来使用单例属性

单例对象是一个仅仅只会实例化一次的类。单例对象通常表示一个无状态对象，例如一个函数(项目24)或一个本质上惟一的系统组件。使类成为单例会使测试它的客户端变得困难，因为不可能用模拟实现代替单例，除非它实现一个接口作为它的类型。

实现单例有两种常见的方法。两者都基于保持构造函数为私有，并对外提供公共静态成员以提供对唯一实例的访问。在这种方法中，这个成员是一个不可变字段:

```java
// Singleton with public final field
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public void leaveTheBuilding() { ... }
}
```

只调用私有构造函数一次，以初始化公共静态final字段elvi . instance。不提供公有的或者受保护的构造函数保证了全局唯一性：当Elvis类初始化的时候，仅仅只会有一个Elvis实例存在。普通的客户端不能修改该对象到底任何属性，但有一点注意是：有权限的客户端可以通过反射的方式调用AccessibleObject.setAccessible这个方法改变构造函数访问修饰来的目的，然后在客户端调用构造函数实现实例化。如果需要防范这种攻击，请修改构造函数，使其在被要求创建第二个实例时抛出异常。

第二种实现单例模式的方法是，提供一个公有的静态工厂方法：

```java
// Singleton with static factory
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }
	public void leaveTheBuilding() { ... }
}
```

所有调用Elvis类的getInstance方法，返回相同的对象引用，并且不会有其它的Elvis对象被创建。（但同样有上面提到的警告）

公共字段方法的主要优点是API明确表示该类是单例：public static字段是final的，所以它将始终包含相同的对象引用。 第二个优点是它更简单。静态工厂方法的一个优点是它可以让你灵活地改变你的想法，即该类是否要设计成单例的而不用更改变其API。工厂方法返回唯一的实例，但可以修改它以返回调用它的每个线程的单独实例。所以，如果你的项目需要，你可以定义一个通用的单例工厂。使用静态工厂方法最后一个好处就是，方法引用可以当做一个Supplier实例使用，例如，Elvis::instance就是一个Supplier<Elvis>  的实例。除非这些优点有其它相关连的地方，不然提供公有字段的方法更适合。

通过上面提到的方法，来实现单例对象的序列化，仅仅在声明中添加实现Serializable接口是不够的。为了保证单例，将所有实例字段声明为transient，并提供一个readResolve方法(第89条)。否则，每次反序列化一个序列化的实例时，都会创建一个新的实例，在我们的示例中，这会导致伪造的Elvis。为了阻止这样的事发生，给Elvis类添加一个readResolve方法。

```java
// readResolve method to preserve singleton property
private Object readResolve() {
    // Return the one true Elvis and let the garbage collector
    // take care of the Elvis impersonator.
    return INSTANCE;
}
A third way to implement a singleton is to declare a single-element enum:
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```

这个方法跟提供公有的字段方法很类似，但它更简洁，提供免费的可序列化机制和提供坚固的保证阻止多实例化对象，甚至面对复杂的序列化和反射的攻击下。这种方法可能看起来不太自然，但是拥有单元素的枚举类型可能是实现单例模式的最佳实践。注意，如果你的单例对象必须继承除枚举类型之外的超类，则不能使用这种方法(因为可以声明枚举来实现接口)。