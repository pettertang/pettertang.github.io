---
layout: post_layout
title: ThreadPool学习总结(一)
time: 2017年7月27日 星期四
location: 北京
pulished: true
excerpt_separator: "基于这个理解，我们可以"
---
### ThreadPool

我们知道，在程序中new一个新的thread是需要成本的。如果我们在项目开发中，每一个task我们都去new一个thread来处理，并且有很多小task需要处理，那产生thread的成本是相当高的。所以，比较好的解决办法就是产生一堆的threads，称之为thread pool，让这些开好的thread来处理这些小的task。  
Thread pool的三大元素是thread，task和queue。我们可以将thread pool理解成生产者消费者模式的一种形式，consumer就是一堆threads，当queue中有task进来，一个空闲的thread就会取出来处理。  
基于这个理解，我们可以自己实现一个最基本的thread pool，如下是一个简单的实现：
```Java
package com.talkingdata.utils.concurrent;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.LinkedBlockingQueue;

/**
 * Created by jianming.tang on 2017/7/19.
 */
public class TDThreadPool implements Runnable {

    private final LinkedBlockingQueue<Runnable> queue;
    private final List<Thread> threads;
    private boolean shutdown;

    public TDThreadPool(int countOfThreads) {
        queue = new LinkedBlockingQueue<>();
        threads = new ArrayList<>();
        for (int i = 0; i < countOfThreads; ++i) {
            Thread thread = new Thread(this);
            thread.start();
            threads.add(thread);
        }
    }

    public void execute(Runnable task) throws InterruptedException{
        queue.put(task);
    }

    private Runnable consume() throws InterruptedException{
        return queue.take();
    }

    @Override
    public void run() {
        try {
            while (!shutdown) {
                Runnable task = this.consume();
                task.run();
            }
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " shutdown");
    }

    public void shutdown() {
        shutdown = true;
        threads.forEach((thread) -> thread.interrupt());
    }

    public static void main(String[] args) throws InterruptedException{
        TDThreadPool threadPool = new TDThreadPool(5);
        Random random = new Random();
        for (int i = 0; i < 10; ++i) {
            int taskNumber = i;
            threadPool.execute(() -> {
                try {
                    Thread.sleep(random.nextInt(1000));
                    System.out.printf(Thread.currentThread().getName() +":" +"task %d complete\n", taskNumber);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        Thread.sleep(5000);
        threadPool.shutdown();
    }
}
```
上面这段代码的输出结果类似下面这样：
```
Thread-1: task 1 complete
Thread-0: task 0 complete
Thread-1: task 5 complete
Thread-1: task 7 complete
Thread-4: task 4 complete
Thread-4: task 9 complete
Thread-3: task 3 complete
Thread-1: task 8 complete
Thread-2: task 2 complete
Thread-0: task 6 complete
Thread-0 shutdown
Thread-3 shutdown
Thread-2 shutdown
Thread-1 shutdown
Thread-4 shutdown
```
产生了一个有5个thread的thread pool，queue里面放的是一个Runnable，代表一个可以执行的task。  
当然，上面这个只是基于线程池的概念实现的一个最基本的线程池，真正的线程池包含的东西要多的多，下面来介绍一下Java中已经给我们实现好的一些Thread Pool。  

### Java中的Basic Thread Pool  

**1**. 关于Java线程池  
每个线程池由几个模块组成：  
1.一个任务队列；  
2.一个工作线程的集合；  
3.一个线程工厂；  
4.管理线程状态的元数据。

顶层接口是**Executor**，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；  
然后**ExecutorService**接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；  
抽象类**AbstractExecutorService**实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；  
**ScheduledExecutorService**接口继承自ExecutorService接口；  
**ThreadPoolExecutor**继承了类AbstractExecutorService；  
**ScheduledThreadPoolExecutor**则继承自ThreadPoolExecutor和ScheduledExecutorService；   
**ForkJoinPool**继承了类AbstractExecutorService。  

---

**2**.向线程池提交任务：  
2.1可以使用execute提交任务，但是execute方法没有返回值，所以无法判断任务是否被线程池执行成功。通过以下代码可知execute方法输入的任务是一个Runnable类的实例。
```Java
threadPool.execute(() -> {
                try {
                    Thread.sleep(random.nextInt(1000));
                    System.out.printf(Thread.currentThread().getName() +":" +"task %d complete\n", taskNumber);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
```
2.2也可以使用submit 方法来提交任务，它会返回一个future,可以通过这个future来判断任务是否执行成功，通过future的get方法来获取返回值，get方法会阻塞住直到任务完成，而使用get(long timeout, TimeUnit unit)方法则会阻塞一段时间后立即返回，这时有可能任务没有执行完。
```Java
Future<Object> future = threadPoolExecutor.submit(hasReturnValueTask);
        try {
            Object s = future.get();
        } catch (InterruptedException e) {
            //处理中断异常
        } catch (ExecutionException e) {
            //处理无法执行任务异常
        } finally {
            //关闭线程池
            threadPoolExecutor.shutdown();
        }
```
---
2.3线程池的关闭：  
　　我们可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池，但是它们的实现原理不同，shutdown的原理是只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。shutdownNow的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。shutdownNow会首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。  
　　只要调用了这两个关闭方法的其中一个，isShutdown方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow。

---


**3**.几种常见的线程池如下：  
**newSingleThreadExecutor()**：创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。  
**newSingleThreadScheduledExecutor()**：创建一个单线程化的支持定时的线程池，可以用一个线程周期性执行任务。  
**newCachedThreadPool()**：创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。  
**newFixedThreadPool(int numberOfThreads)**：创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。  
**newScheduledThreadPool(int corePoolSize)**：创建一个定长线程池，支持定时（scheduleWithFixedDelay()函数的initdelay 参数）及周期（delay 参数）任务执行。

---

3.1应用场景：  
对于需要保证所有提交的任务都要被执行的情况，使用FixedThreadPool；  
如果限定只能使用一个线程进行任务处理，使用SingleThreadExecutor；  
如果希望提交的任务尽快分配线程执行，使用CachedThreadPool；  
如果希望定期执行某个任务,使用ScheduledThreadPool。  

---
3.2在Java中，以上这些thread pool都是通过Executors类来产生。   
**newFixedThreadPool**创建的线程池corePoolSize和maximumPoolSize值是相等的，它使用的LinkedBlockingQueue；
```Java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
        for (int i = 0;i < 10;++i) {
            final int index = i;
            fixedThreadPool.execute(() -> {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + index);
                    Thread.sleep(2000);
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
```
可能的输出如下：
```
pool-1-thread-1:0
pool-1-thread-2:1
pool-1-thread-3:2
pool-1-thread-1:3
pool-1-thread-2:4
pool-1-thread-3:5
pool-1-thread-3:6
pool-1-thread-1:7
pool-1-thread-2:8
pool-1-thread-1:9
```
因为线程池大小为3，每个任务输出index后sleep 2秒，所以每两秒打印3个数字。

---
**newSingleThreadExecutor**将corePoolSize和maximumPoolSize都设置为1，也使用的LinkedBlockingQueue；  
```Java
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        for (int i = 0;i < 10;++i) {
            final int index = i;
            singleThreadExecutor.execute(() -> {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + index);
                    Thread.sleep(3000);
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
```
输出如下：
```
pool-1-thread-1:0
pool-1-thread-1:1
pool-1-thread-1:2
pool-1-thread-1:3
pool-1-thread-1:4
pool-1-thread-1:5
pool-1-thread-1:6
pool-1-thread-1:7
pool-1-thread-1:8
pool-1-thread-1:9
```
结果依次输出，相当于顺序执行各个任务。

---
**newCachedThreadPool**将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。 
```Java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        for (int i = 0; i < 10; ++i) {
            final int index = i;
            try {
                Thread.sleep(index * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            cachedThreadPool.execute(() -> System.out.println(Thread.currentThread().getName() + ":" + index));
        }
```
输出如下：
```
pool-1-thread-1:0
pool-1-thread-1:1
pool-1-thread-1:2
pool-1-thread-1:3
pool-1-thread-1:4
pool-1-thread-1:5
pool-1-thread-1:6
pool-1-thread-1:7
pool-1-thread-1:8
pool-1-thread-1:9
```
线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

---
**newScheduledThreadPool**创建一个定长线程池，支持定时及周期性任务执行。
```Java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
        scheduledThreadPool.scheduleAtFixedRate(() -> System.out.println(Thread.currentThread().getName() + ":scheduledThreadPool"),1,3, TimeUnit.SECONDS);
```
可能的输出如下：
```
pool-1-thread-1:scheduledThreadPool
pool-1-thread-1:scheduledThreadPool
pool-1-thread-2:scheduledThreadPool
pool-1-thread-1:scheduledThreadPool
pool-1-thread-3:scheduledThreadPool
pool-1-thread-2:scheduledThreadPool
pool-1-thread-2:scheduledThreadPool
pool-1-thread-2:scheduledThreadPool
pool-1-thread-2:scheduledThreadPool
pool-1-thread-2:scheduledThreadPool
```
此例表示延迟3秒执行。

---
一般情况下以上这些JDK已经提供好的线程池可以满足我们的开发需求，如果ThreadPoolExecutor达不到要求，可以自己继承ThreadPoolExecutor类进行重写。

---

以上这五种线程池都直接或间接继承自ThreadPoolExcecutor线程池类。   
不难看出，ThreadPoolExecutor类是Java线程池中最核心的一个类，接下来也会重点介绍一下这个类。  
THreadPoolExecutor类中提供了四个构造方法：
```Java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```
观察上面这段代码，我们可以发现在创建一个线程池的时候，有5个参数是必须的：  
1.**corePoolSize**：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；  
2.**maximumPoolSize**：线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；  
3.**keepAliveTime**：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；   
4.**unit**：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：
```
TimeUnit.DAYS;              //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒
```
5.**workQueue**：一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择：
```
ArrayBlockingQueue;
PriorityBlockingQueue;
LinkedBlockingQueue;
SynchronousQueue;
```
另外还有两个参数说明如下：  
6.**threadFactory**：线程工厂，主要用来创建线程。  
7.**handler**：表示当拒绝处理任务时的策略，有以下四种取值：
```
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
```

---
### ForkJoinPool
从Java8开始，JDK又给我们提供了一种新的线程池：  
**newWorkStealingPool()**：创建一个拥有多个任务队列（以便减少连接数）的线程池。   
这是一种ForkJoinPool  
相较于一般的ThreadPool，大家共用一个queue，ForkJoinPool是每个Thread有一个自己的queue，而且是双向的queue(deque)。当一个task进来，它会把一部分的工作fork（切）出来先放到queue的最后面，另外一部分开始做。如果做进去后，发现task还是很大，就会继续切的更小，并放到queue的最后面。如此一边分割一边往下进行，直到task够小可以直接运算为止。当一个小的工作完成之后，它会从最后端拿出来（这里比较像栈的行为），继续往下做。然后很多完成的小的task的结果会进行join操作，join会取得每个subtask的结果，最终成为目前task的结果回传回去。  
当其他thread的queue为空时，它会去别的thread的queue的最前端取一个task到自己的queue中，之后的行为就和上面描述的一样。  
ForkJoinPool有一个很大的好处是减少thread因为blocking造成上下文切换，这样thread可以一直保持running的状态，一直到时间到了被上下文切换，而不是由于自己的阻塞造成的上下文切换。但是ForkJoinPool对于不可分割的task，并且处理时间差异很大的情景比较不适合。

---




  


