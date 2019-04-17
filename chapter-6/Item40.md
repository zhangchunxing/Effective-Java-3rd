# 始终如一地使用Override注解

Java库提供了几个注解类型。对于程序员来说，最重要的注解当属`@Override`了。该注解只能用在方法声明上，表示被注解的方法声明重写了父类型中的声明。如果始终如一地使用该注解，那么它会保护你免受Bug的困扰。考虑如下程序，其中的类`Bigram`表示一个双字母组合，或是有序的一对字母：

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

主程序会重复添加26个双字母组合，每个都包含两个相同的小写字母，并将其添加到`Set`中。接下来，它会打印出该`Set`的大小。你可能觉得该程序会打印出26，因为`Set`不能包含重复的元素。如果运行该程序，你会发现它打印的是26而非260。到底是哪里出问题了呢？

显然，`Bigram`类的作者想要重写`equals`方法（条款10），甚至还记得重写了与之相关的`hashCode`方法（条款11）。但遗憾的是，这个运气欠佳的程序员并没有重写`equals`，而是对其进行了重载（条款52）。要想重写`Object.equals`，你必须要定义一个参数类型为`Object`的`equals`方法，但`Bigram`的`equals`方法的参数并非`Object`类型，这样`Bigram`只是从`Object`继承了`equals`方法。这个`equals`方法用来测试对象的相等性，就像==运算符一样。每个双字母组合的10个副本中的每一个都与其他9个不同，因此`Object.equals`的结果注定是不等的，这就说明了程序打印出260的原因所在。

幸好，编译器可以帮助你找到错误，不过前提是你得告诉编译器想要重写`Object.equals`。为了做到这一点，请在`Bigram.equals`上加上`@Override`注解，如下代码所示：

```java
@Override
public boolean equals(Bigram b) {
	return b.first == first && b.second == second;
}
```

如果加上这个注解并再次编译程序，编译器就会生成如下所示的错误消息：

```java
Bigram.java:10: method does not override or implement a method
from a supertype
@Override public boolean equals(Bigram b) {
^
```

你会立刻发现哪里出问题了，可能会拍一下自己的脑袋，然后使用如下这个正确的`equals`实现（条款10）来替换掉有问题的：

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram))
    	return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
}
```

因此，你应该在每个重写了父类声明的方法声明上使用`Override`注解。不过，这个规则有一个小小的例外。如果编写的类没有声明为`abstract`，即便你相信它重写了父类中的抽象方法，也不需要在该方法上使用`Override`注解。因为在没有声明为`abstract`的类中，如果没有重写抽象的父类方法，那么编译器会生成一条错误消息。不过，对于类中重写了父类方法的所有方法声明来说，你可能希望重点强调一下，那么在这种情况下，你还是可以为这些方法加上`Override`注解。大多数IDE都可以通过设置来为那些重写了父类方法的方法声明上自动加上`Override`注解。

大多数IDE都有另外一个原因来始终如一地使用`Override`注解。如果启用了恰当的检查，那么当有方法没有加上`Override`注解，但又重写了父类的方法，IDE就会生成警告。如果始终如一地使用`Override`注解，那么这些警告就会对无意的重写发出警告。这对编译器的错误消息形成了补充，提示你那些无意中的重写失败情况。在IDE与编译器之间，你可以确保真正重写了想要重写的方法，没有重写不想重写的方法。

`Override`注解可以用在重写了接口或是类中的方法声明的方法声明上。随着默认方法的引入，一个好的做法是在接口方法的具体实现上使用`Override`注解来确保签名是正确的。如果你知道接口没有默认方法，那么可以选择忽视在接口方法的具体实现上使用`Override `注解，以减少混乱。

不过，在抽象类或是接口中，好的做法是对你认为重写了父类或是父接口方法的所有方法上加上`Override`注解，无论他们是具体的还是抽象的。比如说，`Set`接口没有对`Collection`接口增加新方法，这样它就应该对所有方法声明添加`Override`注解来确保没有无意中向`Collection`接口增加任何新方法。

总结一下，如果在每个重写了父类型声明的方法声明上使用`Override`注解，那么编译器就会使你免于很多错误的困扰，不过有一个例外。在具体类中，不需要在重写抽象方法声明的方法上使用`Override`注解（但这么做也没什么问题）。