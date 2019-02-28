# 优先考虑泛型类型

一般来说，参数化声明并使用JDK所提供的泛型类型与方法并不是太难的事情。不过，编写自己的泛型类型就不是那么简单的了，但这是值得我们去学习的：

考虑如下来自于条款7的简单的栈实现：

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

这个类本应该一开始就参数化的，不过既然没有，那我们就将其泛型化吧。换句话说，我们可以在不影响之前非参数化版本的客户端的情况下将其参数化。根据现在的情况，客户端需要将从栈中弹出的对象进行类型转换，这些转换有可能在运行期出现失败。泛型化一个类的第一步就是向其声明添加一个或多个类型参数。在该示例中有一个类型参数，它表示栈中的元素类型，该类型参数约定的名字叫做`E`（条款68）。

接下来需要将所有使用到`Object`类型的地方替换为恰当的类型参数，然后编译修改后的程序：

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

你至少会收到一个错误或是警告，这个类也不例外。幸好，这个类只有一个错误：

```java
Stack.java:8: generic array creation
        elements = new E[DEFAULT_INITIAL_CAPACITY];
                   ^
```

如条款28所述，你不能创建非具化类型的数组，比如说`E`。这个问题在每次编写背后是数组的泛型类型时都会出现。有两种合理的方式可以解决这一问题。第一种是直接规避泛型数组创建的限制：创建一个Object的数组，并将其转换为泛型数组类型。现在，错误消失了，不过编译器会生成一个警告。这种用法是合法的，但并非（一般来说）类型安全：

```java
Stack.java:8: warning: [unchecked] unchecked cast
found: Object[], required: E[]
       elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
             	      ^
```

编译器无法证明程序是类型安全的，不过你自己可以。你必须要说服自己，未检查的类型转换并不会危害到程序的类型安全。涉及到的数组（`elements`）存储在一个私有字段中，永远不会返回给客户端或是传递给其他方法。存储在数组中的唯一元素就是传递给`push`方法的参数，其类型是`E`。因此，未检查的类型转换就不会产生什么问题。

一旦证明未检查的类型转换是安全的，那就请在最小的作用域范围内压制警告（条款27）。对于该示例来说，构造方法只包含了未检查的数组创建，因此在整个构造方法上压制警告就是很恰当的。当增加了注解后，`Stack`就可以干净地通过编译，使用时就无需再进行显式类型转换，也不用再担心出现`ClassCastException`了：

```java
// The elements array will contain only E instances from push(E).
// This is sufficient to ensure type safety, but the runtime
// type of the array won't be E[]; it will always be Object[]!
@SuppressWarnings("unchecked")
public Stack() {
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

消除`Stack`中泛型数组创建错误的第2种方式是将字段`elements`的类型由E[]修改为`Object[]`。如果这样做了，那么你会收到另一个错误：

```java
Stack.java:19: incompatible types
found: Object, required: E
       E result = elements[--size];
                          ^
```

可以通过将从数组中接收到的元素类型转换为`E`来把这个错误转换为警告：

```java
Stack.java:19: warning: [unchecked] unchecked cast
found: Object, required: E
       E result = (E) elements[--size];
                              ^
```

由于`E`是个非具化的类型，编译器没有任何办法可以在运行期检查这个转换。又一次，你可以证明这个未检查的转换是安全的，因此我们可以压制警告。根据条款27的建议，我们只在包含未检查转换的赋值上压制警告，而不是在整个`pop`方法上：

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

这两种消除泛型数组创建的技术都有各自的拥趸。第一种方式可读性更好：数组被声明为`E[]`类型，这清晰地表明了它只包含`E`实例。此外，这种方式也更加简洁：在典型的泛型类中，你会在代码的很多地方读取数组；第一种方式只需要进行一次转换即可（即在创建数组时），而第二种方式则需要在每次读取数组元素时都要进行一次转换。这样，第一种方式应该是首选，并且在实际情况下用得也更多。不过，这会导致堆污染（条款32）：数组的运行期类型与其编译期类型不匹配（除非E碰巧就是`Object`）。这使得一些程序员会倾向于选择第二种方式，不过堆污染在这种情况下并不会产生什么问题。

如下程序演示了泛型类`Stack`的使用。程序会按照倒序打印出命令行参数，并将其转换为大写。对于从栈中弹出的元素来说，调用`String`的`toUpperCase`方法并不需要进行显式的类型转换；同时，自动生成的类型转换是一定可以成功的：

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