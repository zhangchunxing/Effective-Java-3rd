#  为子类设计接口

在Java 8之前，在不破坏现有实现的情况下向接口添加方法是不可能的。如果你向接口添加新方法，那么现有的实现通常会缺少该方法，然后导致一个编译期异常。在Java 8中，添加了**默认方法**的概念[JLS 9.4]，目的是允许向现有接口添加额外的方法。但是向现有接口添加新方法充满了风险。

默认方法的声明包含一个默认实现，所有实现该接口但没有实现默认方法的类都可以使用这个默认实现。虽然在Java中添加默认方法使向现有接口添加方法成为可能，但不能保证这些方法在所有现有实现中都能很好地工作。默认方法被“注入”到现有的实现中，而无需实现者的知情或同意。在Java 8之前，编写这些实现时都默认它们的接口永远不会获得任何新方法。

Java 8的核心集合接口中添加了许多新的默认方法，主要是为了方便lambdas的使用(第6章)。Java库的默认方法是高质量的通用实现，在大多数情况下，它们工作得很好。但是，**编写一个默认方法来维护所有可能实现的不变量是不可能的**。

例如，考虑一下在Java 8中`Collection`接口新加的方法`removeIf`。这个方法会移除所有可以使给定的boolean函数返回`true`的元素。这个方法的默认实现是：通过自身的迭代器遍历集合时，在每个元素上调用这个谓词，并且使用迭代器的`remove`方法去移除这个谓词返回`true`的元素。实现大概看起来就像这样子：

```java
// Default method added to the Collection interface in Java 8
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext(); ) {
        if (filter.test(it.next())) {
            it.remove();
            result = true;
        }
    }
    return result;
}
```

这是为`removeIf`方法编写的最好的通用实现，但遗憾的是，它在一些实际的集合实现上失败了。比如说，考虑一下`org.apache.commons.collections4.collection.SynchronizedCollection `。这个类来自于 **Apache Commons**库，它跟`java.util`包下的`Collections.synchronizedCollection `静态工厂返回的结果很相似。Apache版本的库额外提供了使用一个客户端提供的对象来做锁的功能，以代替集合。换一句话说，它是一个包装器类（条款18），它所有的方法在委托给被包装的对象之前，都会依靠一个锁对象来做同步的。

Apache的`SynchronizedCollection`类仍然被积极维护着，但是在撰写本文时，它还没有覆盖`removeIf`方法。如果这个类与Java 8一起使用，那么它将继承`removeIf`的默认实现，而`removeIf`并没有(实际上也不能)维护这个类的基本承诺：自动同步每个方法调用。默认实现并不知道『同步』这件事，也无法访问包含锁定对象的字段。如果客户端在另一个线程对集合进行并发修改的情况下调用了`SynchronizedCollection`实例上的`removeIf`方法，可能会导致`ConcurrentModificationException`或其它未知行为发生。

为了阻止这种事情在类似的Java平台库实现中发生，比如`Collections.synchronizedCollection`返回的包私有类，JDK维护者们不得不重写默认的`removeIf`方法实现和其他类似的方法，以便在调用默认实现之前执行必要的同步。之前的那些不是Java平台的集合实现没有机会随着接口的改变同步做出类似的变动，并且有些实现还没这样做。

**默认方法的出现，当前存在的接口实现类可能编译的时候不会报错或警告，但是运行时可能失败**。这个问题虽然不是很常见，但也不是孤立的事件。在Java 8中一些被添加到集合接口中的方法是易受影响的，而且会影响一些现有的实现。