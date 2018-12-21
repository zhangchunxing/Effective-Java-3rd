## 优先使用`try-with-resources`来代替`try-finally`

Java库包含许多必须通过手动调用`close`方法关闭的资源。其中包括：`InputStream` ， `OutputStream`  和
`java.sql.Connection`。关闭资源常常会被客户端所忽视，这会导致可怕的性能问题。虽然很多资源使用了终结器来作为安全网，不过终结器却并不那么尽如人意（条款8）。

纵观历史，`try-finally`语句是保证资源被正确关闭的最好方法，即便在遇到异常或是返回语句时亦如此：

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

看起来还不错，不过当添加了第二个资源时情况就变得有些糟糕了：

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

这可能难以置信，但即使是优秀的程序员也有犯这个错误的时候。对于初学者来说，我在Java Puzzlers [Bloch05]的第88页指出了问题，但多年来没人注意到。事实上，2007年，在Java库中对close方法的使用有2/3是错误的。

即使使用`try-finally`语句关闭资源的正确代码(如前两个代码示例所示)也有一个细微的缺陷。try块和finally块中的代码都能够抛出异常。例如，在`firstLineOfFile`方法中，由于底层物理设备发生故障，对`readLine`的调用可能会抛出异常，而`close`的调用也可能出于同样的原因而失败。在这种情况下，第二个异常完全把第一个异常给覆盖了。异常堆栈跟踪中没有第一个异常的记录，这可能会使实际系统中的调试变得非常复杂——通常这是你希望看到的第一个异常，以便诊断问题。虽然可以通过编写代码来抑制第二个异常而支持第一个异常，但实际上没有人会这样做，因为它太过啰嗦。

当Java 7引入了`try-with-resources`语句[JLS，14.20.3]时，所有这些问题都被一举解决了。要想使用这个结构，资源必须实现`AutoCloseable`接口，该接口包含了唯一一个返回`void`类型的`close`方法。。现在Java库和第三方库中的许多类和接口去实现或继承了`AutoCloseable`接口。如果你要编写一个代表必须关闭的资源的类，那么你的类也应该实现`AutoCloseable接口`。

如下代码使用`try-with-resources`改写了上面第一个示例：

```java
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    	return br.readLine();
	}
}
```

如下代码使用`try-with-resources`改写了上面第二个示例：

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

与以前的版本相比，`try-with-resources`版本不仅更短，可读性更好，而且提供了更好的诊断。仔细想想`firstLineOfFile`方法。如果`firstLineOfFile`和`close`方法（不可见）都抛出了异常，则后一个异常将被抑制，来支持前一个异常。 实际上，可能会抑制多个异常，而保留你实际希望看到的异常。这些被压制的异常并不是被丢弃掉；它们会被打印到堆栈信息里，并用一个标记来说明它们是被抑制的。在程序中你可以用`getSuppressed`方法来访问它们，该方法是在Java 7中被添加到`Throwable`中的。

你可以将`catch`从句放到`try-with-resources`语句上，就像在正常的`try-finally`语句中那样。这样就可以在处理异常的同时又不会在另一个嵌套层次上搞乱代码了。举个例子，举个例子下面是不抛出异常的`firstLineOfFile`方法版本，不过如果无法打开文件或是无法读取文件，那么它会接收一个默认值来返回：

```java
/ try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    	return br.readLine();
    } catch (IOException e) {
    	return defaultVal;
    }
}
```

结论很明显：当使用了必须关闭的资源时，总是优先使用`try-with-resources`，来代替`try-finally`。结果代码更短、也更清晰，它所生成的异常也更加有用。`try-with-resources`语句使得编写使用了必须要关闭的资源的代码更加轻松，而这是`try-finally`所做不到的。
