## 组合优先于继承

继承是实现代码重用的一种强大方法，但它并不总是最佳的工具。如果使用不当，会导致软件变得脆弱。在一个包内使用继承是安全的，其中子类与父类实现都处于相同的程序员的掌控之下。当扩展的类是经过专门设计，并且拥有良好的文档，目的就是在于扩展时，那么使用继承也同样是安全的（条款19）。我们可以跨包继承一个普通的具体类，但是，这是危险的。提醒一下，本书使用术语『继承』来表示**实现继承**（表示一个类继承了另一个类）。本条款所讨论的问题并不适用于**接口继承**（即一个类实现了一个接口或是一个接口继承了另外一个接口）。

**与方法调用不同，继承违背了封装原则[Snyder86]**。换句话说，子类要依赖于它的父类的实现细节才能完成恰当的功能。父类的实现可能随着版本的迭代发生变化，如果发生了变化，子类可能会崩溃，即使它的代码没有被修改过。结果，子类必须同它的父类一起演化，除非父类的作者专门为扩展的目的进行了专门的设计并撰写了相应的文档。

具体一点，假设我们有一个程序使用到了`HashSet`。为了优化程序的性能，我们需要查询这个`HashSet`，以确定自创建以来一共添加了多少个元素(不要与当前的大小混淆，当元素被删除时，当前的大小会减少)。为了提供这个功能，我们编写了一个`HashSet`的变种，它会将要插入的元素的数量记录下来并为这个数量公开一个访问器。`HashSet`已经包含了2个添加元素的方法，`add`和`addAll`，因此，我们重载这两个方法就好了：

```java
// Broken - Inappropriate use of inheritance!
public class InstrumentedHashSet<E> extends HashSet<E> {
	// The number of attempted element insertions
	private int addCount = 0;
    
	public InstrumentedHashSet() {
	}
    
	public InstrumentedHashSet(int initCap, float loadFactor) {
		super(initCap, loadFactor);
	}
    
    @Override
    public boolean add(E e) {
		addCount++;
		return super.add(e);
	}
    
	@Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
    	return addCount;
    }
}
```

这个类看起来合理，不过却无法正常使用。假设我们创建了一个实例，并使用`addAll`方法添加3个元素。碰巧，我们使用静态工厂方法`List.of`（Java9新加的方法）来创建一个列表。如果你使用的是更早的版本。可以使用`Arrays.asList`代替：

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("Snap", "Crackle", "Pop"));
```

我们希望`getAddCount`方法此时返回3，但它返回的是6。哪里出错了呢？在`HashSet`内部，`addAll`方法是在其`add`方法之上实现的，尽管`HashSet`完全有理由不去记录这个的实现细节。`InstrumentedHashSet`的`addAll`方法给`addCount`加3，然后使用` super.addAll`去调用`HashSet`的`addAll`实现。这个方法反过来调用`add`方法(在`InstrumentedHashSet`中被重写过)，每个元素都会调用一次。这三次调用每一次都往`addCount`上加1，所以总共增加了6个元素：使用`addAll`方法添加的每个元素都会被重复计数。

我们可以通过消除对addAll方法的重写来『修复』子类。虽然得到的类可以正常使用，不过它却依赖于这样一个事实：HashSet的addAll方法实现是基于其add方法的。这种『自我使用』是个实现细节，并无法保证在Java平台的所有实现中都如此，而且可能会在版本变更中发生变化。因此，得到的`InstrumentedHashSet`类是非常脆弱的。

有个做法稍微好点，就是重载`addAll`方法，在方法里去迭代指定的集合，为每个元素调用一次`add`方法。无论`HashSet`的`addAll`方法是否是在其`add`方法之上实现的，这都将保证正确的结果，因为`HashSet`的`addAll`的实现将不再被调用。然而这个技术并不能解决我们所有问题。它相当于重新实现父类方法，无论方法是否自我使用了，这么做是困难的、耗时的、容易出错的，并且可能会降低性能。此外，这么做也并非总是可行的，因为当无法访问到私有字段（这些字段无法访问到父类）时，一些方法是无法实现的。

子类脆弱的一个相关原因是他们的超类可能在后续版本中获得新的方法。假设一个程序的安全性取决于插入到某个集合中的所有元素满足某个谓词。这可以通过子类化集合并且覆盖每个能够添加元素的方法来确保在添加元素之前满足谓词。这样就可以很好地工作，即时在后续版本中向超类中添加了能够插入元素的新方法。一旦发生这种情况，只需调用新方法就可以添加“非法”元素，而新方法在子类中不会被覆盖。这不是一个纯粹的理论问题。当`Hashtable`和`Vector`被改造以加入集合框架时，必须修复这一性质的几个安全漏洞。

这两个问题都源于重写方法。