## 减少可变性

不可变类就是一个实例不能被修改的类。每个实例中包含的所有信息在对象的生命周期内都是固定的，因此永远不会观察到任何更改。Java平台库包含许多不可变的类，包括String、原生类型的包装类、如：`BigInteger`和`BigDecimal`。这样做有很多很好的理由：不可变类比可变类更容易设计、实现和使用。他们意味着更少的出错和更加安全。

创建一个不可变类，要遵循以下5条规则：

1. **不要提供修改对象状态的方法**(称为mutators)。

2. **确保类不能被继承**。这可以防止粗心的或恶意的子类通过可能使对象状态发生变化的行为来破坏类的不可变行为。防止子类化通常通过使类成为终态（final）来完成，但是还有一种替代方法，我们将在后面讨论。

3. **使所有字段都是final类型的**。这个清楚地表达了你的意图，以系统强制的方式。如果一个指向新创建的实例的引用在没有同步操作的情况下从一个线程传递到另一个线程，那么必须确保行为的不出错，正如内存模型那样[JLS, 17.5;Goetz06,16]。

4. **使所有字段都是私有的**。这将阻止客户端访问指向可变对象的字段并且直接修改这些对象。虽然技术上允许不可变类拥有包含了原生值或者不可变对象的引用的的公共的终态的（public final）字段。但不建议这样做，因为它排除了在以后的版本中更改内部表示的可能性(项目15和16)。

5. **确保对任意可变组件的互斥访问**。如果你的类持有任何引用可变对象的字段，请确保该类的客户端无法获得对这些对象的引用。所以，永远不要将这样的字段初始化为客户端提供的对象引用或从访问器返回字段。在构造方法、访问器和`readObject`(条款88)方法中使用**防御性复制**(条款50)。

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

