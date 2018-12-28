# 仅使用接口定义类型

当一个类实现了一个接口时，这个接口作为一种类型，可以用来引用一个类的实例。所以，实现了接口的那个类应该说明客户端处理这个类的实例。为任何其他的目的而定义的接口都是不恰当的。

有一种接口违反了这个规定，它就是**常量接口**。这种接口它不包含方法；它仅仅由`static final`修饰的常量字段组成。想使用这些常量的类会实现这个接口，这样可以避免通过类名来访问常量名的情况。如下所示：

```java
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
	// Avogadro's number (1/mol)
	static final double AVOGADROS_NUMBER = 6.022_140_857e23;
	// Boltzmann constant (J/K)
	static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
	// Mass of the electron (kg)
	static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

**常量接口模式是对接口的糟糕使用**。内部使用常量的类成为一个实现细节。实现常量接口会导致实现细节泄露给了类的对外的API。一个类实现了一个常量接口，这对使用该类的用户没有任何影响。事实上，这样做会让用户感到困惑。更糟糕的是，它代表了一个承诺：如果在一个未来版本中，这个类被修改了，结果它再也不需要使用这些常量了，但是它仍然必须实现这个接口以确保二进制的兼容性。如果一个非终态（nonfinal）类实现了一个常量接口	，那么它的所有子类的命名空间都会被接口中的常量污染。

Java平台库中有几个常量接口，例如`Java .io.ObjectStreamConstants`。这些接口应该被当做反例，而不应该被模仿。

如果你想提供常量，这里有几个合理的选择。如果常量跟现有类或接口有很强的关联，那么你应该将他们添加到类或接口中。比如说，所有的被装箱过的数字原生类型的类，如`Integer`和`Double`，都提供了`MIN_VALUE`和`MAX_VALUE`常量。如果常量最好是看做一个枚举类型的成员，那么你应该通过一个枚举类型来提供它们。否则，你应该通过一个不可实例化的工具类来提供常量。这里有一个很早之前看过的物理常量类的例子的工具类版本。

```java
// Constant utility class
package com.effectivejava.science;
public class PhysicalConstants {
    private PhysicalConstants() { } // Prevents instantiation
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

