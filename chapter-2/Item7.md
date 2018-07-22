## 消除废弃的对象引用

如果您从使用手动管理内存的语言(如C或c++)切换到使用垃圾收集语言(如Java)，那么您作为程序员的工作就会变得容易得多，因为您的对象在使用完后会自动被回收。 当你第一次体验这种编程的时候，它看起来就像是魔术一般。它很容易给人留下这样的印象：你不必考虑内存管理，但这并不完全正确。 

思考下面这个简单的堆栈实现。

```java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
   		elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
        	throw new EmptyStackException();
        return elements[--size];
    }
    
    /**
    * Ensure space for at least one more element, roughly
    * doubling the capacity each time the array needs to grow.
    */
    private void ensureCapacity() {
        if (elements.length == size)
        	elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

这个程序没有明显的错误(但是通用版本请参阅项目29)。你可以对它进行详尽的测试，它可以通过所有的测试，但是有一个潜在的问题。笼统地说，，该程序存在“内存泄漏”，由于垃圾收集器活动增加或内存占用增加，可能会悄悄地表现出性能下降的情况。在极端情况下，这样的内存泄漏可能会导致磁盘分页甚至程序失败并出现OutOfMemoryError，但这种故障相对较少。