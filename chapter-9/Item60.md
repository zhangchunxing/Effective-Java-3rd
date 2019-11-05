# 在需要精确答案的情况下请避免使用float与double

单精度浮点型和双精度浮点型主要用于科学和工程计算。它们执行二进制浮点运算，该运算经过精心设计，能够在大范围内快速提供精确的近似值。但是，它们不能提供准确的结果，并且它们不应该在需要精确结果的地方使用。单精度浮点型和双精度浮点型特别不适合进行货币计算。因为无法使用单精度浮点数或双精度浮点数来精确地表示0.1（或任何其他10的负幂）。

例如，假设你口袋里有1.03美元，你消费了42美分。你还剩下多少钱？下面是一个简单的程序片段，试图回答这个问题：

```java
System.out.println(1.03 - 0.42);
```

不幸的是，它输出了0.610000000001。这不是一个孤立的例子。假设你口袋里有一美元，你买了9台洗衣机，每台10美分。你能得到多少零钱？

```java
System.out.println(1.00 - 9 * 0.10);
```

根据这个程序片段，你将得到$ 0.09999999999999998。

你可能认为只需在打印之前对结果进行四舍五入即可解决问题，但不幸的是，这种方法并不总是有效。例如，假设你的口袋里有一美元，你看到一个架子上有一排好吃的糖果，它们的价格仅仅是10美分，20美分，30美分，等等，一直到一美元。从10美分的那颗开始，每一种糖买一个，直到你无法购买下一个糖为止。你买了多少个糖果，找了多少零钱？这里有一个简单的程序设计来解决这个问题：

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

如果你运行这个程序，你会发现你能买得起三块糖，而且你还剩下0.3999999999999999美元。这是错误的答案！解决这个问题的正确方法是使用`BigDecimal`、`int`或`long`来进行货币计算。

这里是前一个程序的一个简单转换，使用`BigDecimal`类型代替`double`。注意，这里使用的是`BigDecimal`的字符串构造函数，而不是它的`double`构造函数。这是为了避免在计算中引入不准确的值[Bloch05, Puzzle 2]：

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

如果你运行修改后的程序，你会发现你可以买4块糖果，还剩下0美元。这才是正确答案。

然而，使用`BigDecimal`有两个缺点：相比于使用原生算术类型，它要麻烦得多，而且要慢得多。如果你解决的是一个简单的问题，后一种缺点是无关紧要的，但前一种缺点可能会惹恼你。