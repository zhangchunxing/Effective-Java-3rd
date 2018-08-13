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

