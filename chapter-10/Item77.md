# 不要忽略异常

虽然该建议看起来很直接，不过⼈们经常违背它，因此值得再次强调⼀下。当`API`设计者在⽅法声明处抛出了异常，他们是在告诉你⼀些事情。请不要忽略这个异常！忽略异常是很容易的事情，只要使⽤`try`语句将⽅法调⽤包围起来，然后保持`catch`块为空即可。

```java
// Empty catch block ignores exception - Highly suspect!
try {
	...
} catch (SomeException e) {
  
}
```

空`catch`块违背了异常的目的，⽽异常的⽬的则是强制你处理异常情况。忽略异常就好⽐是忽略⽕警⼀样——将其关掉，没⼈会知道是否真的有⽕情。你可以侥幸逃脱，但结果也可能是灾难性的。⽆论何时，当看到空的`catch`块时，脑海中应该响起警铃声。

有⼀些情况可以忽略掉异常。⽐如说，在关闭`FileInputStream`时。你没有修改⽂件的状态， 因此没必要执⾏任何恢复动作，你已经从⽂件中读取了所需的信息，所以没理由终⽌操作。将异常以⽇志的形式记录下来是明智之选，这样如果异常频繁出现时就可以探究其原因了。如果选择忽略异常，那么`catch`块就应该加上注释，说明为何要这么做，同时要将变量命名为`ignored`：

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // Default; guaranteed sufficient for any map

try {
	numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
	// Use default: minimal coloring is desirable, not required
  
}
```

本条款所给出的建议既适⽤于检查异常，也适⽤于未检查异常。⽆论异常表示⼀种可预测的异常情况还是编程错误，使⽤`catch`块忽略它都会导致程序在继续执⾏时会遭遇到错误情况。接下来，程序可能会在未来的某个时间点出现失败的情况，⽽出现问题的代码可能与问题根源之间并没有明显的关联关系。恰当地处理异常会避免完全失败的可能。仅仅将异常向外传播⾄少会让程序很快失败，这保留了信息以供调试失败所⽤。

