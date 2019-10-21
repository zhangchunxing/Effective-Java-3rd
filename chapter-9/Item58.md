# 优先选择for-each循环而非传统的for循环

正如我们在条款45中所介绍的，一些任务最好通过流来完成，而其他一些则需要使用迭代器。如下代码展示了如何通过传统的`for`循环来遍历一个集合：

```java
// Not the best way to iterate over a collection!
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
	Element e = i.next();
	... // Do something with e
}
```

如下代码展示了如何通过传统的`for`循环来遍历数组：

```java
// Not the best way to iterate over an array!
for (int i = 0; i < a.length; i++) {
	... // Do something with a[i]
}
```

这些惯用法要比`while`循环好（条款57），不过也并非完美。迭代器与索引变量都只会增加混乱——你所需要的是元素。此外，他们还增加了出错的可能性。在每个循环中，迭代器会出现3次，索引变量会出现4次，这使得用错变量的可能性大大增加。如果用错了，那么没人可以保证编译器能够捕捉到错误。最后，这两个循环的差别非常大，使得人们将不必要的注意力放到了容器类型上，而且当改变类型时还会引起不必要的麻烦。

`for-each`循环（官方说法叫做『增强的for语句』）可以解决所有这些问题。它通过隐藏迭代器或是索引变量去除掉了混乱以及出错的可能性。其惯用法可用于集合与数组，简化了从一种容器类型实现切换至另外一种容器类型实现的过程：

```java
// The preferred idiom for iterating over collections and arrays
for (Element e : elements) {
	... // Do something with e
}
```

当看到冒号时（:），请将其读作『对…中』。这样，上述循环就可以读作『对elements中的每个元素』。`for-each`循环并没有性能上的损失，即便对于数组也是如此：他们所生成的代码本质上等价于手工所编写的代码。

当涉及到嵌套迭代时，相比于传统的`for`循环来说，`for-each`循环的优势就更为明显了。下面是在执行嵌套迭代时，人们所犯的一个常见错误：

```java
// Can you spot the bug?
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, 
           NINE, TEN, JACK, QUEEN, KING }
...
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
	for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
		deck.add(new Card(i.next(), j.next()));
```

如果你没有发现`bug`不要难过。许多专业程序员都曾犯过这样或那样的错误。问题就是，对于外部集合`suits`，`next`方法在迭代器上调用了多次。它应该在外部循环中调用，这样每个`suit`，`next`方法只会调用一次。但是，它现在是在内部循环中调用，所以每个`cad`，都会被调用一次。当`suits`集合用完后，循环抛出`NoSuchElementException`。

如果你真地不幸运，外部集合的大小是内部集合大小的倍数——可能因为他们是相同的集合——循环会正常地终止，但是它不会执行你想要的操作。举个例子，考虑一下打印一对骰子所有可能的掷骰结果的错误尝试:	

```java
// Same bug, different symptom!
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);
for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
	for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
		System.out.println(i.next() + " " + j.next());
```

程序不会抛出异常，但它只打印6个双数（从“`ONE ONE`”到“`Six Six`”），而不是预期的36个组合。

要修复这些例子中的错误，你必须在外层循环的范围内添加一个变量来保存外层元素：

```java
// Fixed, but ugly - you can do better!
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
	Suit suit = i.next();
	for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
		deck.add(new Card(suit, j.next()));
}
```

如果你使用嵌套的`for-each`循环，这个问题简单地就消失了。最后的代码如你所希望的那样简洁：

```java
// Preferred idiom for nested iteration on collections and arrays
for (Suit suit : suits)
	for (Rank rank : ranks)
		deck.add(new Card(suit, rank));
```

不幸的是，有三种常见的情况你不能使用`for-each`：

- 破坏性的过滤——如果你需要遍历一个集合并删除选定的元素，那么你需要使用一个显式的迭代器，以便你可以调用它的`remove`方法。通过使用`Collection`的`removeIf`方法（Java8添加的），通常你可以避免显式地遍历。

- 转换——如果你需要遍历一个列表或数组，并替换其中部分或全部元素的值，那么你需要使用列表的迭代器或数组的索引来替换元素的值。

- 并行迭代——如果你需要并行地遍历多个集合，那么你需要显式地控制迭代器或索引变量，这样所有的迭代器或索引变量都可以同步地进行（正如上面的错误卡片和骰子示例中无意中演示的那样）。

如果你发现自己处于这些情况中的任何一种，请使用普通的`for`循环，并小心本条款中提到的陷阱。

`for-each`循环不仅允许你遍历集合和数组，还允许你遍历任何一个实现`Iterable`接口的对象，该接口由单个方法组成。该接口如下：

```java
public interface Iterable<E> {
	// Returns an iterator over the elements in this iterable
	Iterator<E> iterator();
}
```

如果你必须从头开始编写自己的迭代器实现，那么实现`Iterable`就有点棘手了。但是，如果你正在写一个代表一组元素的类型，那么你应该好好考虑让它实现`Iterable`，即便你不打算让它实现`Collection`。这将允许你的用户使用`for-each`循环来遍历你的类型，他们会一直很愉快。

总之，与传统的`for`循环相比，`for-each`循环在清晰度、灵活性和`bug`预防方面提供了引人注目的优势，并且没有性能损失。所以，在任何情况下，都应该尽量使用`for-each`循环而不是`for`循环。