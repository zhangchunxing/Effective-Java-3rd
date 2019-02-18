# 优先考虑泛型类型

通常，对声明进行参数化并使用JDK提供的泛型类型和方法并不太难。编写自己的泛型类型有点困难，但是值得学习如何编写泛型类型。

考虑条款7中的简单(玩具)堆栈实现：

```java
// Object-based collection - a prime candidate for generics
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
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        	return result;
    }
    
    public boolean isEmpty() {
        return size == 0;
    }
    
    private void ensureCapacity() {
        if (elements.length == size)
        	elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

这个类一开始就应该被参数化，但是因为它不是参数化的，所以我们可以在事后对它进行泛化。换句话说，我们可以在不损害原始非参数化版本的客户端的情况下对其进行参数化。按照目前的情况，客户端必须强制转换堆栈中弹出的对象，而这些强制转换可能在运行时失败。泛化一个类的第一步是向其声明中添加一个或多个类型参数。在这种情况下，有一个类型参数，表示堆栈的元素类型，该类型参数的常规名称是`E `(条款68)。

下一步是用恰当的类型参数来替换类型`Object`的所有用法，然后尝试编译得到的程序：

```java
// Initial attempt to generify Stack - won't compile!
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
    	elements = new E[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public E pop() {
        if (size == 0)
        	throw new EmptyStackException();
        
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    ... // no changes in isEmpty or ensureCapacity
}
```

