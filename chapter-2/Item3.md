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