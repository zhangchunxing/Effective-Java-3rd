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

如果找不到`Bug`也不用感到沮丧。很多专家级程序员也曾经犯过这个错误。问题在于外层集合（suits）迭代器的`next`方法调用次数过多。它应该在外层循环调用，这样每个`suit`就只会被调用一次，不过代码中却在内层循环中调用了，因此每个`card`都会被调用一次。当`suits`遍历完后，循环就会抛出`NoSuchElementException`。

如如果真的很不幸，外层集合的大小是内层集合的倍数（或许因为他们是同一个集合），那么循环就会正常终止，不过结果却不对。比如说，考虑下面这个设计不周的代码，它想打印出一对骰子所有可能的组合数：

```java
// Same bug, different symptom!
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);
for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
	for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
		System.out.println(i.next() + " " + j.next());
```

该程序并不会抛出异常，不过只会打印出6对（从『ONE ONE』到『SIX SIX』），而非期望的36个组合。

要想修复这些示例中的Bug，你需要在外层循环作用域中添加一个变量来持有外部元素：

```java
// Fixed, but ugly - you can do better!
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
	Suit suit = i.next();
	for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
		deck.add(new Card(suit, j.next()));
}
```

如果使用嵌套的`for-each`循环，那么就可以轻而易举地将问题解决掉。代码像期待的那样简洁：

```java
// Preferred idiom for nested iteration on collections and arrays
for (Suit suit : suits)
	for (Rank rank : ranks)
		deck.add(new Card(suit, rank));
```

不过遗憾的是，有3种情况无法使用`for-each`：

-  破坏性过滤——如果想在遍历集合时删除所选元素，那就需要使用显式迭代器，这样才能调用其`remove`方法。可以通过Java 8向`Collection`中所增加的`removeIf`方法来避免显式的遍历。
-  转换——如果在遍历列表或是数组时需要替换掉部分或是全部元素值，那就需要通过列表的迭代器或是数组的索引来进行替换。
-  并行迭代——如果需要并行遍历多个集合，那就需要对迭代器或是索引变量进行显式的控制，这样就可以提前确定好所有的迭代器或是索引变量（这一点在上述的扑克与骰子示例中已经介绍过了）。

如果遇到以上这些情况，请使用普通的for循环并注意一下上面提到的陷阱。

`for-each`循环不仅可以迭代集合与数组，还可以迭代实现了`Iterable`接口的任意对象，对象包含了唯一一个方法。如下代码展示了该接口：

```java
public interface Iterable<E> {
	// Returns an iterator over the elements in this iterable
	Iterator<E> iterator();
}
```

如果要从头编写自己的`Iterator`实现来实现`Iterable`，那还有点麻烦；不过，如果编写了一个类型，它表示一组元素，那就应该考虑让其实现`Iterable`，即便它没有实现`Collection`亦如此。这样，用户就可以通过`for-each`循环来迭代这个类型了，而且用起来会非常不错。

总结一下，相比于传统的`for`循环来说，`for-each`在简洁性、灵活性与避免`Bug`上提供了强大的优势，而且又没有性能上的损失。请在可能的情况下优先使用`for-each`循环而非`for`循环。

