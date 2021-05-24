# 在详细消息中包含捕获失败的信息

当程序由于未捕获的异常⽽失败时，系统会⾃动打印出异常的堆栈信息。堆栈信息包含了异常的字符串表示，即调⽤其`toString`⽅法的结果。这通常会包含异常的类名，后跟其详细消息。这常常是程序员或是`SRE`（⽹站可靠性⼯程师）在探究软件失败时所拥有的唯⼀信息。如果失败不那么容易复现，那么获取进⼀步的信息将会变得异常困难或是压根⼉就不可能。因此，对于异常的`toString`⽅法来说，返回尽可能多的与失败原因相关的信息就变得极其重要了。换⾔之，异常的详细消息应该捕获失败以供后续分析。

**要想捕获失败，异常的详细消息应该包含导致异常的所有参数与字段的值**。⽐如说，`IndexOutOfBoundsException`的详细消息应该包含下边界、上边界以及不在上下边界之间的索引值。该信息就包含了与失败相关的很多内容。这3个值中的任何⼀个或是全部都有可能是错误的。实际的索引可能小于下界或等于上界（“越界错误”），或者它可能是个无效值，⽐下边界⼩很多或是⽐上边界⼤很多。下界也有可能大于上界（严重违反内部约束的一种情况）。每⼀种情况都代表了不同的问题，如果知道待寻找的错误是哪⼀种，就可以极大地加速诊断的过程。

关于安全敏感的信息这⾥有⼀个警告。由于堆栈信息可能会在诊断与修复软件问题的过程中被很多⼈看到，因此请不要在详细消息中包含密码、密钥以及类似的信息。

虽然在异常的详细消息中包含所有相关的数据是很重要的事情，不过请不要包含太多的不必要信息。堆栈信息旨在与⽂档以及源代码配合起来进⾏分析所⽤。它通常会包含异常抛出时所在的精确⽂件与⾏号，以及栈上所有其他⽅法调⽤的⽂件与⾏号。对失败过度冗余的描述是没必要的；这些信息可以通过阅读⽂档与源代码来获取。

异常的详细消息不应该与⽤户级别的错误消息混为⼀谈，后者要能被最终⽤户所理解。与⽤户级别的错误消息不同的是，在分析失败时，详细消息主要针对于程序员或是`SRE`（站点可靠性⼯程师）。因此，信息内容要⽐可读性更加重要。⽤户级别的错误消息通常是本地化的，⽽异常详细消息则并⾮如此。

确保异常在其详细消息中包含足够的能捕获失败的信息，一种方法是在异常的构造方法中而不是字符串详细信息中引入这些信息。这样，详细信息就会自动生成出来以包含这些信息。例如，`IndexOutOfBoundsException`可以有一个这样的构造函数，而不是`String`构造方法：

```java
 /**
  * Constructs an IndexOutOfBoundsException.
  *
  * @param lowerBound the lowest legal index value
  * @param upperBound the highest legal index value plus one
  * @param index      the actual index value
  */
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
       // Generate a detail message that captures the failure
       super(String.format(
               "Lower bound: %d, Upper bound: %d, Index: %d",
               lowerBound, upperBound, index));
       // Save failure information for programmatic access
       this.lowerBound = lowerBound;
       this.upperBound = upperBound;
       this.index = index;
}

```

在`Java 9`中，`IndexOutOfBoundsException`终于有了接收一个`int`类型参数的构造方法，但遗憾的是它忽略了`lowerBound`和`upperBound`参数。更为普遍地，`Java`库并没有大规模使用这种方式，不过我强烈建议你这么做。这使得抛出异常的程序员能够轻松捕获到失败。实际上，不捕获失败反⽽变得困难了！事实上，这种⽅式将代码集中起来在异常类中声明⾼质量的详细消息，⽽非要求类的使⽤者再冗余地⽣成详细消息。

如条款70所建议的那样，对于异常来说，为失败捕获信息（在上述示例中就是`lowerBound`、`upperBound`与`index`）提供访问器方法是恰当的做法。相比于非受检来说，为受检异常提供这种访问器方法更加重要，因为失败的捕获信息对于失败恢复来说是很有用的。程序员很少（可以理解）会以编程的方式来访问未受检异常的详细消息。不过，对于非受查异常来说，作为通用规则，提供这些访问器也是明智之举。