## 减少可变性

不可变类就是一个实例不能被修改的类。每个实例中包含的所有信息在对象的生命周期内都是固定的，因此永远不会观察到任何更改。Java平台库包含许多不可变的类，包括String、原生类型的包装类、以及`BigInteger`和`BigDecimal`。这样做有很多很好的理由：不可变类比可变类更容易设计、实现和使用。他们意味着更少的出错和更加安全。

创建一个不可变类，要遵循以下5条规则：

1. **不要提供修改对象状态的方法**(叫做可变器)。

2. **确保类不能被继承**。这会防止粗心或是恶意的子类修改对象的状态而导致不可变行为发生变化。防止子类化通常通过使类成为终态（final）来完成，但是还有一种替代方法，我们将在后面讨论。

3. **使所有字段都是final类型的**。这种方式可以借助于系统的强制限制来清晰表达你的意图。如果将对一个新创建实例的引用从一个线程传递给另外一个线程，但又没有进行同步，那就需要确保正确的行为了，这在内存模型[JLS, 17.5; Goetz06, 16]中有过介绍。

4. **使所有字段都是私有的**。这将阻止客户端访问指向可变对象的字段并且直接修改这些对象。虽然从技术上来说，不可变类所包含的public final字段是可以包含原生值或是对不可变对象的引用的，不过并不推荐这么做，因为在后续的版本中会导致无法修改内部表示（条款15与16）。

5. **确保对任意可变组件的互斥访问**。如果你的类持有任何引用可变对象的字段，请确保该类的客户端无法获得对这些对象的引用。所以，永远不要将这样的字段初始化为客户端提供的对象引用或从访问器返回字段。在构造方法、访问器和`readObject`(条款88)方法中使用**防御式拷贝**(条款50)。

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

这种方法可以进一步改造。不可变类可以提供静态工厂(条款1)，这些工厂缓存经常请求的实例，以避免在现有实例可用时创建新实例。所有原生类型的包装类和`Biginteger`都是这样做的。使用这种静态工厂会使得客户端能够共享实例而非创建新的实例，从而减少内存占用和垃圾收集成本。在设计一个新类时，选择静态工厂而非公有构造方法会使得你在后续增加缓存时拥有一定的灵活性，而无需修改客户端。

不可变类可以自由共享这一事实的结果就是你不需要对他们进行防御性拷贝（条款50）。事实上，你根本不需要做任何拷贝，因为副本永远等价于原件。所以，你不需要也不应该在不可变类上提供克隆方法或复制构造函数(条款13)。在Java平台早期的时候，我们没能很好地去理解上面的话，所以`String`类才会有一个复制构造方法，但是它应该很少被使用（条款6），如果有需要。

**你不仅可以共享不可变对象，而且可以共享它们的内部**。例如，`BigInteger`类在内部使用一对【符号-大小】表示。符号由int表示，大小由int数组表示。`negate`方法会产生一个大小一样并且符号相反的新`BigInteger`对象。即使数组是可变的，它也不需要复制；因为新创建的`BigInteger`对象指向跟源对象相同的内部数组。