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

子类之所以会变得脆弱的另外一个原因在于，其父类可能会在后续版本中增加新的方法。假设某个程序的安全性取决于这样一个事实，即插入到某个集合中的所有元素都要满足某个断言。这可以通过子类化这个集合，并重写其每个方法，使之能够在添加元素之前就满足这个断言来确保。这种方式可以工作得很好，但是当能够插入元素的新方法在后续版本中被添加到父类中时就不行了。一旦出现这种情况，那么仅仅通过调用新方法（子类并未对其进行重
写）就可以添加『不合法的』元素了。这不是个单纯的理论上的问题。当改造`Hashtable`与`Vector`以便将其添加到集合框架中时，我们就得修复其所带来的几个安全漏洞。

这些问题源起自重写方法。你可能觉得如果仅仅是添加新的方法并且避免重写既有的方法，那就可以安全地扩展一个类。虽然这种扩展会更加安全一些，不过也不是毫无风险的。如果父类在后续的版本中需要添加新的方法，但是很不幸，子类中已经存在相同签名但是返回类型不同的方法，那么子类将无法编译成功[JLS, 8.4.8.3]。如果子类方法的签名与返回类型同父类新添加的方法完全一致，那实际上就是在进行方法重写，这样就会遭遇到之前所提及的问题。此外，方法能够完全满足父类新方法的契约这件事还是存疑的，因为在编写子类方法时，契约实际上还不存在呢。

幸好，有一种方式可以避免上面提到的所有问题。相比于继承一个既有的类，你还可以为新类添加一个私有字段，使之引用既有类的一个实例。新类中的每个实例方法都调用所包含的既有类的实例所对应的方法，并返回结果。这叫做转发，新类中的方法则叫做转发方法。得到的类是非常稳定的，它不依赖于既有类的实现细节。甚至向既有类添加方法也不会对新类产生任何影响。具体一点，下面是`InstrumentedHashSet`的一个替代类，使用到了组合与转发方式。注意到其实现被划分成了两部分，类本身与可重用的转发类，后者包含了所有的转发方法，除此之外就没有其他的了：

```java
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    public InstrumentedSet(Set<E> s) {
    	super(s);
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

// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    
    public ForwardingSet(Set<E> s) { this.s = s; }
    
    public void clear() { s.clear(); }
    
    public boolean contains(Object o) { return s.contains(o); }
    
    public boolean isEmpty() { return s.isEmpty(); }
    
    public int size() { return s.size(); }
    
    public Iterator<E> iterator() { return s.iterator(); }
    
    public boolean add(E e) { return s.add(e); }
    
    public boolean remove(Object o) { return s.remove(o); }
    
    public boolean containsAll(Collection<?> c)
    { return s.containsAll(c); }
    
    public boolean addAll(Collection<? extends E> c)
    { return s.addAll(c); }
    
    public boolean removeAll(Collection<?> c)
    { return s.removeAll(c); }
    
    public boolean retainAll(Collection<?> c)
    { return s.retainAll(c); }
    
    public Object[] toArray() { return s.toArray(); }
    
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    
    @Override
    public boolean equals(Object o) { return s.equals(o); }
    
    @Override
    public int hashCode() { return s.hashCode(); }
    
    @Override
    public String toString() { return s.toString(); }
}
```

`InstrumentedSet`类的设计是通过`Set`接口来驱动的，它拥有`HashSet`类的功能。除了健壮外，这个设计还非常灵活。`InstrumentedSet`类实现了`Set`接口，并拥有一个参数类型也是`Set`的构造方法。本质上，该类会将一个`Set`转换为另外一个`Set`，并添加了增强（instrumentation）功能。与基于继承的方式不同（只能针对于单个具体类，并且需要为父类每个所支持的构造方法都提供单独的构造方法），包装类可用于增强任何`Set`实现，并且可以与任何预先存在的构造方法搭配使用：

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

`InstrumentedSet`类甚至还可以用于临时增强一个没有增强功能的`Set`实例：

```java
static void walk(Set<Dog> dogs) {
    InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
    ... // Within this method use iDogs instead of dogs
}
```

`InstrumentedSet`类称为包装类，因为每个`InstrumentedSet`实例都包含（“包装”）另一个`Set`实例。这也称为『装饰器模式』[Gamma95]，因为`InstrumentedSet`类通过添加增强功能『装饰』了一个`Set`。有时，组合与转发合起来叫做委托。从技术角度来说，如果包装对象没有将自身传递给被包装的对象，那就不是委托[Lieber- man86; Gamma95]。

包装类没有什么缺点。值得注意的一点就是，包装类不适合在回调框架中使用。在该框架里，对象将自身引用传递给另一个对象，来保证后续调用（“回调”）。由于被包装的对象并不知晓它的包装器，因此它会传递指向自身的引用（this），所以回调是无法得到包装器的。这叫做SELF问题[Lieberman86]。有些人会担心转发方法调用的性能影响，或是包装对象的内存占用。不过在实际情况下，这些都不是什么问题。编写转发方法有些单调乏味，不过你只需要为每个接口编写一次可重用的转发类即可，有时转发类是现成的。比如说，Guava就为所有的集合接口提供了转发类[Guava]。

继承只在子类真的是父类的子类型这种情况下适用。换句话说，类B若要继承类A，那就需要确保这两个类之间存在『is-a』关系。如果你想让B类继承A类，那么先问问自己：每个B实例都是A类型吗？如果你不能直接回答yes，那么B就不应该继承A。如果答案是否，那么通常来说，B应该包含一个私有的A实例并公开一个不同的API：A并非B的一个必要组成部分，仅仅只是其实现细节而已。

在Java平台库中存在不少明显违背该原则的地方。例如，栈不是矢量，因此`  Stack`不应该继承`Vector`。同样地，属性列表不是哈希表，因此`Properties`不应该继承`Hashtable`。在这两种情况下，组合的方式可能会更好。

如果在应当使用组合的情况下使用了继承，那就没必要公开实现细节了。得到的API会将你绑定到原来的实现上，永远限制了类的执行。更为严重的是，由于公开了内部实现，客户端就可以直接访问他们了。至少，这会导致令人困惑的语义问题。比如说，如果p指向了一个`Properties`实例，那么`p.getProperty(key)`就会与`p.get(key)`得到不同的结果：前者考虑到了默认值，而后者则继承自`Hashtable`，并未考虑这个问题。最为严重的是，客户端可能会通过直接修改父类而破坏子类的不变性。对于`Properties`来说，设计者的意图是只有字符串才能作为键与值，不过直接访问底层的`Hashtable`却违背了这种不变性。一旦违背，我们就无法再使用Properties API的其他部分了（load与store）。在发现这个问题时，想要纠正就已经为时过晚了，因为客户端已经使用上了非字符串的键与值了。

在决定使⽤用继承而非组合前，你还应该问自己下面这些问题。你想要继承的类在其API中是否有瑕疵？如果有，那么将这些瑕疵传播到自己的类的API中是否可以？继承会传播父类API中的所有瑕疵，而组合则可以设计全新的API，它可以隐藏掉这些瑕疵。

总结一下，继承很强大，不过它也是存在问题的，因为它违背了封装原则。只有当父类与子类之间存在真正的子类型关系时，继承才是恰当的。甚至这时，如果子类与父类不在同一个包中，而父类在设计时并不不想被继承，那么继承就会导致脆弱性。为了避免这种脆弱性，请使用组合与转发来替代继承，特别是实现包装类的一个恰当的接口存在时更是如此。相比于子类来说，包装类不仅更加健壮，而且更加强大。