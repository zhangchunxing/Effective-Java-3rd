# 消除未检查的警告

在使用泛型进行编程时，你会看到很多编译器警告：未检查的类型转换警告，未检查的方法调用警告，未检查的参数化可变类型警告和未检查的转换警告。你的泛型使用经验越多，收到的警告就会越少，不过不要指望新写的代码能够干净地通过编译。

很多未检查异常都是很容易消除的。比如说，假设你不小心写出了如下声明：

```java
Set<Lark> exaltation = new HashSet();
```

编译器会清楚地提醒你哪里做的不对：

```java
Venery.java:4: warning: [unchecked] unchecked conversion
Set<Lark> exaltation = new HashSet();
^
required: Set<Lark>
found: HashSet
```

接下来可以根据提示进行修改，这样警告就消除了。注意，你不必非得指定类型参数，只需通过Java 7所引入的菱形运算符（`<>`）来表示类型参数存在即可这样，编译器就会推断出正确的实际类型参数（在该示例中就是`Lark`）：

```java
Set<Lark> exaltation = new HashSet<>();
```

有些警告则难以消除。本章就会介绍这种类型的警告示例。当你接收到需要一些思考才能消除的警告时，请一定要坚持住。请消除每一个未检查的警告。如果消除了所有警告，那就表明你的代码是类型安全的，这非常棒。这意味着运行期不会出现`ClassCastException`，同时也增强了你的自信：程序的行为与你的意图是保持一致的。

**如果无法消除某个警告，不过你可以确定引起警告的代码是类型安全的，那就通过`@SuppressWarnings(“unchecked”)`注解来压制这个警告（请只在这种情况下才这么做）**。如如果在尚未确定代码是类型安全的情况下就压制警告，那只会给自己一个安全的假象。代码可能在编译时没有出现任何警告，不过在运行期依旧会抛出`ClassCastException`。不过，如果忽略掉了你知道是安全的未检查警告（没有压制他们），那么当表示真正的问题的警告出现时，你可能就不会注意到。新的警告就会淹没在你没有消除的那些假警报之中。

`SuppressWarnings`注解可以用在任何声明之上，从单个的局部变量声明到整个类都可以。**请将`SuppressWarnings`注解的使用保持在最小的作用域中**。通常来说，这会是一个变量声明，以及非常短小的方法或是构造方法。永远不要在整个类之上使用`SuppressWarnings`。这么做会掩盖掉重要的警告信息。

如果发现在一个代码长度超过一行的方法或是构造方法上使用了`SuppressWarnings`注解，那就可以将其移动到局部变量声明上。这样就不得不声明一个新的局部变量，不过这么做是值得的。比如说，考虑如下`toArray`方法，它来自于`ArrayList`：

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
    	return (T[]) Arrays.copyOf(elements, size, a.getClass());
    
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
   	 	a[size] = null;
    return a;
}
```

如果你编译`ArrayList`，该方法会生成如下警告：

```java
ArrayList.java:305: warning: [unchecked] unchecked cast
return (T[]) Arrays.copyOf(elements, size, a.getClass());
^
required: T[]
found: Object[]
```

将`SuppressWarnings`注解放到`return`语句上是不合法的，因为它并非声明 [JLS, 9.7]。你可能会想将注解放到整个方法上，不过请不要这么做。相反，请声明一个局部变量来持有返回值，并将注解放到这个局部变量声明上，如下代码所示：

```java
// Adding local variable to reduce scope of @SuppressWarnings
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // This cast is correct because the array we're creating
        // is of the same type as the one passed in, which is T[].
        @SuppressWarnings("unchecked") T[] result = 
            (T[]) Arrays.copyOf(elements, size, a.getClass());
        
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    
    if (a.length > size)
    	a[size] = null;
    return a;
}
```

这样得到的方法会干净地通过编译，并且将压制未检查警告的范围缩减到最小。

**每次使用`@SuppressWarnings("unchecked”)`注解时，请添加一条注释，说明为何这么做是安全的**。这会帮助他人更好地理解代码，更为重要的是，这会减少其他人修改代码导致计算变得不安全的几率。如果觉得这样的注释不好写，请好好想想。毕竟，你知道未检查操作是不安全的。

总结一下，未检查警告是很重要的，请不要忽略他们。每个未检查警告都代表了运行期出现`ClassCastException`的可能性。请尽全力消除掉这些警告。如果无法消除某个未检查警告，并且可以确定引起警告的代码是类型安全的，那就请在最小的作用域中使用`@SuppressWarnings("unchecked”)`注解来压制警告。请在注释中说明压制警告的原因是什么。