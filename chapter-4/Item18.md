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

`InstrumentedSet`类称为包装类，因为每个`InstrumentedSet`实例都包含（“包装”）另一个`Set`实例。这也称为『装饰器模式』[Gamma95]，因为`InstrumentedSet`类通过添加仪表盘来“装饰”`Set`集合。有时，组合和转发的结合被宽泛地称为『委托』。严格来说，只有包装器对象可以将自身传递给包装对象，才是委托[Lieber-man86; Gamma95]。

包装类的缺点很少。不过有一个需要警惕的是，包装类不适合在回调框架中使用。在该框架里，对象将自身引用传递给另一个对象，来保证后续调用（“回调”）。因为被包装的对象无法感知到自己的包装器，所以它只能传递一个自身引用（`this`），这样回调就可以避开包装器。这被称作『自我问题』[Lieberman86]。有些人担心转发方法的调用带来的性能影响或包装器对象造成的内存占用影响。但是，实际上，这两个都没产生太大的影响。编写转发方法很麻烦，但是你不得不为每一个接口编写一次可重用的转发类，而且可能还要为你自己提供。举个例子，Guava为所有的集合接口提供了转发类 [Guava]。

只有在子类确实是超类的子类型的情况下，继承才合适。换句话说，只有当两个类之间存在“is-a”关系时，B类才应该继承A类。如果你想让B类继承A类，那么先问问自己：每个B实例都是A类型吗？如果你不能直接回答yes，那么B就不应该继承A。如果答案是否定的，那么通常情况下B类应该包含一个A类的私有实例并公开一个不同的API：A不是B的必要部分，仅仅是B的实现的细节。

在Java平台库中有许多明显违反这一原则的地方。例如，栈不是矢量，因此`  Stack`不应该继承`Vector`。同样地，属性列表不是哈希表，因此`Properties`不应该继承`Hashtable`。在这两种情况下，组合的方式可能会更好。

如果你在适合组合的地方使用继承，那么你没必要暴露实现细节。最后的API会把你和最初的实现绑定一起，永远限制你的类的性能。更严重的是，通过公开内部组件，你让客户端可以直接访问它们。至少，它会导致语义混乱。例如，如果p引用了一个`Properties`实例，那么`p. getproperty (key)`可能会产生与`p.get(key)`不同的结果：前者考虑了默认值，而后者(从`Hashtable`继承而来)则不会。最严重的是，客户端可以通过直接修改父类来破坏子类的不变量。对于`Properties `，设计者希望只允许字符串作为键和值，但是直接访问底层`HashTable`允许违反这个不变量。一旦违反，就不再可能使用`Properties` API的其他部分(加载和存储)。当发现这个问题时，已经太晚了，无法纠正它，因为客户端已经依赖于非字符串的键和值的使用。

在决定使用继承代替组合之前，你还有最后一些问题需要问自己。比如，你想要继承的这个类，他的API中会有一些缺陷吗？如果有，你愿意把这些缺陷传播到你自己类的API里吗？继承会传播父类API中的任意缺陷，然而组合可以让你设计出新的API，并将这些缺陷隐藏掉。

总而言之，继承是强大的，但是它是有问题的，因为它违反了封装。只有子类和父类之间存在真实的父子关系时，才适合使用继承。即使这样，如果子类与父类不在同一个包中，并且父类不是为继承而设计的，那么继承可能导致脆弱性。为了避免这种脆弱性，我们可以使用组合和转发代替继承，尤其是如果一个恰当的实现了包装器的接口存在的话。包装类不仅比子类更健壮，而且更强大。