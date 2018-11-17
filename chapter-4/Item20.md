# 优先选择接口而不是抽象类

针对于一个类型的多种实现这一问题，Java提供了两种机制，分别是接口与抽象类。由于Java 8中引入了接口的默认方法[JLS 9.4.3]，因此这两种机制都可以为实例方法提供实现。一个主要的差别在于，要想实现由抽象类所定义的类型，这个类必须是抽象类的子类。由于Java只允许单继承，因此这种对抽象类的限制严重违背了其作为类型定义的使用。因为Java只允许单继承，所以这种对抽象类的限制严重限制了它们作为类型定义的使用。无论什么类，只要它定义了所有必须的方法并且遵循通用契约，那么它就可以实现接口，无论这个类位于类层次体系中的哪个地方均如此。

**我们可以轻松改造既有类，使之实现新的接口。**你所要做的只不过是添加必要的方法（如果这些方法尚不存在），并在类声明处添加一个`implements`从句即可。比如说，很多既有类在添加到平台后，都被改造以实现`Comparable`、`Iterable`与`Autocloseable`接口。一般来说，既有类无法被改造以继承新的抽象类。如果想让两个类继承同一个抽象类，那么你就需要将这个抽象类放置到类层次较高的位置上，使之成为这两个类的祖先。但遗憾的是，这会对类型体系造成严重的破坏作用，需要强制要求新抽象类的所有后代都要继承它，无论是否恰当均如此。

**接口是定义混合类型（`mixins`）的一种理想方式**。大致来说，混合类型指的是这样一种类型：一个类除了其『主要类型』外，还可以实现该类型，表示会提供一些可选的行为。比如说，`Comparable`就是个混合类型接口，可以让一个类声明其实例与其他相互可比较的对象之间是有序的。这种接口之所以叫做混合类型是因为它将可选功能『混合』到了类型的主功能中。抽象类不能用作定义混合类型，原因与之前一样，即他们无法被改造进入到既有的类中：一个类不能有一个以上的双亲，而且在类层次中也并没有恰当的位置来插入混合类型。

**接口允许构造非层次化类型的框架**。类型层次对于组织一些东西来说是非常棒的，不过其他一些东西却无法恰到好处地融入到严格的层次体系中。比如，假设我们有一个代表歌手的接口和另一个代表词曲作者的接口：

```java
public interface Singer {
	AudioClip sing(Song s);
}

public interface Songwriter {
	Song compose(int chartPosition);
}
```

在现实生活中，一些歌手也是词曲作者。因为我们使用接口而不是抽象类来定义这些类型，所以完全允许单个类同时实现`Singer`和`Songwriter`。事实上，我们可以定义第3个接口，同时继承`Singer`和`Songwriter`接口，并添加适合于该组合的新⽅方法：

```java
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}
```

你并不总是需要这种级别的灵活性，但是当你需要时，接口就是救命稻草。另外一种实现方式则是使用一个膨胀的类层次体系，该体系为每个受⽀支持的属性组合都包含单独的类。如果在类型系统中有n个属性，那就需要支持`2^n `个可能的组合。这就是所谓的组合爆炸。膨胀的类层次体系会导致很多类方法仅仅在参数类型上不同，因为类层次体系中没有类型能够捕获到常见的行为。

**接口通过包装类的做法实现了安全、强大的功能增强（条款18）**。如果你使用抽象类来定义类型，那么你将让希望添加功能的程序员除了继承之外别无选择。这样，相比于包装类来说，所得到的类功能更差且更加脆弱。

当一个接口方法相对于其他接口方法而言有明显的实现时，请考虑以默认方法的形式提供一个实现，以此来帮助程序员。比如说，请参阅第104页的`removeIf`方法。如果提供了默认方法，请务必通过`@implSpec Javadoc`标记将其标记为可继承（条款19）。

借助于默认方法所提供的实现辅助是有限制要求的。虽然很多接口指定了Object方法如`equals`和`hashCode`的行为，不过你不能为其提供默认方法。此外，接口不允许包含实例段或是非公有静态成员（除了private static方法）。最后，你不能向自己没有控制权的接口添加默认方法。

不过，你可以通过提供一个抽象的骨架实现类与接口来组合接口与抽象类的优势。接口定义一个类型，可能提供了一些默认方法，而骨架实现类则在主要的接口方法基础上实现了其余非主要的接口方法。继承骨架实现就可以完成接口实现的大部分工作。这叫做模板方法模式[Gamma95]。

按照惯例，骨架实现类被称为抽象接口，其中接口是它们实现的接口的名称。比如说，集合框架提供了一个骨架实现处理每一个主要的集合接口：`AbstractCollection `, `AbstractSet` ,  `AbstractList` ,和` AbstractMap` 。可以说，将它们称为`SkeletalCollection`、`SkeletalSet`、`SkeletalList`和`SkeletalMap`是有意义的，但抽象的约定现在已经牢固确立了。如果设计得当，框架实现(无论是单独的抽象类，还是仅仅由接口上的默认方法组成)可以使程序员非常容易地提供他们自己的接口实现。例如，这里有一个静态工厂方法，它在`AbstractList`上包含一个完整的、功能强大的列表实现：

```java
// Concrete implementation built atop skeletal implementation
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    // The diamond operator is only legal here in Java 9 and later
    // If you're using an earlier release, specify <Integer>
    return new AbstractList<>() {
        @Override
        public Integer get(int i) {
        	return a[i]; // Autoboxing (Item 6)
        }
        
        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val; // Auto-unboxing
            return oldVal; // Autoboxing
        }
        
        @Override
        public int size() {
        	return a.length;
        }
    };
}
```

