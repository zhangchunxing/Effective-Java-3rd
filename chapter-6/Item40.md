# 始终如一地使用Override注解

Java库包含几种注释类型。对于典型的程序员来说，其中最重要的是`@Override`。这个注解只能用于方法声明，它表示被注解的方法声明覆盖了超类中的声明。如果你始终使用这个注解，它将保护你免受大量恶意bug的攻击。考虑下面程序，其中类`Bigram`表示一个双字母组，或有序的字母对：

```java
// Can you spot the bug?
public class Bigram {
    private final char first;
    private final char second;
    
    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    
    public boolean equals(Bigram b) {
    	return b.first == first && b.second == second;
    }
    
    public int hashCode() {
    	return 31 * first + second;
    }
    
    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
        	for (char ch = 'a'; ch <= 'z'; ch++)
        		s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

主程序反复向一个集合添加26个双字母组，每个双字母组由两个相同的小写字母组成。然后它打印出这个集合的大小。你可能希望程序打印26，因为集合不能包含重复的数据。如果你试着运行这个程序，你会发现它输出的不是26而是260。出了什么问题呢?

显然，`Bigram`类的作者打算重写`equals`方法（条款10），甚至还记得同时重写`hashCode`（条款11）。不幸的是，我们倒霉的程序员没有重写`equals`，而是重载了它（条款52）。为了能重写`Object.equals`方法，你必须定义一个参数为`Object`类型的`equals`方法，所以`Bigram`只是继承了`Object`的`equals`方法。这个`equals`方法是用来测试对象一致性的，就像==操作符一样。每一个双字母组在它的10个副本中都与其它9份不同，因此通过`Object.equals`方法，它们被认为是不相等的。这就解释了为什么程序输出260。

幸运的是，编译器可以帮助你找到这个错误，但前提是你要告诉它你打算覆盖`Object.equals`方法。为此，请在`Bigram.equals`上使用`@override`注解，如下所示：

```java
@Override
public boolean equals(Bigram b) {
	return b.first == first && b.second == second;
}
```

如果你插入此注解并尝试重新编译程序，编译器将生成如下错误消息：

```java
Bigram.java:10: method does not override or implement a method
from a supertype
@Override public boolean equals(Bigram b) {
^
```

你会立刻意识到自己做错了什么，并拍自己的额头，然后用正确的做法替换掉错误的`equals`实现（条款10）：

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram))
    	return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
}
```

因此，你应该在每一个你确信覆盖了父类中声明的方法声明上使用`Override`注解。这条规则有一个小小的例外。如果你正在写一个非抽象类，并且你确信它覆盖了它父类中的抽象方法，然而，你不必费心将`Override`注解放在该方法上。在没有声明为抽象的类中，如果你没有覆盖抽象父类方法，编译器将发出错误消息。不过，你可能希望将注意力放在你的类中所有覆盖了超类方法的方法上，在这种情况下，你也可以随意地给这些方法加上注解。大多数ide都可以设置成自动插入`Override`注解，当你选择了一个重写方法时。