# 抛出与抽象相匹配的异常

当一个方法抛出一个与它执行的任务没有明显关系的异常时，这会让人困惑。当一个方法传播一个由更底层的抽象抛出的异常时，通常就会发生这种情况。它不仅使人困惑，而且还会污染较上层的`API`实现细节。如果上层的实现在以后的版本中发生变化，那么它抛出的异常也会发生变化，这可能会破坏现有的客户端程序。

为了避免这个问题，上层应该捕获底层异常，然后抛出上层抽象术语可以解释的异常。这种做法叫做异常转换：

```java
// Exception Translation
try {
	... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException e) {
	throw new HigherLevelException(...);
}
```

下面是取自`AbstractSequentialList`类的异常转换示例，该类是List接口的骨架实现（条款20）。在这个例子中，异常转换是由`List<E>`接口中的`get`方法规范强制要求的：

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

当底层异常有助于我们去调试上层异常时，人们会使用一种特殊形式的异常转换——异常链。底层异常传递给上层异常，上层异常提供了访问底层异常的方法（`Throwable’s getCause method`）：

```java
// Exception Chaining
try {
	... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) {
  throw new HigherLevelException(cause);
}

```

上层异常的构造方法把`cause`（原因）传递给可链接的超类构造方法，因此它最终会被传递到`Throwable`的可链接的构造方法里，比如：`Throwable(Throwable)`：

```java
// Exception with chaining-aware constructor
class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

大多数标准异常都有可链接的构造方法。对于没有这种构造方法的异常，你可以使用`Throwable`的`initCause`方法来设置异常原因。可链接的异常不仅可以让你以编程的方式访问异常原因，通过`getCause`。还可以将异常的堆栈信息集成到上层异常中。

**即便异常转换优于从下层随意传播异常，但它不应该被过度使用**。在可能的情况下，解决下层异常的最好方法是，通过确保下层方法的成功来避免异常发生。比如，在上层方法的参数传递给下层时，你可以先校验该参数的有效性。

