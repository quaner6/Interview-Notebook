<!-- GFM-TOC -->
* [使用线程](#使用线程)
    * [1. 实现 Runnable 接口](#1-实现-runnable-接口)
    * [2. 实现 Callable 接口](#2-实现-callable-接口)
    * [3. 继承 Thread 类](#3-继承-thread-类)
    * [4. 实现接口 vs 继承 Thread](#4-实现接口-vs-继承-thread)
* [Executor](#executor)
* [基础线程机制](#基础线程机制)
    * [1. sleep()](#1-sleep)
    * [2. yield()](#2-yield)
    * [3. join()](#3-join)
    * [4. deamon](#4-deamon)
* [线程之间的协作](#线程之间的协作)
    * [1. 线程通信](#1-线程通信)
    * [2. 线程同步](#2-线程同步)
        * [2.1 synchronized](#21-synchronized)
        * [2.2 Lock](#22-lock)
        * [2.3 BlockingQueue](#23-blockingqueue)
* [结束线程](#结束线程)
    * [1. 阻塞](#1-阻塞)
    * [2. 中断](#2-中断)
* [线程状态转换](#线程状态转换)
* [Java 内存模型与线程](#java-内存模型与线程)
    * [1. 硬件的效率与一致性](#1-硬件的效率与一致性)
    * [2. Java 内存模型](#2-java-内存模型)
        * [2.1 主内存与工作内存](#21-主内存与工作内存)
        * [2.2 内存间交互操作](#22-内存间交互操作)
        * [2.3 内存模型三大特性](#23-内存模型三大特性)
            * [2.3.1 原子性](#231-原子性)
            * [2.3.2 可见性](#232-可见性)
            * [2.3.3 有序性](#233-有序性)
    * [3. 未完待续](#3-未完待续)
* [多线程开发良好的实践](#多线程开发良好的实践)
* [参考资料](#参考资料)
<!-- GFM-TOC -->


# 使用线程

有三种使用线程的方法：

1. 实现 Runnable 接口；
2. 实现 Callable 接口；
3. 继承 Thread 类；

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

## 1. 实现 Runnable 接口

需要实现 run() 方法。

通过 Thread 调用 start() 方法来启动线程。

```java
public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
    public static void main(String[] args) {
        MyRunnable instance = new MyRunnable();
        Tread thread = new Thread(instance);
        thread.start();
    }
}
```

## 2. 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```java
public  class  MyCallable  implements  Callable<Integer> {
    public Integer call() {
        // ...
    }
    public  static  void  main(String[]  args) {
        MyCallable mc = new MyCallable();
        FutureTask<Integer> ft = new FutureTask<>(mc);
        Thread thread = new Thread(ft);
        thread.start();
        System.out.println(ft.get());
    }
}
```

## 3. 继承 Thread 类

同样也是需要实现 run() 方法，并且最后也是调用 start() 方法来启动线程。

```java
class MyThread extends Thread {
    public void run() {
        // ...
    }
    public  static  void  main(String[]  args) {
        MyThread mt = new MyThread();
        mt.start();
    }
}
```

## 4. 实现接口 vs 继承 Thread

实现接口会更好一些，因为：

1. Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口。
2. 类可能只要求可执行即可，继承整个 Thread 类开销会过大。

# Executor

Executor 管理多个异步任务的执行，而无需程序员显示地管理线程的生命周期。

主要有三种 Excutor：

1. CachedTreadPool：一个任务创建一个线程；
2. FixedThreadPool：所有任务只能使用固定大小的线程；
3. SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

```java
ExecutorService exec = Executors.newCachedThreadPool();
for(int i = 0; i < 5; i++) {
    exec.execute(new MyRunnable());
}
```

# 基础线程机制

## 1. sleep()

**Thread.sleep(millisec)**  方法会休眠当前正在执行的线程，millisec 单位为毫秒。也可以使用 TimeUnit.TILLISECONDS.sleep(millisec)。

sleep() 可能会抛出 InterruptedException。因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

```java
public void run() {
    try {
        // ...
        Thread.sleep(1000);
        // ...
    } catch(InterruptedException e) {
        System.err.println(e);
    }
}
```

## 2. yield()

对静态方法  **Thread.yield()**  的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。

```java
public void run() {
    // ...
    Thread.yield();
}
```

## 3. join()

在线程中调用另一个线程的  **join()**  方法，会将当前线程挂起，直到目标线程结束。

可以加一个超时参数。

## 4. deamon

后台线程（ **deamon** ）是程序运行时在后台提供服务的线程，并不属于程序中不可或缺的部分。

当所有非后台线程结束时，程序也就终止，同时会杀死所有后台线程。

main() 属于非后台线程。

使用 setDaemon() 方法将一个线程设置为后台线程。

# 线程之间的协作

-  **线程通信** ：保证线程以一定的顺序执行；
-  **线程同步** ：保证线程对临界资源的互斥访问。

线程通信往往是基于线程同步的基础上完成的，因此很多线程通信问题也是线程同步问题。

## 1. 线程通信

**wait()、notify() 和 notifyAll()**  三者实现了线程之间的通信。

wait() 会在等待时将线程挂起，而不是忙等待，并且只有在 notify() 或者 notifyAll() 到达时才唤醒。

sleep() 和 yield() 并没有释放锁，但是 wait() 会释放锁。实际上，只有在同步控制方法或同步控制块里才能调用 wait() 、notify() 和 notifyAll()。

这几个方法属于基类的一部分，而不属于 Thread。

```java
private boolean flag = false;

public synchronized void after() {
    while(flag == false) {
        wait();
        // ...
    }
}

public synchronized void before() {
    flag = true;
    notifyAll();
}
```

**wait() 和 sleep() 的区别** 

1. wait() 是 Object 类的方法，而 sleep() 是 Thread 的静态方法；
2. wait() 会放弃锁，而 sleep() 不会。

## 2. 线程同步

给定一个进程内的所有线程，都共享同一存储空间，这样有好处又有坏处。这些线程就可以共享数据，非常有用。不过，在两个线程同时修改某一资源时，这也会造成一些问题。Java 提供了同步机制，以控制对共享资源的互斥访问。

### 2.1 synchronized

**同步一个方法** 

使多个线程不能同时访问该方法。

```java
public synchronized void func(String name) {
    // ...
}
```

**同步一个代码块** 

```java
public void func(String name) {
    synchronized(this) {
        // ...
    }
}
```

### 2.2 Lock

若要实现更细粒度的控制，我们可以使用锁（lock）。

```java
private Lock lock;
public int func(int value) {
    lock.lock();
    // ...
    lock.unlock();
}
```

### 2.3 BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

-  **FIFO 队列** ：LinkedBlockingQueue、ArrayListBlockingQueue（固定长度）
-  **优先级队列** ：PriorityBlockingQueue

提供了阻塞的 take() 和 put() 方法：如果队列为空 take() 将一直阻塞到队列中有内容，如果队列为满  put() 将阻塞到队列有空闲位置。它们响应中断，当收到中断请求的时候会抛出 InterruptedException，从而提前结束阻塞状态。

**使用 BlockingQueue 实现生产者消费者问题** 

```java
// 生产者
import java.util.concurrent.BlockingQueue;

public class Producer implements Runnable {
    private BlockingQueue<String> queue;

    public Producer(BlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " is making product...");
        String product = "made by " + Thread.currentThread().getName();
        try {
            queue.put(product);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
// 消费者
import java.util.concurrent.BlockingQueue;

public class Consumer implements Runnable{
    private BlockingQueue<String> queue;

    public Consumer(BlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            String  product = queue.take();
            System.out.println(Thread.currentThread().getName() + " is consuming product " + product + "...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
// 客户端
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class Client {
    public static void main(String[] args) {
        BlockingQueue<String> queue = new LinkedBlockingQueue<>(5);
        for (int i = 0; i < 2; i++) {
            new Thread(new Consumer(queue), "Producer" + i).start();
        }
        for (int i = 0; i < 5; i++) {
            // 只有两个 Product，因此只能消费两个，其它三个消费者被阻塞
            new Thread(new Producer(queue), "Consumer" + i).start();
        }
        for (int i = 2; i < 5; i++) {
            new Thread(new Consumer(queue), "Producer" + i).start();
        }
    }
}
```

```html
// 运行结果
Consumer0 is making product...
Producer0 is consuming product made by Consumer0...
Consumer1 is making product...
Producer1 is consuming product made by Consumer1...
Consumer2 is making product...
Consumer3 is making product...
Consumer4 is making product...
Producer2 is consuming product made by Consumer2...
Producer3 is consuming product made by Consumer3...
Producer4 is consuming product made by Consumer4...
```

# 结束线程

## 1. 阻塞

一个线程进入阻塞状态可能有以下原因：

1. 调用 Thread.sleep() 方法进入休眠状态；
2. 通过 wait() 使线程挂起，直到线程得到 notify() 或 notifyAll() 消息（或者 java.util.concurrent 类库中等价的 signal() 或 signalAll() 消息；
3. 等待某个 I/O 的完成；
4. 试图在某个对象上调用其同步控制方法，但是对象锁不可用，因为另一个线程已经获得了这个锁。

## 2. 中断

使用中断机制即可终止阻塞的线程。

使用  **interrupt()**  方法来中断某个线程，它会设置线程的中断状态。Object.wait(), Thread.join() 和 Thread.sleep() 三种方法在收到中断请求的时候会清除中断状态，并抛出 InterruptedException。

应当捕获这个 InterruptedException 异常，从而做一些清理资源的操作。

**不可中断的阻塞** 

不能中断 I/O 阻塞和 synchronized 锁阻塞。

**Executor 的中断操作** 

Executor 避免对 Thread 对象的直接操作，但是使用 interrupt() 方法必须持有 Thread 对象。Executor 使用 shutdownNow() 方法来中断所有它里面的所有线程，shutdownNow() 方法会发送 interrupt() 调用给所有线程。

如果只想中断一个线程，那么使用 Executor 的 submit() 而不是 executor() 来启动线程，就可以持有线程的上下文。submit() 将返回一个泛型 Futrue，可以在它之上调用 cancel()，如果将 true 传递给 cancel()，那么它将会发送 interrupt() 调用给特定的线程。

**检查中断** 

通过中断的方法来终止线程，需要线程进入阻塞状态才能终止。如果编写的 run() 方法循环条件为 true，但是该线程不发生阻塞，那么线程就永远无法终止。

interrupt() 方法会设置中断状态，可以通过 interrupted() 方法来检查中断状，从而判断一个线程是否已经被中断。

interrupted() 方法在检查完中断状态之后会清除中断状态，这样做是为了确保一次中断操作只会产生一次影响。

# 线程状态转换

<div align="center"> <img src="../pics//38b894a7-525e-4204-80de-ecc1acc52c46.jpg"/> </div><br>

1. NEW（新建）：创建后尚未启动的线程。
2. RUNNABLE（运行）：处于此状态的线程有可能正在执行，也有可能正在等待着 CPU 为它分配执行时间。
3. BLOCKED（阻塞）：阻塞与等待的区别是，阻塞在等待着获取到一个排它锁，这个时间将在另一个线程放弃这个锁的时候发生；而等待则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态。
4. Waiting（无限期等待）：处于这种状态的进行不会被分配 CPU 执行时间，它们要等待其它线程显示地唤醒。以下方法会让线程进入这种状态：
5. TIMED_WAITING（限期等待）：处于这种状态的线程也不会被分配 CPU 执行时间，不过无序等待其它线程显示地唤醒，在一定时间之后它们会由系统自动唤醒。
6. TERMINATED（死亡）

以下方法会让线程陷入无限期的等待状态：

- 没有设置 Timeout 参数的 Object.wait() 方法
- 没有设置 Timeout 参数的 Thread.join() 方法
- LockSupport.park() 方法

以下方法会让线程进入限期等待状体：

- Thread.sleep()
- 设置了 Timeout 参数的 Object.wait() 方法
- 设置了 Timeout 参数的 Thread.join() 方法
- LockSupport.parkNanos() 方法
- LockSupport.parkUntil() 方法

# Java 内存模型与线程

## 1. 硬件的效率与一致性

对处理器上的寄存器进行读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

每个处理器都有一个高速缓存，但是所有处理器共用一个主内存，因此高速缓存引入了一个新问题：缓存一致性。当多个处理器的运算都涉及同一块主内存区域时，将可能导致各自的缓存数据不一致。缓存不一致问题通常需要使用一些协议来解决。

<div align="center"> <img src="../pics//352dd00d-d1bb-4134-845d-16a75bcb0e02.jpg"/> </div><br>

除了增加高速缓存之外，为了使得处理器内部的运算单元能尽量被充分利用，处理器可能会对输入代码进行乱序执行（Out-Of-Order Execution）优化，处理器会在计算之后将乱序执行的结果重组，保证该结果与顺序执行的结果是一致的，但并不保证程序中各个语句计算的先后顺序与输入代码中的顺序一致，因此，如果存在一个计算任务依赖另外一个计算任务的中间结果，那么其顺序性并不能靠代码的先后顺序来保证。与处理器的乱序执行优化类似，Java 虚拟机的即时编译器中也有类似的指令重排序（Instruction Reorder）优化。

## 2. Java 内存模型

Java 虚拟机规范中试图定义一种 Java 内存模型来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。在此之前，主流程序语言（如 C/C++等）直接使用物理硬件和操作系统的内存模型，因此，会由于不同平台上内存模型的差异，有可能导致程序在一套平台上并发完全正常，而在另外一套平台上并发访问却经常出错，因此在某些场景就必须针对不同的平台来编写程序。

### 2.1 主内存与工作内存

Java 内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。此处的变量（Variables）与 Java 编程中所说的变量有所区别，它包括了实例字段、静态字段和构成数组对象的元素，但不包括局部变量与方法参数，因为后者是线程私有的，不会被共享，自然就不会存在竞争问题。

Java 内存模型规定了所有的变量都存储在主内存（Main Memory）中。每条线程还有自己的工作内存，线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成，线程、主内存、工作内存三者的交互关系如图所示。

<div align="center"> <img src="../pics//b02a5492-5dcf-4a69-9b5b-c2298b2cb81c.jpg"/> </div><br>

### 2.2 内存间交互操作

Java 内存模型定义了 8 种操作来完成工作内存与主内存之间的交互：一个变量从主内存拷贝到工作内存、从工作内存同步回主内存。虚拟机实现时必须保证下面提及的每一种操作都是原子的、不可再分的。

- lock（锁定）：作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
- unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的 load 动作使用。
- load（载入）：作用于工作内存的变量，它把 read 操作从主内存中得到的变量值放入工作内存的变量副本中。
- use（使用）：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
- assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的 write 操作使用。
- write（写入）：作用于主内存的变量，它把 store 操作从工作内存中得到的变量的值放入主内存的变量中。

### 2.3 内存模型三大特性

#### 2.3.1 原子性

除了 long 和 double 之外的基本数据类型的访问读写是具备原子性的。

Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据的读写操作划分为两次 32 位的操作来进行，即虚拟机可以不保证 64 位数据类型的 load、store、read 和 write 这 4 个操作的原子性。但是目前各种平台下的商用虚拟机几乎都选择把 64 位数据的读写操作作为原子操作来对待。

**AtomicInteger、AtomicLong、AtomicReference**  等特殊的原子性变量类提供了下面形式的原子性条件更新语句，使得比较和更新这两个操作能够不可分割地执行。

```java
boolean compareAndSet(expectedValue, updateValue);
```

AtomicInteger 使用举例：

```java
private AtomicInteger ai = new AtomicInteger(0);

public int next() {
    return ai.addAndGet(2)
}
```

如果应用场景需要一个更大范围的原子性保证，Java 内存模型还提供了 lock 和 unlock 操作来满足这种需求，尽管虚拟机未把 lock 和 unlock 操作直接开放给用户使用，但是却提供了更高层次的字节码指令 monitorenter 和 monitorexit 来隐式地使用这两个操作，这两个字节码指令反映到 Java 代码中就是同步块——synchronized 关键字，因此在 synchronized 块之间的操作也具备原子性。

#### 2.3.2 可见性

可见性是指当一个线程修改了共享变量的值，其他线程能立即得知这个修改。

Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方式来实现可见性的，无论是普通变量还是 volatile 变量都是如此，普通变量与 volatile 变量的区别是，volatile 的特殊规则保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。因此，可以说 volatile 保证了多线程操作时变量的可见性，而普通变量则不能保证这一点。

除了 volatile 之外，Java 还有两个关键字能实现可见性，即 synchronized 和 final。同步块的可见性是由“对一个变量执行 unlock 操作之前，必须先把此变量同步回主内存中（执行 store、write 操作）”这条规则获得的，而 final 关键字的可见性是指：被 final 修饰的字段在构造器中一旦初始化完成，并且构造器没有把“this”的引用传递出去（this 引用逃逸是一件很危险的事情，其他线程有可能通过这个引用访问到“初始化了一半”的对象），那在其他线程中就能看见 final 字段的值。

#### 2.3.3 有序性

本线程内观察，所有的操作都是有序的；如果在一个线程中观察另一个线程，所有的操作都是无序的。前半句是指线程内表现为串行的语义，后半句是指指令重排和工作内存和主内存存在同步延迟的现象。

Java 语言提供了 volatile 和 synchronized 两个关键字来保证线程之间操作的有序性，volatile 关键字本身就包含了禁止指令重排序的语义，而 synchronized 则是由“一个变量在同一个时刻只允许一条线程对其进行 lock 操作”这条规则获得的，这条规则决定了持有同一个锁的两个同步块只能串行地进入。

synchronized 关键字在需要这 3 种特性的时候都可以作为其中一种的解决方案，看起来很“万能”。的确，大部分的并发控制操作都能使用 synchronized 来完成。synchronized 的“万能”也间接造就了它被程序员滥用的局面，越“万能”的并发控制，通常会伴随着越大的性能影响。

## 3. 未完待续

# 多线程开发良好的实践

- 给线程命名；
- 最小化同步范围；
- 优先使用 volatile；
- 尽可能使用更高层次的并发工具而非 wait 和 notify() 来实现线程通信，如 BlockingQueue, Semeaphore；
- 多用并发容器，少用同步容器，并发容器比同步容器的可扩展性更好。
- 考虑使用线程池
- 最低限度的使用同步和锁，缩小临界区。因此相对于同步方法，同步块会更好。

# 参考资料

- Java 编程思想
- 深入理解 Java 虚拟机
- [Java 线程面试题 Top 50](http://www.importnew.com/12773.html)
- [Java 面试专题 - 多线程 & 并发编程 ](https://www.jianshu.com/p/e0c8d3dced8a)
- [可重入内置锁](https://github.com/francistao/LearningNotes/blob/master/Part2/JavaConcurrent/%E5%8F%AF%E9%87%8D%E5%85%A5%E5%86%85%E7%BD%AE%E9%94%81.md)
