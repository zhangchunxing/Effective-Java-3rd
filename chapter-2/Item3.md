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

