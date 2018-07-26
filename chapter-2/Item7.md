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

所以，内存泄漏在哪呢？如果一个堆栈，先增长然后收缩，那么出站的数据并不会被垃圾回收。即便程序中的堆栈不再引用他们了。这是因为堆栈维护着对他们的***过时的引用***。过时的引用就是一个普通的引用，它永远不会被取消引用。在这种情况下，元素数组的除“活动部分”之外的任何引用都是过时的。活动部分由索引小于数组长度的元素组成。

有垃圾收集功能的语言中的内存泄漏(更确切地说是无意识地对象保留)是潜伏的。如果一个对象引用被无意地保留了，那么不仅该对象被排除在垃圾收集之外，该对象引用的任何对象也是如此，等等。即使只有少数对象引用是无意保留的，但可能许多对象被阻止被垃圾收集，这可能会对性能造成很大的影响。

解决这类问题的方法很简单:一旦这些对象过时了，就把他们的引用指向null。在我们的Stack类的例子中，只要这个元素出栈了，它的引用就变得过时了。pop方法的正确版如下：

```java
public Object pop() {
    if (size == 0)
    	throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

将过时的引用指向null的另一个好处是，如果他们后面被错误地取消引用了，程序会立刻报NullPointerException 的错误，而不是静静地做错误的事情。尽可能快地发现程序的错误总是有益的。

当程序员第一次被这个问题所困扰时，他们可能会在程序使用完毕后立即清空每个对象引用，从而过度补偿。其实这是没必要的也是不希望的；这样做会使程序变得混乱。取消对象引用应该是例外而不是规范。消除过时引用的最佳方法是让包含引用的变量超出范围。如果您在尽可能小的范围内定义每个变量，那么刚刚提到的方法就自然会发生（第57条）。

所以什么时候应该取消对象引用呢？Stack类的哪些方面使它容易受到内存泄漏的影响?简单说，它需要自己管理内存。正常来讲，缓存池是由元素数组(对象引用单元格，而不是对象本身)的元素组成。数组中激活部分的元素被分配内存，剩下的部分应该释放内存。但是垃圾回收器不可能知道这些；对于垃圾收集器，元素数组中的所有对象引用都是同样有效的。只有程序员知道数组的非活动部分是不重要的。程序员可以通过手动取消那些已经变为非活跃部分的元素的引用，然后有效地把这件实事告诉垃圾收集器。

一般来说，**当一个类管理它自己的内存时，程序员应该警惕内存泄漏**。无论何时释放一个元素，元素中包含的任何对象引用都应该被取消。