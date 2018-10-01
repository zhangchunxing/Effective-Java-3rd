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

