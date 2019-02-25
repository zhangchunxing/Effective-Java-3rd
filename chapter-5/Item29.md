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

通常你至少会得到一个错误或警告，这个类也不例外。幸运的是，这个类只生成一个错误：

```java
Stack.java:8: generic array creation
        elements = new E[DEFAULT_INITIAL_CAPACITY];
                   ^
```

正如第28项中所解释的，你不能创建不可具体化类型的数组，如`E`。你每次写下一个泛型类型数组的时候，这个问题就会出现。有两种合理的方法可以解决这个问题。第一个解决方案直接绕过了创建泛型数组的禁令：创建Object数组并将其转换为泛型数组类型。现在，编译器将发出一个警告，而不是错误。一般来说，这种用法是合法的，但它不是类型安全：

```java
Stack.java:8: warning: [unchecked] unchecked cast
found: Object[], required: E[]
       elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
             	      ^
```

编译器可能无法证明你的程序是类型安全的，但你可以。你必须说服自己，未经检查的强制转换不会损害程序的类型安全。问题中的数组(`elements`)存储在私有字段中，并且从未返回给客户端或传递给任何其他方法。存储在数组中的元素仅仅传递给了`push`方法，方法中的元素类型也是`E`，所以未检查的强制转换不会造成任何危害。

一旦你证明了未经检查的强制转换是安全的，请将警告限制在尽可能小的范围内(条款27)。在这种情况下，构造函数只包含了未检查的数组创建，因此在整个构造函数中去抑制警告是合适的。通过添加注解来抑制异常，`Stack`可以干净地编译，并且你可以使用它而不用显式强制转换或担心`ClassCastException`。

```java
// The elements array will contain only E instances from push(E).
// This is sufficient to ensure type safety, but the runtime
// type of the array won't be E[]; it will always be Object[]!
@SuppressWarnings("unchecked")
public Stack() {
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

消除`Stack`中泛型数组创建错误的第二种方法是将字段`elements`的类型从`E[]`更改为`Object[]`。如果你这样做，你会得到一个不同的错误：

```java
Stack.java:19: incompatible types
found: Object, required: E
       E result = elements[--size];
                          ^
```

你可以通过将从数组中检索到的元素转换为`E`来将这个错误转换为一个警告，但是你会得到一个警告：

```java
Stack.java:19: warning: [unchecked] unchecked cast
found: Object, required: E
       E result = (E) elements[--size];
                              ^
```

因为`E`不是一个具体的类型，编译器无法在运行时检查类型类型转换的。但是，你可以很容易地向自己证明未经检查的强制类型转换是安全的，所以抑制警告是允许的。根据条款27的建议，我们仅在包含未了未检查的类型转换的赋值语句上去抑制警告，而不是在整个`pop`方法上：

```java
// Appropriate suppression of unchecked warning
public E pop() {
    if (size == 0)
    	throw new EmptyStackException();
    
    // push requires elements to be of type E, so cast is correct
    @SuppressWarnings("unchecked")
    E result = (E) elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

这两种消除范型数组创建的技术都有人使用。第一个方法可读性更强：数组被声明为类型`E[]`，这清楚地表明它只包含实例E。它也更简洁：在一个典型的泛型类中，从代码中的许多点读取数组；第一种技术只需要一次转换（在创建数组的地方）而第二种技术在每次读取数组元素时都需要单独的转换。因此，第一种技术是可取的，在实践中更常用。但是，它确实会造成堆污染（条款32）：数组的运行时类型与其编译时类型不匹配(除非`E`恰好是`Object`)。这使得一些程序员非常不安，他们选择了第二种技术，尽管在这种情况下堆污染是无害的。

下面的程序演示了我们的泛型类`Stack`的使用。该程序以相反的顺序打印命令行参数并转换为大写。在从`Stack`弹出的元素上调用`String`的`toUpperCase`方法不需要显式转换，自动生成的转换保证成功：

```java
// Little program to exercise our generic Stack
public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for (String arg : args)
    	stack.push(arg);
    while (!stack.isEmpty())
    	System.out.println(stack.pop().toUpperCase());
}
```

上面的例子可能与条款28相矛盾，条款28鼓励优先使用列表而不是数组。在泛型类型中使用列表并不总是可行的或令人满意的。Java本身不支持列表，因此一些泛型类型（如`ArrayList`）必须在数组上实现。其他泛型类型（如`HashMap`）是在数组上实现的，以提高性能。

大多数泛型类型都像我们的`Stack`示例，因为它们的类型参数没有限制：

你可以创建一个`Stack<Object>`，`Stack<int[]>`，` Stack<List<String>> `或这任何其他对象引用类型的`Stack`。注意你不能创建原生类型的`Stack`：试着创建`Stack<int>`或`Stack<double>`，这将导致编译期错误。这是`Java`泛型类型系统的一个基本限制。你可以通过使用装箱的基本类型（条款61）来绕过这一限制。

有一些泛型类型限制其类型参数的允许值。比如说，考虑`java.util.concurrent.DelayQueue`的声明如下：

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

类型参数列表（`<E extends Delayed>`）要求实际的类型参数`E`是` java.util.concurrent.Delayed `的一个子类型。这允许`DelayQueue`的实现和客户端对`DelayQueue`的元素使用方法`Delayed `，而不需要显式强制转换或冒着`ClassCastException`的风险。类型参数`E`称为**有界类型参数**。注意，子类型关系的定义使得每个类型都是其自身的子类型[JLS, 4.10]，因此创建`DelayQueue<Delayed>`是合法的。

总之，相对于在客户端代码中进行类型转换，泛型类型更安全，也更容易使用。当你在设计新类型的时候，确保它们不需要类型转换就可以使用。这通常意味着使得类型泛型化。如果你有一些已经存在的类型，它们本应该泛型化的但是没有，那么去将他们泛型化。对初次使用这些类型的人来说，用起来更容易，在不破坏现有客户端的情况下（条款26）。