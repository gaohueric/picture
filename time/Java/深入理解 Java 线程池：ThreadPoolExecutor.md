##### 线程池介绍

在web开发中，服务器需要接受并处理请求，所以会为一个请求来分配一个线程进行处理，如果每次请求都创建一个线程的话实现起来非常简便，但是存在一个问题。

如果并发请求数量非常多，但每个线程执行的时间很短，这样就会频繁的创建和销毁线程，如此一来会大大降低系统的效率，可能出现服务器在为每个请求创建新线程和销毁线程上花费的时间和消耗的系统资源要比处理实际的用户请求的时间和资源更多。

线程池的目的就是执行完一个任务，并不销毁，而是可以继续执行其他任务。线程池为线程生命周期的开销和资源不足提供了解决方案，通过对多个任务重用线程，线程创建的开销别分摊到多个任务上。

线程池使用场景

- 单个任务处理时间很短
- 需要处理的任务数量很大。

使用线程池的好处

- 降低资源消耗，通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度，当任务到达时，任务可以不需要等到线程创建就能立即执行。
- 提高线程的可管理性，线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，挑用和监控。

java中的线程池利用ThreadPoolExecutor 类来实现的，接下来分析线程池的源码实现。



Executor 框架

Executor框架是一个根据一组执行策略调用、调度、执行和控制的异步任务的框架，目的是提供一种将任务提交与任务如何运行分离开来的机制。

- Executor： 一个运行新任务的简单接口
- ExecutorService: 扩展了Executor接口。添加了一些用来管理执行器生命周期和任务生命周期的方法；
- ScheduledExecutorService: 扩展了ExecutorService.支持Future和定期执行任务。

Executor接口

```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```



Executor接口只有一个execute方法，用来替代通常创建或启动线程的方法。例如，使用Thread来创建并启动线程的代码如下：

```java
Thread t = new Thread();
t.start();
```

使用Executor来启动线程执行任务的代码如下：

```java
Thread t = new Thread();
executor.execute(t);
```

对于不同的Executor实现，execute()方法可能是创建一个新线程并立即启动，也有可能是使用已有的工作线程来运行传入的任务，也可能是根据设置线程的容量或者阻塞队列的容量来决定是否要将传入的线程放入阻塞队列中或者拒绝接受传入的线程。

