[TOC]

### Callable与Runnable

先看一下`java.lang.Runnable`：

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

可以看出`run()`方法的返回值为`void`类型，所以在任务执行完毕后无法返回任何结果。



然后再看一下`java.util.concurrent.Callable<V>`，它也是一个接口，并且也只生命了一个方法：

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

可以看出这是一个泛型接口，`call()`函数返回的类型就是传递进来的V类型。

一般情况下需要配合`ExecutorService`来使用`Callable`，在`ExecutorService`接口中声明了多个`submit()`方法的重载版本：

```java
<T> Future<T> submit(Callable<T> task);

<T> Future<T> submit(Runnable task, T result);

Future<?> submit(Runnable task);
```

一般情况下使用第一个和第三个方法较多。



### Future

`java.util.concurrent.Future`也是一个接口，定义如下：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

`Future`可以用来对于具体的`Runnable`或者`Callable`任务执行结果进行取消、查询完成状态和获取执行结果。必要时可以通过`get()`方法来获取执行结果，该方法会阻塞线程直到任务返回结果。

下面对`Future`中定义的方法进行解释：

* `cancel()`：用于取消任务，如果任务取消成功则返回`true`，否则返回`false`。参数`mayInterruptIfRunning`表示是否允许取消正在执行的任务，如果设置为`true`则表示允许取消正在执行过程中的任务。如果任务已经执行完毕，则无论`mayInterruptIfRunning`的值为什么，`cancel()`方法一定返回false；如果任务尚未执行完毕，且`mayInterruptIfRunning`设置为`true`，则返回`true`；如果`mayInterruptIfRunning`为`false`，则返回`false`。如果任务尚未开始执行，则无论`mayInterruptIfRunning`的值为何，`cancel()`方法都会返回`true`。
* `isCancelled()`：用于查询任务是否已经被成功的取消，如果在任务执行完毕前（无论任务是否开始执行）被取消，则返回`true`。
* `isDone()`：用于查询任务是否已经执行完毕。
* `get()`：用于获取任务的执行结果，该方法会阻塞当前线程直到任务执行完毕返回结果。
* `get(long timeout,TimeUnit unit)`：用于获取任务的执行结果，如果在指定的时间内还没有获取到执行结果，则直接返回`null`。该方法也会阻塞线程。

综上，`Future`提供了三种功能：

1. 判断任务是否执行完毕；
2. 中断任务；
3. 获取任务执行结果。

### FutureTask

由于`Future`是一个接口，无法直接使用，故有了它的实现`FutureTask`。

`FutureTask`实现了`RunnableFuture`接口，我们先来看一下`RunnableFuture`的定义：

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

可以看出`RunnableFuture`继承了`Runnable`和`Future`，而`FutureTask`实现了`RunnableFuture`接口，所以它既可以作为`Runnable`被线程执行，也可以作为`Future`得到`Callable`的返回值。



