# 在需要精确答案的情况下请避免使用float与double

设计`float`与`double`类型的主要目的是用于科学与工程计算。他们会执行二进制浮点运算，这种运算经由谨慎的设计以在各种量级上快速提供精确的近似值。不过，他们并不会提供确切的结果，也不应该用在需要精确结果的场景下。`float`与`double`类型特别不适合用在货币计算的情况下，因为我们无法将0.1（或是其他的10的负指数方）精确表示为`float`或是`double`。

比如说，假设你的口袋中有$1.03，然后花了42¢，那还剩下多少呢？如下是回答这个问题的一个幼稚的程序片段：

```java
System.out.println(1.03 - 0.42);
```

遗憾的是，上述代码会打印出0.6100000000000001。这并非一个孤立的情况。假设你口袋中有1美金，购买了9个螺母垫圈，每个价格10美分，那会找你多少钱呢？

```java
System.out.println(1.00 - 9 * 0.10);
```

根据上述程序片段，会找你$0.09999999999999998。

你可能认为这个问题可以通过在打印前对结果进行四舍五入来解决，不过遗憾的是，这么做是不行的。比如说，假设你口袋中有一美金，你看到货架上有一排美味的糖果，价格分别是10¢、20¢、30¢等等，一直到一美金。从10¢的开始，每种糖果买一个，一直到钱不够再买货架上的下一个糖果为止。你总共能买多少个糖果呢，会找你多少钱呢？如下是解决这个问题的一个幼稚的程序：

```java
// Broken - uses floating point for monetary calculation!
public static void main(String[] args) {
	double funds = 1.00;
	int itemsBought = 0;
	for (double price = 0.10; funds >= price; price += 0.10) {
		funds -= price;
		itemsBought++;
	}
	System.out.println(itemsBought + " items bought.");
	System.out.println("Change: $" + funds);
}
```

运行该程序，你会发现可以买3个糖果，剩余$0.3999999999999999，这是个错误的答案！解决该问题的正确之道是使用`BigDecimal`、`int`或是`long`来进行货币计算。

如下对上述程序进行了直接的转换，使用`BigDecimal`来代替`double`。注意到这里使用了`BigDecimal`的`String`构造方法而非其`double`构造方法。这么做的目的在于避免向计算中引入不精确的值[Bloch05, Puzzle 2]：

```java
public static void main(String[] args) {
	final BigDecimal TEN_CENTS = new BigDecimal(".10");
	int itemsBought = 0;
	BigDecimal funds = new BigDecimal("1.00");
	for (BigDecimal price = TEN_CENTS;
         funds.compareTo(price) >= 0;
         price = price.add(TEN_CENTS)) {
        
		funds = funds.subtract(price);
		itemsBought++;
	}
	System.out.println(itemsBought + " items bought.");
	System.out.println("Money left over: $" + funds);
}
```

运行上述修正的程序，你会发现可以购买4块糖果，剩余$0.00。这才是正确的答案。

不过，使用`BigDecimal`存在两点不足：它不如原生算数类型那么方便，而且还会慢一些。如果待解决的是单个小问题，那么后一个缺点倒还可以接受，不过前者可能会令你挠头。

`BigDecimal`的一个替代方案是使用`int`或是`long`，这取决于涉及到的数量，而且需要你自己来记录小数点。在该示例中，显而易见的做法是以分而非元来进行所有计算。如下是采用这种方式进行的直接转换：

```java
public static void main(String[] args) {
	int itemsBought = 0;
	int funds = 100;
	for (int price = 10; funds >= price; price += 10) {
		funds -= price;
		itemsBought++;
	}
    
	System.out.println(itemsBought + " items bought.");
	System.out.println("Cash left over: " + funds + " cents");
}
```

总结一下，在需要精确答案的情况下请不要使用`float`或是`double`来进行计算。如果想让系统来记录小数点，并且不介意不使用原生类型所带来的不方便和开销，那就请使用`BigDecimal`。使用`BigDecimal`会有一些额外的好处，你可以完全控制四舍五入，当操作需要进行四舍五入时，`BigDecimal`提供了8种模式可供选择。如果正在进行的业务计算需要强制的四舍五入行为，这就会很方便了。如果性能非常重要，你又不介意自己记录小数点，同时数量又不太大，那就请使用`int`或是`long`。如果数量没有超过9个小数点，可以使用`int`；如果没有超过18个数字，可以使用`long`。如果数量超过了18个数字，那就请使用`BigDecimal`。