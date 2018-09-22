## 减少可变性

不可变类就是一个实例不能被修改的类。每个实例中包含的所有信息在对象的生命周期内都是固定的，因此永远不会观察到任何更改。Java平台库包含许多不可变的类，包括String、原生类型的包装类、以及`BigInteger`和`BigDecimal`。这样做有很多很好的理由：不可变类比可变类更容易设计、实现和使用。他们意味着更少的出错和更加安全。

创建一个不可变类，要遵循以下5条规则：

1. **不要提供修改对象状态的方法**\(叫做可变器\)。

2. **确保类不能被继承**。这会防止粗心或是恶意的子类修改对象的状态而导致不可变行为发生变化。防止子类化通常通过使类成为终态（final）来完成，但是还有一种替代方法，我们将在后面讨论。

3. **使所有字段都是final类型的**。这种方式可以借助于系统的强制限制来清晰表达你的意图。如果将对一个新创建实例的引用从一个线程传递给另外一个线程，但又没有进行同步，那就需要确保正确的行为了，这在内存模型\[JLS, 17.5; Goetz06, 16\]中有过介绍。

4. **使所有字段都是私有的**。这将阻止客户端访问指向可变对象的字段并且直接修改这些对象。虽然从技术上来说，不可变类所包含的public final字段是可以包含原生值或是对不可变对象的引用的，不过并不推荐这么做，因为在后续的版本中会导致无法修改内部表示（条款15与16）。

5. **确保对任意可变组件的互斥访问**。如果你的类持有任何引用可变对象的字段，请确保该类的客户端无法获得对这些对象的引用。所以，永远不要将这样的字段初始化为客户端提供的对象引用或从访问器返回字段。在构造方法、访问器和`readObject`\(条款88\)方法中使用**防御式拷贝**\(条款50\)。

之前的条款里的许多示例类都是不可变的。第11条中的`PhoneNumber`就是这样的一个类，它为每个属性都提供了访问器，但没有对应的赋值方法。下面是一个稍微复杂一点的例子：

```java
// Immutable complex number class
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                           (im * c.re - re * c.im) / tmp);
    }

    @Override 
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;
        // See page 47 to find out why we use compare instead of ==
        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }

    @Override 
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override 
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

这个类表示复数（包含实部和虚部的数字）。除了标准的对象方法之外，它还为实部和虚部提供了访问器，并提供了四种基本的算术运算：加法，减法，乘法和除法。请注意算术运算如何创建并返回新的`Complex`实例，而不是修改当前实例。这种模式被称为**函数式方法**，因为方法返回了将一个函数应用到其操作数的结果，而非修改它。把它和过程式或命令式方法相比，这些方法将过程应用到操作中，导致他的状态改变。请注意，这里方法的名称是介词（如`plus`）而不是动词（如`add`）。这强调了一个事实，即方法不会改变对象值。`BigInteger`和`BigDecimal`类不遵守这个命名约定，所以导致许多使用错误。

如果你不熟悉它，那么函数式方法可能看起来不自然，但它具有不可变性，这个特性具有许多优点。**不可变对象很简单**。不变对象只会处于一种精确的状态下，即创建时所处的那个状态。如果确信所有构造方法都会建立类的不变性，那就可以确保这些不变性始终都是成立的，你自己和使用这个类的程序员不需要做任何额外的事情。另一方面，可变对象可以具有任意复杂的状态空间。如果文档没有对变化器方法所执行的状态转换进行精确的描述，那就很难或是根本不可能可靠地使用可变类。

**不可变对象天生就是线程安全的;他们不需要同步**。它们不会因多线程并发访问而被损坏。这无疑是实现线程安全的最简单方法。因为任何线程都无法观察到另一个线程对不可变对象的任何影响，因此不可变对象可以自由共享。因此，不可变类应该鼓励客户端尽可能重用现有实例。一种简单的方式是为常用值提供公共静态终态常量。例如，复数类可能提供以下常量：

```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```

这种方法可以进一步改造。不可变类可以提供静态工厂\(条款1\)，这些工厂缓存经常请求的实例，以避免在现有实例可用时创建新实例。所有原生类型的包装类和`BigInteger`都是这样做的。使用这种静态工厂会使得客户端能够共享实例而非创建新的实例，从而减少内存占用和垃圾收集成本。在设计一个新类时，选择静态工厂而非公有构造方法会使得你在后续增加缓存时拥有一定的灵活性，而无需修改客户端。

不可变类可以自由共享这一事实的结果就是你永远都不需要对他们进行防御性拷贝（条款50）。事实上，你永远都不需要对其进行拷贝，因为副本与原件一定是相同的。因此，你不需要，也不应该为不变类提供clone方法或是[拷贝构造方法](https://blog.csdn.net/ab113/article/details/73332096)（条款13）。这一点在早期的Java平台上并未得到充分的理解和认识，所以`String`类提供了一个拷贝构造方法，不过我们不应该使用它（条款6）。

**不变对象不仅可以共享，而且还可以共享其内部实现**。比如说，`BigInteger`类在内部使用了一种符号数值表示。这个符号是由一个int来表示的，而数值则是由一个int数组来表示。`negate`方法会生成一个新的数值相同，但是符号相反的`BigInteger`。它不需要拷贝数组，即便其是可变的；新创建的BigInteger会指向与原来的`BigInteger`相同的内部数组。

**不可变对象是构建其他对象的一个绝佳基础，无论所构建的对象是可变的还是不可变的均如此**。如果你知道构成一个复杂对象的这些对象不会发生变化，那么维护这样一个复杂对象的不变性就会变得异常简单。该原则的一个特别使用场景是将不变对象作为`map`的键或是`set`的元素是非常好的：一旦位于map或是set中，你就不必担心其值的变化了，因为这种变化会破坏`map`和`set`的不变性。。

不变对象天然就能提供失败原子性（条款76）。其状态永远不会发生变化，因此不可能出现临时的不一致性。

**不可变类的主要缺点是每个不同的值都需要一个单独的对象**。创建这些对象的成本可能很高，尤其是如果对象很大的话。比如说，假设你有一个百万位的`BigInteger`，想要修改其低位：

```java
BigInteger moby = ...;
moby = moby.flipBit(0);
```

`flipBit`方法会创建一个新的`BigInteger`实例，该实例也具有百万位的长度，它与原来的`BigInteger`只有一位不同。这个操作消耗的时间和空间跟`BigInteger`的大小成正比。这一点与`java.util.BitSet`不同。与`BigInteger`一样，`BitSet`也表示任意长的一个位序列，不过与`BigInteger`不同的是，`BitSet`是可变的。`BitSet`类提供了一个方法，可以让你在常量时间内修改百万位实例其中的一位：

```java
BitSet moby = ...;
moby.flip(0);
```

如果执行了一个多步操作，每一步都生成一个新的对象，最后除了最终的结果，其他对象都被丢弃掉了，那这就会导致性能问题。那么性能的问题就会被放大出来。这个问题有两个解决方案。首先是猜测哪些多步操作会被共用，然后将其以原子形式提供。如果将一个多步操作以原子形式提供，那么不变类就无需在每个步骤中都创建单独的对象了。在内部，不变类可以自由发挥。比如说，`BigInteger`拥有一个包级别的可变『伴生类』，它用于加速多步操作，比如说模指数运算。使用这个可变伴生类要比`BigInteger`困难多了，原因在之前都已经说过了。幸好，你不用非得使用它：`BigInteger`的实现者已经帮你完成了这些困难的工作。

如果能够精确预测到客户端会在不变类上执行哪些复杂操作，那么包级别的私有可变伴生类方式就会很好用。如果做不到这一点，那就最好提供一个公有的可变伴生类。在Java平台库中，这种方式的一个典型示例就是`String`类，其可变伴生类是`StringBuilder`（以及其已被废弃的前辈`StringBuffer`）。

既然已经知道了如何创建不变类，并且理解了不变的优缺点，下面我们就来讨论几个设计选择。回忆一下，为了确保不变性，类必须要保证自己不能被子类化。这可以通过将类标记为`final`来实现，不过还有另外一种更加灵活的方式。相比于将不变类标记为`final`，你可以将其所有构造方法标记为私有或是包级别私有，并添加公有的静态工厂来替换掉公有构造方法（条款1）。具体一点，下面展示了采用这种方式编写的`Complex`类：

```java
// Immutable class with static factories instead of constructors
public class Complex {
    
	private final double re;
    
	private final double im;
    
	private Complex(double re, double im) {
		this.re = re;
		this.im = im;
	}
    
	public static Complex valueOf(double re, double im) {
		return new Complex(re, im);
	}
... // Remainder unchanged
}
```

通常来讲，这个方法是最好的选择。它最灵活，因为它允许多个包私有的实现类的使用。对于属于包之外的客户端来说，不可变类是有效的终态，因为包外的类是不可能继承一个没有公共或者受保护的构造方法的类的。除了支持多实现类的灵活性之外，这种方法还可以通过改进静态工厂的对象缓存功能，然后在后续版本中调优该类的性能。

当编写`BigInteger`和`BigDecimal`时，不可变类一定是有效的final，所以它们的所有方法都可能被重写，这一点没有得到广泛的理解。遗憾的是，在保留向后兼容性的情况下，这一问题无法在事后得到纠正。如果你编写的类的安全性依赖于来自不受信任的客户端的`BigInteger`或`BigDecimal`参数的不可变性，那么你必须检查该参数是否是『真正的』`BigInteger`或`BigDecimal`实例，而不是一个不受信任的子类的实例。如果是后者，那么在假设它可能是可变的情况下(条款50)，你必须对它进行防御性拷贝：

```java
public static BigInteger safeInstance(BigInteger val) {
    return val.getClass() == BigInteger.class ? val :
    			new BigInteger(val.toByteArray());
}
```

在条款开始部分，不可变类的规则列表里说到没有方法可以修改该对象，而且他的所有字段必须是`final`的。事实上这些规则非常地有必要，它可以轻松提高性能。实际上，没有方法可以在对象的状态上产生额外的可见的变化。然而，有一些不可变类有一个或多个非`final`字段，它们会在第一次需要时，将昂贵的计算结果缓存到这些字段中。如果相同的值被再次请求到，则返回缓存的值，从而节省了重新计算的成本。这个技巧很有效果，因为对象是不可变的，这保证了如果重复计算，计算仍然得到相同的结果。

例如，`PhoneNumber`的`hashCode`方法(条款11，第53页)在第一次调用时计算哈希代码，并缓存它，以防再次调用它。这个技术是一个懒加载的例子(条款83)，`String`也使用这个技术。

有一件事情需要注意，就是序列化。如果你打算使你的不可变类去实现`serializable`接口，而且它包含了一个或者多个引用了可变类对象的字段，那么你必须提供明确的`readObject`方法或者`readResolve`方法，又或者使用`  ObjectOutputStream.writeUnshared`和`ObjectInputStream.readUnshared methods`方法，即便默认的序列形式是可接受的。否则攻击者可能创建该一个类的可变实例。这个主题在条款88中会被详细介绍到。

总结一下，抵制为每个getter编写setter的冲动。**类应该是不可变的，除非有很好的理由让它们可变**。不可变类提供了许多优点，它们唯一的缺点是在某些情况下可能出现性能问题。你应该总是把小的值对象，如`PhoneNumber`和`Complex`，设置为不可变的。（Java平台库中有几个类，比如`java.util.Date`和`java.awt`，本应该都是不可变的，但事实并非如此。）你应该认真地考虑将较大的值对象(如`String`和`BigInteger`)也设置为不可变的。仅仅当你确定获取满意的性能是必须的时候（条款67），你才应该为你的不可变类提供一个公共的可变伴生类。

有些类的不变性是不切实际的。如果不能使类不可变，那么尽可能限制其可变性。减少对象可能存在的状态数量使得对象的推论变得更容易，而且减少了出错的可能。因此，除非有强制的理由设置为非终态的（`nonfinal`），否则尽可能地让每个字段成为终态的（`final`）。将本条款的建议和第15条的结合起来，你自然就倾向于这个观点——**除非有很好的理由不这么做，不然应该声明所有字段为`private final`**。