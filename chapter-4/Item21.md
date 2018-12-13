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

