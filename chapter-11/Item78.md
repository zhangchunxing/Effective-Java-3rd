# 对共享可变资源的同步访问

`synchronized`关键字可以确保，同一时刻只有一个线程能够执行方法或代码块。很多程序员都认为同步仅仅⽤来实现互斥，即当⼀个对象正在被⼀个线程修改时，别的线程是⽆法看到这个对象的不⼀致的状态的。从这个视⻆来看，对象是在⼀种⼀致的状态下被创建的（条款17）且被访问它的⽅法锁定。这些⽅法会观测对象状态，还可能会导致状态变换，即将对象从⼀种⼀致的状态转换为另外⼀种。恰当的使⽤同步可以确保不会有⽅法能够在不⼀致的状态下看到对象。

这种观点是正确的，不过只说了⼀半。如果不使⽤同步，那么⼀个线程的修改可能不会被其他线程看到。同步不仅可以防⽌线程在不⼀致的状态下看到对象，还可以确保进⼊到同步⽅法或是代码块的每个线程都能看到同⼀个锁所保护的之前全部修改的结果。

语⾔规范确保了对变量的读或是写都是原⼦的，除⾮变量类型是`long`或是`double `[JLS, 17.4, 17.7]。换⾔之，读取⾮`long`或是`double`的变量可以确保返回值就是某个线程存储到变量中的那个值，即便多个线程并发修改这个变量且没有使⽤同步亦如此。

你可能听说过，为了改进性能，在读或是写原⼦数据时不应该使⽤同步。这个建议是极其错误的。虽然语⾔规范可以确保⼀个线程在读取字段时不会看到中间值，但它⽆法保证⼀个线程所写⼊的值是否会对另⼀个线程可⻅。**对于线程间的可靠通信以及互斥访问来说，我们需要使⽤同步**。 这是由语⾔规范中的**内存模型**所决定的，它指定了⼀个线程所做的修改何时以及如何能被其他线程可⻅ [JLS, 17.4; Goetz06, 16]。

即便数据是原⼦性读写的，但没有做到对共享可变数据的同步访问的后果也是⾮常严重的。考虑在⼀个线程中停⽌另外⼀个线程这样的任务。`JDK`提供了`Thread.stop`⽅法，不过该⽅法在很久之前就不建议使⽤了，这是因为其内在的不安全性——使⽤它可能会导致数据损坏。请不要使⽤`Thread.stop`。在⼀个线程中停⽌另外⼀个线程的推荐做法是让第一个线程轮询⼀个初始值为`false`的`boolean`字段，该字段可以通过第二个线程设为`true`，以此来表示第一个线程要终止。由于读写`boolean`字段是原⼦性的，因此⼀些程序员在访问该字段时就没有加上同步：

```java
// Broken! - How long would you expect this program to run?
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

你可能认为该程序会运⾏⼤约⼀秒钟，接下来主线程将`stopRequested`设为`true`，这导致后台线程的循环终⽌。不过在我的机器上，该程序永远不会终⽌：后台线程会永远循环下去！

问题在于，当没有使⽤同步时，谁都⽆法保证后台线程何时会看到主线程对`stopRequested`值的修改。在没有使⽤同步时，虚拟机很有可能会将如下代码：

```java
while (!stopRequested)
    i++;
```

转换为：

```java
if (!stopRequested)
    while (true)
        i++;
```

这种优化叫做提升，`OpenJDK Server VM`就是这么做的。结果就是活跃失败（` liveness failure`）：程序⽆法继续进⾏。修复该问题的⼀种⽅式就是对`stopRequested`字段的访问进⾏同步。该程序会在⼀秒钟左右终⽌，如预期⼀样：

```java
// Properly synchronized cooperative thread termination
public class StopThread {
    private static boolean stopRequested;
    
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested()) 
                i++;
        });
        
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

注意到这⾥的写⽅法（`requestStop`）与读⽅法（`stopRequested`）都是同步的。只同步写方法是不够的！**只有读与写操作都是同步的才能确保同步会起作⽤**。有时，只同步了写（或是读）操作的程序在某些机器上看起来也能正常运⾏，不过在这种情况下，表象是骗⼈的。

`StopThread`中同步⽅法的动作是原⼦的，甚⾄不加同步亦如此。换⾔之，这些⽅法中的同步仅仅⽤于通信⽬的，⽽不是为了互斥。虽然在循环中的每个迭代上进⾏同步的代价会很⼩，但还是存在更为正确的⽅式，其代码更加简洁，性能也要更好⼀些。如果将`stopRequested`声明为`volatile`，那么第二个版本的`StopThread`中的锁就可以去掉了。虽然`volatile`修饰符不会进⾏互斥操作，不过它可以保证读取该字段的任何线程都会看到最近所写⼊的值：

```java
// Cooperative thread termination with a volatile field
public class StopThread {
    private static volatile boolean stopRequested;
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
    
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

使⽤`volatile`时要多加⼩⼼。考虑如下⽅法，我们期望这个⽅法会⽣成序列号：

```java
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

该⽅法的意图是确保每次调⽤都会返回⼀个唯⼀值（只要调用次数不超过2³²）。⽅法的状态包含了单个原⼦性可访问的字段`nextSerialNumber`，该字段的所有可能值都是合法的。因此，⽆需同步来保护其不变性。不过，缺少了同步，该⽅法是存在问题的。

问题在于，⾃增运算符（++）并⾮原⼦性的。它会对`nextSerialNumber`字段执⾏两个操作：⾸先，它会读取其值，然后写回⼀个新值，这个新值等于⽼的值加1。如果在线程读取⽼的值和写回新的值中间有另外⼀个线程读取了该字段，那么这个线程将会看到与第⼀个线程相同的值并返回同样的序列号。这是个安全失败（`safe failure`）：程序会计算出错误的结果。

修复`generateSerialNumber`的⼀种⽅式是向其声明添加`synchronized`修饰符。这可以确保多个调⽤不会交叠在⼀起，对该⽅法的每次调⽤都会看到所有之前调⽤的结果。⼀旦这么做了，你就可以，并且也应该将`volatile`修饰符从`nextSerialNumber`上移除。为了让该⽅法更加健壮⼀些，请使⽤`long`来代替`int`，或是在`nextSerialNumber`溢出时抛出异常。

不过，更好的⽅案则是遵循条款59的建议，使⽤类`AtomicLong`，它位于`java.util.concurrent.atomic`包中。该包为原⽣数据类型提供了针对单个变量的⽆锁、线程安全的编程模型。`volatile`只提供了同步的通信效果，该包还提供了原⼦性。这正是我们期待`generateSerialNumber`所拥有的，并且它要优于同步版本：

```java
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
	return nextSerialNum.getAndIncrement();
}
```

避免本条款所讨论的问题的最佳⽅式是不要共享可变数据。要么共享不变数据（条款17），要么完全不共享。换⾔之，请将可变数据限定为单个线程可访问。如果采取了该策略，那么请将其写到⽂档中，这样随着程序的不断演进，该策略会⼀直得到维护和保持。同样重要的是，请深刻理解你正在使⽤的框架与库，因为他们可能会在你⽆意间引⼊线程。

⼀个线程在⼀段时间内修改了数据对象，然后将其共享给其他线程，并且只同步共享对象引⽤的动作，这种做法是可以接受的。接下来，其他线程可以在⽆需进⼀步同步的情况下读取对象，只要该对象不会再次被修改即可。这种对象叫做有效不可变对象 [Goetz06, 3.5.4]。将这种对象引⽤从⼀个线程传递给其他线程叫做安全发布 [Goetz06, 3.5.3]。安全发布对象引⽤有多种⽅式：可以在类初始化过程中将其存⼊到静态字段内；可以将其存储到`volatile`字段、`final`字段，或是通过正常的锁机制来访问的字段中；还可以将其放到并发集合中（条款81）。

总结⼀下，**当多个线程共享可变数据时，对数据进⾏读或写的每个线程都必须要进⾏同步**。如果不使⽤同步，那就⽆法保证⼀个线程所做的修改会对其他线程可⻅。没有对共享可变数据进⾏同步的恶果将是活性失败和安全失败。这些故障是最难以调试的问题。这些问题可能是间歇性的，也依赖于时序，在不同虚拟机上的程序⾏为可能⼤相径庭。如果只需要线程间通信，⽽不需要互斥访问，那么`volatile`修饰符就是⼀种可接受的同步形式，不过要想正确使⽤好它并不是那么容易的事情。
