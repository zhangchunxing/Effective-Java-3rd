# 消除未检查的警告

当你使用泛型编程时，你会看到很多编译器警告：未检查的转换警告，未检查的方法调用警告，未检查的参数化变量类型警告和未检查的转换警告。你使用泛型获得的经验越多，你得到的警告也就越少，但不要指望新编写的代码能够干净地编译。

大部分未检查的警告是很容易消除的。比如说，假设你偶然地写了下面这个声明：

```java
Set<Lark> exaltation = new HashSet();
```

编译器会轻轻地提醒你做错了什么：

> Venery.java:4: warning: [unchecked] unchecked conversion
> Set<Lark> exaltation = new HashSet();
> ^
> required: Set<Lark>
> found: HashSet

然后，你可以进行指定的纠正，使警告消失。注意，实际上你没必要指定类型参数，只要用Java7引进的钻石操作符（`<>`）表示一下就好。然后编译器就可以推断出正确的实际类型参数(在本例中为`Lark`)：

```java
Set<Lark> exaltation = new HashSet<>();
```

