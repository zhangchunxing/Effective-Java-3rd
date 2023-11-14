# 优先使用执行器，任务和流，而不是线程

本书的第1版， 实现了一个简单的工作队列（`work queue`）。这个类允许客户端将一个任务放到队列中异步处理，等待被后台线程执行。当这个工作队列没有用的时候，客户端还可以通过调用一个方法，要求后台线程优雅地终止自己，前提是队列中的任务都已经被完成。这个实现只不过是一个玩具，但即便如此，它也需要一整页微妙的代码，如果你做得不好，就很容易出现安全问题和线上故障（` liveness failure`）。幸运的是，你再也没有理由编写这种代码了。

本书第2版出版时，`java.util.concurrent`已经添加到`Java`中。这个代码包下，有一个执行器框架（`Executor Framework`），它是一个可扩展的，基于接口的任务执行工具。现在，创建一个工作队列只需要一行代码，而且它在各方面都比第1版更好的。

```java
ExecutorService exec = Executors.newSingleThreadExecutor();
```

如何提交一个可执行的任务，如下：

```java
exec.execute(runnable);
```

接下来是，如何让执行器优雅地终止（如果你没有这样做，你的虚拟机很可能不会退出）：

```java
exec.shutdown();
```

你可以使用执行器服务做更多的事情。比如，你可以等待一个特定任务的完成（如条款79所示，通过`get`方法）。你也可以等待一组任务中的任意一个的完成或者全部完成（使用`invokeAny`或`invokeAll`方法）。你可以等待执行器服务的终止（使用` awaitTermination`）。当这些任务都完成事，你还可以一个个地遍历任务执行的结果（使用` ExecutorCompletionService`）。你也可以安排任务在特定的时间运行，或者定期运行（使用`ScheduledThreadPoolExecutor`)等等。

如果想让不止一个线程来处理来自这个队列的请求，只要调用一个不同的静态工厂，这个工厂创建了一种不同的`executor service`，称作线程池（`thread pool`）。你可以用固定或者可变数目的线程创建一个线程池。`java.util.concurrent.Executors`类包含了静态工厂，能为你提供所需的大多数`executor`。然而，如果你想来点特别的，可以直接使用`ThreadPoolExecutor`类。这个类允许你控制线程池操作的几乎每个方面。

为特定的应用程序选择执行器服务（`executor service`）可能有些棘手。对于小型程序或负载较轻的服务器，使用`Executors.newCachedThreadPool`通常是一个不错的选择，因为它不需要配置，而且通常都能达到正确的效果。
