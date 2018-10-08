## 组合优先于继承

继承是实现代码重用的一种强大方法，但它并不总是最佳的工具。如果使用不当，会导致软件变得脆弱。在包中使用继承是安全的，其中子类和超类实现由相同的程序员控制。在扩展专门为扩展而设计和记录的类时，使用继承也是安全的（条款19）。我们可以跨包继承一个普通的具体类，但是，这是危险的。提醒一下，本书使用“继承”一词表示的是***实现继承***(当一个类扩展另一个类时)。本条款讨论的问题不适用于***接口继承***(当类实现接口或一个接口继承另一个接口时)。

**与方法调用不一样，继承违反了封装**。换句话说，子类的正确功能依赖于它父类的实现细节。超类的实现可能随着版本的迭代发生变化，如果发生了变化，子类可能会崩溃，即使它的代码没有被修改过。结果，子类必须同它的父类一起发展，除非父类的作者是在一开始就考虑到扩展性下去设计和记录这个父类的。

具体化上面提到的场景，假设我们有一个使用了`HashSet`的程序。为了优化程序的性能，我们需要查询这个`HashSet`，以确定自创建以来一共添加了多少个元素(不要与当前的大小混淆，当元素被删除时，当前的大小会下降)。为了提供这个功能，我们给这个`HashSet`添加一个变量，它记录着试图插入的元素的个数，并且提供一个该变量的访问方法。`HashSet`已经包含了2个添加元素的方法，`add`和`addAll`,因此，我们重载这两个方法就好了：

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

这个类看起来合理，但是它没有作用。假设我们创建了一个实例，并使用`addAll`方法添加三个元素。顺便，我们使用静态工厂方法`List.of`（Java9新加的方法）来创建一个列表。如果你使用的是更早的版本。可以使用`Arrays.asList`代替：

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("Snap", "Crackle", "Pop"));
```

我们希望`getAddCount`方法此时返回3，但它返回的是6。哪里出错了呢？在`HashSet`内部，`addAll`方法是在其`add`方法之上实现的，尽管`HashSet`完全有理由不去记录这个的实现细节。`InstrumentedHashSet`的`addAll`方法给`addCount`加3，然后使用` super.addAll`去调用`HashSet`的`addAll`实现。这个方法反过来调用`add`方法(在`InstrumentedHashSet`中被重写过)，每个元素都会调用一次。这三次调用每一次都往`addCount`上加1，所以总共增加了6个元素：使用`addAll`方法添加的每个元素都会被重复计数。

我们可以通过消除`addAll`方法的覆盖来“修复”子类。虽然生成的类可以工作，但它的正确功能取决于`HashSet`的`addAll`方法是在`add`方法之上实现的事实。这种『自我使用』是一个实现细节，不能保证在Java平台的所有实现中都能保留，并且可能会随着版本的不同而发生变化。因此，最终的`InstrumentedHashSet`类将是脆弱的。

有个做法稍微好点，就是重载`addAll`方法，在方法里去迭代指定的集合，为每个元素调用一次`add`方法。无论`HashSet`的`addAll`方法是否是在其`add`方法之上实现的，这都将保证正确的结果，因为`HashSet`的`addAll`实现将不再被调用。然而这个技术并不能解决我们所有问题。它相当于重新实现超类方法，这可能会导致自使用，也可能不会，这是困难的、耗时的、容易出错的，并且可能会降低性能。而且这种方法不可能一直有用，因为无法访问那些子类无法访问的私有的不可见的字段，所以有些方法无法被实现。

子类脆弱的一个相关原因是他们的超类可能在后续版本中获得新的方法。假设一个程序的安全性取决于插入到某个集合中的所有元素满足某个谓词。这可以通过子类化集合并且覆盖每个能够添加元素的方法来确保在添加元素之前满足谓词。这样就可以很好地工作，即时在后续版本中向超类中添加了能够插入元素的新方法。一旦发生这种情况，只需调用新方法就可以添加“非法”元素，而新方法在子类中不会被覆盖。这不是一个纯粹的理论问题。当`Hashtable`和`Vector`被改造以加入集合框架时，必须修复这一性质的几个安全漏洞。