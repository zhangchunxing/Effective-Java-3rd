## 使用依赖注入的方式使用资源，而不要硬编码

许多情况下，我们的类依赖一个或多个底层的资源。比如，拼写检查器要依赖字典。像这样的类以静态的辅助类的形式来实现是很常见的（第4条）。

```java
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {} // Noninstantiable
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

同样地，将他们以单例的形式来实现也很常见。

```java
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

这两种方法都不令人满意，因为它们假定只有一个字典值得使用。实际上，每种语言都有自己的字典，特殊的字典用于特殊的词汇。另外，使用专门的字典进行测试也是可取的。想当然地认为一本字典就足够了，这是一厢情愿的想法。

你可以尝试让拼写检查器支持多个字典，方法是使dictionary字段成为非final类型，并在现有的拼写检查器中添加一个方法来更改dictionary的引用，但这在并发环境中是笨拙的、容易出错的和不可用的。静态辅助类和单例类不适合这种类，因为他们的行为被已声明的资源参数化了。

我们所需要的是支持类的多个实例的能力(在我们的例子中，SpellChecker)，每个实例都使用客户机所希望的资源(在我们的例子中，是字典)。满足此需求的一个简单模式是在**创建新实例时将资源传递给它的构造函数**。这是依赖注入的一种形式:字典是法术的依赖当它被创建时被注入到拼写检查器中。

```java
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

依赖注入模式如此简单，以至于许多程序员使用了多年，却不知道它的名字。 虽然我们的拼写检查的示例只有一个资源(字典)，但是依赖项注入可以处理任意数量的资源和任意依赖图。它保持了不可变性(第17项)，因此多个客户机可以共享依赖对象(假设客户机需要相同的底层资源)。依赖项注入同样适用于构造函数、静态工厂(第1项)和构建器(第2项)。

这个模式的有一个有用的变换是将资源工厂传递给构造函数。工厂是一个对象，可以反复调用它来创建类型的实例。这些工厂体现了一种设计模式，即工厂方法模式。Java 8中引入的Supplier<T>接口非常适合表示工厂。入参是Supplier<T>类型的方法会强迫工厂的类型参数使用有界通配符类型(第31条)。例如，这里有一种方法，使用客户提供的工厂来生产每一块砖:

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

尽管依赖注入极大地提高了灵活性和可测试性，但它可能会使大型项目变得混乱，而大型项目通常包含数千个依赖项。如果我们使用依赖注入框架(如Dagger [Dagger]、Guice [Guice]或Spring [Spring])，几乎可以消除这种混乱。这些框架的使用超出了本书的范围，但是请注意，为手动依赖注入设计的api通常是为这些框架所使用的。

总之，不要使用单例或静态辅助类来实现一个类，该类依赖于一个或多个底层资源，这些资源的行为会影响类的行为，并且不让类直接创建这些资源。替代的方法是，将该资源或生产这个资源的工厂传递给构造方法(或者是静态工厂或者是构建器)。这种称为依赖注入的实践将极大地增强类的灵活性、可重用性和可测试性。