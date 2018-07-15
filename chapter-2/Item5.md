## 优先使用依赖注入而非硬编码资源的关联关系

很多类都会依赖于一个或多个底层资源。比如说，拼写检查器会依赖于字典。我们常常会看
到这种类被实现为了静态辅助类（条款4）：

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

这两种方法都不令人满意，因为它们假定只有一个字典值得使用。实际上，每种语言都有自己的字典，特殊的字典用于特殊的词汇表。此外，我们还需要一个特殊的字典用于测试。想当然地认为一本字典就足够了，这是一厢情愿的想法。

你可以尝试让拼写检查器支持多个字典，方法是使dictionary字段成为非final类型，并在现有的拼写检查器中添加一个方法来更改dictionary的引用，不过这么做有些笨拙、易出错，并且在并发设置下无法正常工作。**如果一个类的行为是通过底层资源来参数化的，那么静态辅助类与单例就不适合这种情况**。

我们所需要的是支持类的多个实例的能力(在我们的例子中，SpellChecker)，每个实例都使用客户机所希望的资源(在我们的例子中，是字典)。满足此需求的一个简单模式是在**创建新实例时将资源传递给它的构造函数**。这是依赖注入的一种形式：字典是拼写检查器器的依赖，在创建拼写检查器时会将字典注入到其中 。

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

这个模式的有一个有用的变换是将资源工厂传递给构造函数。工厂是一个对象，可以反复调用它来创建同一类型的实例。这些工厂体现了一种设计模式，即工厂方法模式。Java 8中引入的Supplier<T>接口非常适合表示工厂。将Supplier<T>作为输入的方法会通过绑定的通配符类型（条款31）来限制工厂的类型参数。比如说，如下方法会通过客户端提供的用于生成每个瓷砖的工厂来创建一个马赛克：

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

尽管依赖注入极大地提高了灵活性和可测试性，但它可能会使大型项目变得混乱，而大型项目通常包含数千个依赖项。如果我们使用依赖注入框架(如Dagger [Dagger]、Guice [Guice]或Spring [Spring])，几乎可以消除这种混乱。这些框架的使用介绍超出了本书的范围，不过请注意，针对手工进行依赖管理所设计的APIs也是适合于这些框架的 。

总结一下，如果一个类依赖于一个或多个底层资源，而这些资源的行为会影响到类的行为，那么请不要使用单例或是静态辅助类来实现，也不要让类直接创建这些资源 。替代的方法是，将该资源或生产这个资源的工厂传递给构造方法(或者是静态工厂或者是构建器)。这种实践叫做依赖注入，它会极大增强类的灵活性、重用性与可测试性。