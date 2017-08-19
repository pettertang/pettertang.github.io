---
layout: post_layout
title: TD-Summary for week 10
time: 2017年8月10日 星期四
location: 北京
pulished: true
excerpt_separator: "其中，refreshToken()这个方法"
---

### 第10周总结

**1**.关于synchronized使用的一点说明。  
在多线程并发情况下，我们在使用synchronized时会因为对synchronized的理解不透彻而出错。假如有如下一个场景：  
假设现在有10个线程，thread1 ~ thread10，它们并发的访问如下方法：
```Java
private synchronized Integer refreshToken(int token) {
        //your code here
        return token;
    }
```
其中，refreshToken()这个方法简单说明下：这个方法用来检验当前时刻的token值是否失效（针对不同的用户，系统会分配给它们一个唯一的token值，用来鉴权，因为不同的用户对应不同的角色，会有不同的权限。同时为了安全，token肯定会有时效性，这个依据具体业务来定。），如果失效，则要相应的模块重新获取一遍token值。   
假设在某个时刻，设为t1时刻，线程thread1 ~ thread5同时访问上面这个方法，线程thread1竞争到了这个锁，则线程thread2 ~ thread5会阻塞在这里。假设此时token已经失效，线程thread1会访问授权模块重新获得token，并对token值进行更新，由于synchronized的实现机制，它只会将这个更新告知在t1时刻同时参与竞争这个锁的thread2 ~ thread5线程，而thread6 ~ thread10因为没有参与竞争，所以它们不会知道token值已经更新了。这样就会造成一个问题，那就事过了一会儿到t2时刻，当thread6 ~ thread10需要这个token值时，它们手中的token将还是已经过期的token值，这样这些线程还得重新去访问授权模块获取token值。但是实际上，对于同一个用户，在token失效期内，token值是不变的，所以这些线程重新获取到的token值和thread1之前重新取到的值是一样的，这样就相当于做了无用功，浪费时间和资源。  
遇到这种情况， 我们可以用ReentrantLock来替代synchronized，或者自己实现合理的同步机制。

---
**2**.CountDownLatch简介  
　　CountDownLatch是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。  
　　CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。  
　　CountDownLatch.java类中定义的构造函数：
```Java
public void CountDownLatch(int count) {...}
```
构造器中的计数值（count）实际上就是**闭锁需要等待的线程数量**。这个值**只能被设置一次**，而且CountDownLatch**没有提供任何机制去重新设置这个计数值**。与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用**CountDownLatch.await()方法**。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。  
其他N 个线程必须引用**闭锁对象**，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过**CountDownLatch.countDown()方法**来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。  
　　在实时系统中的应用场景：  
1.实现最大的并行性：有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数为1的CountDownLatch，并让所有线程都在这个锁上等待，那么我们可以很轻松地完成测试。我们只需调用 一次countDown()方法就可以让所有的等待线程同时恢复执行。  
2.开始执行前等待n个线程完成各自任务：例如应用程序启动类要确保在处理用户请求前，所有N个外部系统已经启动和运行了。  
3.死锁检测：一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。  
主要的方法如下：  
public CountDownLatch(int count);  
public void countDown();  
public void await() throws InterruptedException  
说明：  
构造方法参数指定了计数的次数，决定CountDownLatch类需要等待的事件的数量；  
countDown方法，当前线程调用此方法，则计数减一；  
await方法，调用此方法会一直阻塞当前线程，直到计时器的值为0。  


---
**3**.关于join和CountDownLatch  
上面提到了CountDownLatch，那就顺便说一下join，并对两者做一个简单的比较说明。  
1）考虑如下场景一：假设一条流水线上有三个工作者：worker0，worker1，worker2。有一个任务的完成需要他们三者协作完成，worker2可以开始这个任务的前提是worker0和worker1完成了他们的工作，而worker0和worker1是可以并行他们各自的工作的。  
解决办法：上面这个场景很容易想到用join来做。当在当前线程中调用某个线程 thread 的 join() 方法时，当前线程就会阻塞，直到thread 执行完成，当前线程才可以继续往下执行。所以我们可以在woker0和worker1启动后，调用它们的join方法，然后在它们之后再调用worker2的start方法，则可以满足要求。  
补充说明下join的工作原理：不停检查thread是否存活，如果存活则让当前线程永远wait，直到thread线程终止，线程的this.notifyAll 就会被调用。  
2）考虑如下场景二：假设worker的工作可以分为两个阶段，work2 只需要等待work0和work1完成他们各自工作的第一个阶段之后就可以开始自己的工作了，而不是场景1中的必须等待work0和work1把他们的工作全部完成之后才能开始。  
在这种情况下，join是没办法实现这个场景的，而CountDownLatch却可以，因为它持有一个计数器，只要计数器为0，那么主线程就可以结束阻塞往下执行。我们可以在worker0和worker1完成第一阶段工作之后就把计数器减1即可，这样worker0和worker1在完成第一阶段工作之后，worker2就可以开始工作了。  
　　总结下CountDownLatch与join的区别：调用thread.join() 方法必须等thread 执行完毕，当前线程才能继续往下执行；而CountDownLatch通过计数器提供了更灵活的控制，只要检测到计数器为0当前线程就可以往下执行而不用管相应的thread是否执行完毕。
