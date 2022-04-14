# 对共享可变资源的同步访问

关键字`synchronized`可以确保，同一时刻只有一个线程能够执行方法或代码块。许多程序员认为同步仅仅是互斥的一种方式，当一个对象被一个线程修改时，它可以阻止其它线程看到这个对象的中间态。按照这个观点，对象被创建的时候出于一致的状态（Item 17），当有方法访问它的时候，它就被锁定了。
这些方法观察到对象的状态，并且可能会引起状态转变，即把对象从一种一致的状态转换到另一种不一致的状态。正确地使用同步可以保证没有任何方法会看到对象处于不一致的状态中。

这个观点是正确的，但他只说了一半。没有同步，一个线程的更改可能对其他线程不可见。`synchronization`不仅可以阻止线程看到状态不一致的对象，还可以确保每个进入同步方法或者同步代码块的线程都能看到由相同锁保护的所有以前修改的效果。

`Java`语言规范保证了对变量的读写操作都是原子的（`long`或`double`类型的变量除外） [JLS, 17.4, 17.7]。换句话说，在没有同步的情况下，即使多个线程在并发地修改某个变量，此时读取该变量（`long`或`double`类型的变量除外），也是可以保证返回某个线程存储在该变量中的值的。

你可能听说过，为了提高性能，在读写原子数据时应该避免同步。这个建议是危险的、错误的。虽然语言规范保证线程在读取字段时不会看到不一致的值，但它不能保证一个线程写入的值对另一个线程是可见的。**线程间的可靠通信和互斥都需要同步**。 
这是因为语言规范里有一部分叫作**内存模型**的内容，它描述了一个线程所做的更改何时以及如何对其他线程可见的[JLS, 17.4;Goetz06, 16]。

即使数据的读写是原子性的，对共享的可变数据的访问不做同步的后果也可能是严重的。考虑一个线程停止另一个线程的情况。`JDK`提供了`Thread.stop`方法，但是这个方法在很久以前就被弃用了，因为它本质上是不安全的，使用它会导致数据损坏。所以，不要去使用`Thread.stop`。
从另一个线程去停止一个线程的建议方法是让第一个线程轮询一个`boolean`字段，该字段最初为`false`。但可以由第二个线程将其设置为`true`，以此来通知第一个线程，将自己停止。因为读取和写入`boolean`字段是原子的，所以一些程序员在访问这类字段时，没有做同步：

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

你预期这个程序大约运行一秒钟。因为主程序线程设置`stopRequested`为`true`，从会导致后台线程的循环终止。然而，在我的机器上，这个程序永远不会终止：后台线程会一直循环下去！

问题就是没有做同步，所以无法保证后台线程什么时候可以看到由主线程发出的`stopRequested`值的变化。在没有同步的情况下，虚拟机把这段代码：

```java
while (!stopRequested)
    i++;
```

转变成下面代码，也是完全可以接受的。

```java
if (!stopRequested)
    while (true)
        i++;
```

这种优化是有益的，这正是`OpenJDK Server VM`所做的。事实上，结果失败了：程序没有正常运行。解决这个问题的一种方法是对`stopRequested`字段进行同步访问。然后，这个程序会如预期那样，大约在一秒内终止：

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

注意，写方法（requestStop）和读方法（stopRequested）都是同步的。只同步写方法是不够的！**除非读和写操作都同步，否则不能保证同步起作用**。
有时候，只同步写（或读）的程序可能在某些机器上可以正常运行，但这种表面现象是具有欺骗性的。

即使不做同步，`StopThread`中的同步方法的动作也是原子的。换句话说，在这些方法上做同步仅仅是为了达到通信的效果，而不是互斥。
虽然在循环的每次迭代上进行同步的成本很小，但还是有一个正确的替代方案，它不那么冗长，而且性能可能更好。如果`stopRequested`被声明为`volatile`，那么第二个版本的`StopThread`中的锁可以省略。
虽然`volatile`修饰符起不到互斥的作用，但它保证任何读取该字段的线程都能看到最近一次写入的值。

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