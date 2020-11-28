# 抛出与抽象相匹配的异常

当一个方法抛出异常时，如果该异常与这个方法所执行的任务之间没有明显的关系时，这就会使人感到困惑。这种情况常常发生在方法传播底层抽象所抛出的异常的时候。这不仅使人困惑，还会污染高层`API`的实现细节。如果高层的实现在后续发布中发⽣了变化，那么它所抛出的异常也要随之发⽣变化，这会破坏既有的客户端程序。

为了避免这个问题，高层应该就地捕获底层异常，然后根据高层抽象再将可解释的异常抛出去。这种做法叫做异常转换：

```java
// Exception Translation
try {
	... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException e) {
	throw new HigherLevelException(...);
}
```

如下是个来自于`AbstractSequentialList`类的异常转换示例，它是`List`接⼝的一个⻣架实现 (条款20)。在该示例中，异常转换是在`List<E>`接⼝的`get`⽅法中进⾏的：

```java
/**
	* Returns the element at the specified position in this list.
  * @throws IndexOutOfBoundsException if the index is out of range 
  * ({@code index < 0 || index >= size()}).
	*/
public E get(int index) {
	ListIterator<E> i = listIterator(index); 
  try {
		return i.next();
	} catch (NoSuchElementException e) {
    throw new IndexOutOfBoundsException("Index: " + index);
  }
}
```

当底层异常有助于调试导致高层异常的问题时，我们会使用一种叫做异常链的特殊形式的异常转换。底层异常（原因）会传递给高层异常，它提供了访问器方法（`Throwable`的` getCause`方法）来获取底层异常：

```java
// Exception Chaining
try {
	... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) {
  throw new HigherLevelException(cause);
}

```

高层异常的构造方法把`cause`（原因）传递给链接的父类构造方法，这样它就会最终传递给`Throwable`的一个构造方法里，比如：`Throwable(Throwable)`：

```java
// Exception with chaining-aware constructor
class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

大多数标准异常都有链接的构造方法。对于没有的异常，你可以通过`Throwable`的`initCause`方法来设置原因。不仅可以通过异常链来以编程的⽅式访问异常原因（借助于 `getCause`），还可以将异常堆栈集成到⾼层异常中。

**虽然异常转换要⽐直接传播底层异常好很多，但也不要过度使⽤**。在可能的情况下，处理来自于底层的异常的最好方式是避免出现异常，这是通过确保底层方法要执⾏成功来达成的。 有时，可以通过先检查⾼层⽅法参数的有效性后再将其传递给底层⽅法来做到。

如果⽆法避免底层出现异常，那么退⽽求其次的⽅法就是让⾼层静默处理掉这些异常，将⾼层⽅法的调⽤者与底层问题隔离开来。在这些情况下，使⽤诸如`java.util.logging`之类恰当的日志基础设施来记录下异常是⽐较好的做法。这样，程序员就可以探究问题，同时⼜又会将客户端代码与用户将其隔离开来。

总结一下，如果防止或是处理来自于底层的异常不是那么方便的话，那就请使用异常转换， 除非底层异常碰巧可以保证其异常也刚好适合于高层。链接方式提供了了最好的解决方案：它可以让我们抛出恰当的高层异常，同时还可以捕获底层原因来进行失败分析（条款75）。

