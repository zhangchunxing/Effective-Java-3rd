## 使用依赖注入的方式使用资源，而不要硬编码

许多情况下，我们的类依赖一个或多个已声明的资源。比如，拼写检查器要依赖字典。像这样的类以静态的辅助类的形式来实现是很常见的（第4条）。

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