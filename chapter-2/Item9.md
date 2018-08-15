## 优先使用`try-with-resources`来代替`try-finally`

Java库包含许多必须通过手动调用`close`方法关闭的资源。其中包括：`InputStream` ， `OutputStream`  和
`java.sql.Connection`。资源的关闭经常被客户端忽略，而这会导致槽糕的性能后果。虽然这些资源中的许多都使用终结器作为安全网，但终结器并不能很好地工作(第8项)。

纵观历史，一个`try-finally`语句是保证一个资源被正确关闭的最好方法，即使面对异常或返回:

```java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```

这可能看起来不坏，但当你添加第二个资源时，情况会变得更糟：

```java
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
			} finally {
				out.close();
			}
		} finally {
			in.close();
	}
}
```

这可能难以置信，但即使是优秀的程序员也有很多错误时间。首先，我在《Java Puzzlers》[Bloch05]书中第88页里就犯过这错，并且多年来也没人注意到。事实上，在2007年的时候，Java库中三分之二的`close`方法的使用都是错误的。

即使使用`try-finally`语句关闭资源的正确代码(如前两个代码示例所示)也有一个细微的缺陷。try块和finally块中的代码都能够抛出异常。例如，在`firstLineOfFile`方法中，由于底层物理设备发生故障，对`readLine`的调用可能会抛出异常，而`close`的调用也可能出于同样的原因而失败。在这种情况下，第二个异常完全把第一个异常给覆盖了。异常堆栈跟踪中没有第一个异常的记录，这可能会使实际系统中的调试变得非常复杂——通常这是你希望看到的第一个异常，以便诊断问题。虽然可以通过编写代码来抑制第二个异常而支持第一个异常，但实际上没有人会这样做，因为它太过啰嗦。

当Java 7引入了`try-with-resources`语句[JLS，14.20.3]时，所有这些问题都被一举解决了。要想使用这个结构，资源必须实现`AutoCloseable`接口，这个接口由一个单独的`void close()`方法组成。现在Java库和第三方库中的许多类和接口去实现或扩展了`AutoCloseable`接口。如果你要编写一个代表必须关闭的资源的类，那么你的类也应该实现`AutoCloseable接口`。

下面是我们如何使用`try-with-resources`方式的第1个例子：

```java
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    	return br.readLine();
	}
}
```

接下来是我们如何使用`try-with-resources`的第2个例子：

```java
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
    		OutputStream out = new FileOutputStream(dst)) {
        
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
        	out.write(buf, 0, n);
    }
}
```

