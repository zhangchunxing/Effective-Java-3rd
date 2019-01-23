# 使用接口只用来定义类型

当一个类实现了一个接口时，该接口就作为一个类型，可用于引用该类的实例。一个类实现了一个接口时，我们就可以说客户端可以对该类的实例做什么事情。除此之外，定义接口用作其他任何目的都是不恰当的。

不符合该原则的一种接口就是所谓的**常量接口**。这种接口不包含任何方法；它仅仅包含`static final`字段，每个都会输出为一个常量。使用这些常量的类实现接口以便不用再使用类名来引用常量名。参见如下示例：

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

**常量接口模式是对接口的滥用**。一个类在内部使用一些常量是一种实现细节。实现一个常量接口会导致这种实现细节泄露给类所导出的API。对于类的用户来说，类实现了一个常量接口并不重要实际上，这甚至会让他们感到困惑。更糟糕的是，它表名了一个承诺：在未来的发布中，如果类被修改了，不再需要这些常量，但它还得实现这个接口以保证二进制兼容性。如果非终态类实现了一个常量接口，那么它的所有子类的命名空间都会被接口中的常量所污染。

Java平台库中有几个常量接口，例如`Java .io.ObjectStreamConstants`。这些接口应该被当做反例，而不应该被模仿。

如果你想提供常量，这里有几个合理的选择。如果常量跟现有类或接口有很强的关联，那么你应该将他们添加到类或接口中。比如说，所有原生数字的装箱类，如`Integer`和`Double`，都提供了`MIN_VALUE`和`MAX_VALUE`常量。如果常量适合于作为枚举类型的成员，那就应该使用枚举类型将其导出（条款34）。否则，你应该使用一个不可实例化的辅助类来导出常量（条款4）。如下是之前介绍的`PhysicalConstants`示例的辅助类版本：

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

请注意在数字字面值中所使用的下划线（_）。下划线是在Java 7中引入的，它对于数字字面值没有任何影响，但如果使用得当，则会增强可读性。如果包含了5个以上连续的数字，那就请考虑对数字字面值（无论是定点数还是浮点数）使用下划线。对于十进制数字来说（无论是定点数还是浮点数），你都应该使用下划线将数字划分为3个一组，表示1000的正乘方与负乘方。

一般来说，辅助类会要求客户端使用类名来引用常量名，比如说，`PhysicalConstants.AVOGADROS_NUMBER`。如果大量使用某个辅助类所导出的常量，那就可以通过使用`static import`省去类名。

```java
// Use of static import to avoid qualifying constants
import static com.effectivejava.science.PhysicalConstants.*;
public class Test {
    double atoms(double mols) {
    return AVOGADROS_NUMBER * mols;
}
...
// Many more uses of PhysicalConstants justify static import
}
```

总结一下，接口只应该用作定义类型，不应该仅仅用于导出常量。