# 核心知识

## 1. 线程转换状态

![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/ace830df-9919-48ca-91b5-60b193f593d2-1.png)

![](https://gitee.com/dukangming/PicBedGitee/raw/master/img/1536767960941.png)

1. **新建（New）**

   创建后尚未启动

2. **可运行（Runnable）**

   可能正在运行，也可能正在等待 CPU 时间片。

   包含了操作系统线程状态中的 运行（Running ） 和 就绪（Ready）。

3. **阻塞（Blocking）**

   这个状态下，是在多个线程有同步操作的场景，比如正在等待另一个线程的 synchronized 块的执行释放，或者可重入的 synchronized 块里别人调用 wait() 方法，也就是线程在等待进入临界区。 

   阻塞可以分为：等待阻塞，同步阻塞，其他阻塞

4. **无限等待期（Waiting）**

   等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

   | 进入方法                                   | 退出方法                             |
   | ------------------------------------------ | ------------------------------------ |
   | 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
   | 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
   | LockSupport.park() 方法                    | -                                    |

5. **限期等待（Timed Waiting）**

   无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

   调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用 “**使一个线程睡眠**” 进行描述。

   调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用 “**挂起一个线程**” 进行描述。

   **睡眠和挂起**是用来描述**行为**，而**阻塞**和等待用来描述**状态**。

   阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

   | 进入方法                                 | 退出方法                                        |
   | ---------------------------------------- | ----------------------------------------------- |
   | Thread.sleep() 方法                      | 时间结束                                        |
   | 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
   | 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
   | LockSupport.parkNanos() 方法             | -                                               |
   | LockSupport.parkUntil() 方法             | -                                               |

6. **死亡（Terminated）**

   - 线程因为 run 方法正常退出而自然死亡
   - 因为一个没有捕获的异常终止了 run 方法而意外死亡

## 2. 多线程的Java实现

有三种使用线程的方法：

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

### 2.1 实现 Runnable 接口

需要实现 run() 方法。

通过 Thread 调用 start() 方法来启动线程。

``` java
public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
}
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
``` 

### 2.2 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

``` java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
``` 

### 2.3 继承 Thread 类

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

``` java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
``` 

### 2.4 实现接口 VS 继承 Thread 

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

### 2.5 三种区别

- 实现 Runnable 接口可以避免 Java 单继承特性而带来的局限；增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的；适合多个相同程序代码的线程区处理同一资源的情况。 
- 继承 Thread 类和实现 Runnable 方法启动线程都是使用 start() 方法，然后 JVM 虚拟机将此线程放到就绪队列中，如果有处理机可用，则执行 run() 方法。 
- 实现 Callable 接口要实现 call() 方法，并且线程执行完毕后会有返回值。其他的两种都是重写 run() 方法，没有返回值。 

## 3. 基础线程机制

### 3.1 Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

- CachedThreadPool：一个任务创建一个线程：

  创建一个可缓存的线程池。

  - 如果线程池的规模超过了处理需求，将自动回收空闲线程。
  - 当需求增加时，则可以自动添加新线程。线程池的规模不存在任何限制。

- FixedThreadPool：所有任务只能使用固定大小的线程：

  创建一个固定长度的线程池。

  - 每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程规模将不再变化。
  - 当线程发生未预期的错误而结束时，线程池会补充一个新的线程。

- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool：

  创建一个单线程的线程池。

  - 它创建单个工作线程来执行任务，如果这个线程异常结束，会创建一个新的来替代它。
  - 它的特点是，能确保依照任务在队列中的顺序来串行执行。

``` java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
``` 

**为什么引入Executor线程池框架？**

new Thread() 的缺点

- 每次 new Thread() 耗费性能 
- 调用 new Thread() 创建的线程缺乏管理，被称为野线程，而且可以无限制创建，之间相互竞争，会导致过多占用系统资源导致系统瘫痪。 
- 不利于扩展，比如如定时执行、定期执行、线程中断

采用线程池的优点

- 重用存在的线程，减少对象创建、消亡的开销，性能佳 
- 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞 
- 提供定时执行、定期执行、单线程、并发数控制等功能

### 3.2 Daemon

Java 中有两类线程：User Thread (用户线程)、Daemon Thread (守护线程)

用户线程即运行在前台的线程，而守护线程是运行在后台的线程。  守护线程作用是为其他前台线程的运行提供便利服务，而且仅在普通、非守护线程仍然运行时才需要，比如垃圾回收线程就是一个守护线程。当 JVM  检测仅剩一个守护线程，而用户线程都已经退出运行时，JVM  就会退出，因为没有如果没有了被守护这，也就没有继续运行程序的必要了。如果有非守护线程仍然存活，JVM 就不会退出。

守护线程并非只有虚拟机内部提供，用户在编写程序时也可以自己设置守护线程。用户可以用 Thread 的 setDaemon(true) 方法设置当前线程为守护线程。

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

使用 setDaemon() 方法将一个线程设置为守护线程。

``` java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
``` 

### 3.3 sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

``` java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
``` 

### 3.4 yield()

对静态方法 Thread.yield() 的调用声明了**当前线程已经完成了生命周期中最重要的部分**，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

``` java
public void run() {
    Thread.yield();
}
``` 

### 3.5 线程阻塞

线程可以阻塞于四种状态：

- 当线程执行 Thread.sleep() 时，它一直阻塞到指定的毫秒时间之后，或者阻塞被另一个线程打断；
- 当线程碰到一条 wait() 语句时，它会一直阻塞到接到通知 notify()、被中断或经过了指定毫秒时间为止（若制定了超时值的话）
- 线程阻塞与不同 I/O 的方式有多种。常见的一种方式是 InputStream 的 read() 方法，该方法一直阻塞到从流中读取一个字节的数据为止，它可以无限阻塞，因此不能指定超时时间；
- 线程也可以阻塞等待获取某个对象锁的排他性访问权限（即等待获得 synchronized 语句必须的锁时阻塞）。

> 注意，并非所有的阻塞状态都是可中断的，以上阻塞状态的前两种可以被中断，后两种不会对中断做出反应

















参考：

- https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%B9%B6%E5%8F%91.md
- https://github.com/frank-lam/fullstack-tutorial/blob/master/notes/JavaArchitecture/03-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B.md
- 



