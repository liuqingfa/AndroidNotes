[TOC]


参考

周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践（第2版） (原创精品系列) . 机械工业出版社. Kindle 版本. 

----


## 1.为什么需要保持一致性

- 高速缓存
- 乱序执行优化
等

>**高速缓存:**  由于计算机的存储设备与处理器的运算速度有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高速缓存（Cache）来作为内存与处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，这样处理器就无须等待缓慢的内存读写了。
基于高速缓存的存储交互很好地解决了处理器与内存的速度矛盾，但是也为计算机系统带来更高的复杂度，因为它引入了一个新的问题：缓存一致性（CacheCoherence）。在多处理器系统中，每个处理器都有自己的高速缓存，而它们又共享同一主内存（MainMemory），
>
>**乱序执行:** 除了增加高速缓存之外，为了使得处理器内部的运算单元能尽量被充分利用，处理器可能会对输入代码进行乱序执行（Out-Of-OrderExecution）优化，处理器会在计算之后将乱序执行的结果重组，保证该结果与顺序执行的结果是一致的，但并不保证程序中各个语句计算的先后顺序与输入代码中的顺序一致，因此，如果存在一个计算任务依赖另外一个计算任务的中间结果，那么其顺序性并不能靠代码的先后顺序来保证。


## 2.Java内存模型

### 2.1 主内存和工作内存

参考 12.3.1

>Java内存模型规定了所有的变量（此处的变量（Variables）与Java编程中所说的变量有所区别，它包括了实例字段、静态字段和构成数组对象的元素，但不包括局部变量与方法参数，因为后者是线程私有的，不会被共享，自然就不会存在竞争问题。
）都存储在主内存（MainMemory）中（此处的主内存与介绍物理硬件时的主内存名字一样，两者也可以互相类比，但此处仅是虚拟机内存的一部分）。每条线程还有自己的工作内存（WorkingMemory，可与前面讲的处理器高速缓存类比），线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成，

### 2.2 内存间交互操作


>关于主内存与工作内存之间具体的交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步回主内存之类的实现细节，Java内存模型中定义了以下8种操作来完成，虚拟机实现时必须保证下面提及的每一种操作都是原子的、不可再分的

有一些关于下述8种操作的一些规则，详细请参考12.3.2

- lock(锁定)
 
  作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
  
  
- unlock

	作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read

	作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
- load（载入）

	作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use

	作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
- assign（赋值）

	作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store（存储）

	作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用
- write（写入）

	作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。
	

java内存模型只是要求上述两个操作是顺序执行的并不是连续执行的。比如对主内存中的a,b进行访问就可以出现这样的操作顺序：read a,read b, load b,load a。

### 2.3 原子性、可见性、有序性
 
此部分可参考 [《线程同步相关笔记》](https://github.com/sparkfengbo/AndroidNotes/blob/master/Android/%E7%BA%BF%E7%A8%8B%E4%B8%8E%E7%BA%BF%E7%A8%8B%E6%B1%A0/%E7%BA%BF%E7%A8%8B%E5%85%B3%E9%94%AE%E5%AD%97%E3%80%81%E9%94%81%E3%80%81%E5%90%8C%E6%AD%A5%E9%9B%86%E5%90%88.md)

#### 1.原子性（Atomicity）

	
>由Java内存模型来直接保证的原子性变量操作包括read、load、assign、use、store和write，我们大致可以认为基本数据类型的访问读写是具备原子性的（例外就是long和double的非原子性协定，读者只要知道这件事情就可以了，无须太过在意这些几乎不会发生的例外情况）。

#### 2.可见性（Visibility）	

>可见性是指当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。上文在讲解volatile变量的时候我们已详细讨论过这一点。
>
>Java内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方式来实现可见性的，无论是普通变量还是volatile变量都是如此，普通变量与volatile变量的区别是，volatile的特殊规则保证内存，以及每次使用前立即从主内存刷新。因此，可以说volatile保证了多线程操作时变量的可见性，而普通变量则不能保证这一点。
  
#### 3.有序性（Ordering）
	
>Java内存模型的有序性在前面讲解volatile时也详细地讨论过了，Java程序中天然的有序性可以总结为一句话：如果在本线程内观察，所有的操作都是有序的；如果在一个线程中观察另一个线程，所有的操作都是无序的。前半句是指“线程内表现为串行的语义”（Within-ThreadAs-If-SerialSemantics），后半句是指“指令重排序”现象和“工作内存与主内存同步延迟”现象。

### 2.4 volatile

>在某些情况下，volatile的同步机制的性能确实要优于锁（使用synchronized关键字或java.util.concurrent包里面的锁），但是由于虚拟机对锁实行的许多消除和优化，使得我们很难量化地认为volatile就会比synchronized快多少。
>
>....
>
>不过即便如此，大多数场景下volatile的总开销仍然要比锁低，


### 2.5 对long和double型变量的特殊规则

long和double是64位的数据类型，可能会进行两次32位的操作，参考 12.3.4 一般不会出现

### 2.6 先行发生原则




>先行发生是Java内存模型中定义的两项操作之间的偏序关系，如果说操作A先行发生于操作B，其实就是说在发生操作B之前，操作A产生的影响能被操作B观察到，“影响”包括修改了内存中共享变量的值、发送了消息、调用了方法等。
>
>
>
>下面是Java内存模型下一些“天然的”先行发生关系，这些先行发生关系无须任何同步器协助就已经存在，可以在编码中直接使用。如果两个操作之间的关系不在此列，并且无法从下列规则推导出来的话，它们就没有顺序性保障，虚拟机可以对它们随意地进行重排序。
>
>- 1.程序次序规则（ProgramOrderRule）：在一个线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作。准确地说，应该是控制流顺序而不是程序代码顺序，因为要考虑分支、循环等结构。
>- 2.管程锁定规则（MonitorLockRule）：一个unlock操作先行发生于后面对同一个锁的lock操作。这里必须
强调的是同一个锁，而“后面”是指时间上的先后顺序。
>- 3.volatile变量规则（VolatileVariableRule）：对一个volatile变量的写操作先行发生于后面对这个变量的读操作，这里的“后面”同样是指时间上的先后顺序。
>- 4.线程启动规则（ThreadStartRule）：Thread对象的start()方法先行发生于此线程的每一个动作。
>- 5.线程终止规则（ThreadTerminationRule）：线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值等手段检测到线程已经终止执行。
>- 6.线程中断规则（ThreadInterruptionRule）：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测到是否有中断发生。
>- 7.对象终结规则（FinalizerRule）：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。
>- 8.传递性（Transitivity）：如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。

参考 12.3.6的例子

>**时间先后顺序与先行发生原则之间基本没有太大的关系，所以我们衡量并发安全问题的时候不要受到时间顺序的干扰，一切必须以先行发生原则为准。**


### 2.4 Java线程的实现

#### 2.4.1 线程的实现

>实现线程主要有3种方式：
>
>- 使用内核线程实现
>
>- 使用用户线程实现
>
>- 使用用户线程加轻量级进程混合实现。
>
>内核线程（Kernel-LevelThread,KLT）就是直接由操作系统内核（Kernel，下称内核）支持的线程，这种线程由内核来完成线程切换，内核通过操纵调度器（Scheduler）对线程进行调度，并负责将线程的任务映射到各个处理器上。每个内核线程可以视为内核的一个分身，这样操作系统就有能力同时处理多件事情，支持多线程的内核就叫做多线程内核（Multi-ThreadsKernel）。
>
>...
>参考12.4.1

#### 2.4.2 Java 线程的实现

#### 2.4.3 Java 线程调度

>主要调度方式有两种，分别是协同式线程调度（CooperativeThreads-Scheduling）和抢占式线程调度（PreemptiveThreads-Scheduling）。


#### 2.4.4 Java 线程状态转换

>Java语言定义了5种线程状态，在任意一个时间点，一个线程只能有且只有其中的一种状态，这5种状态分别如下。
>
>- 1.新建（New）：创建后尚未启动的线程处于这种状态。
>
>- 2.运行（Runable）：Runable包括了操作系统线程状态中的Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着CPU为它分配执行时间。
>
>- 3.1 无限期等待（Waiting）：处于这种状态的线程不会被分配CPU执行时间，它们要等待被其他线程显式地唤醒。
>
>   以下方法会让线程陷入无限期的等待状态：
>   - 没有设置Timeout参数的Object.wait()方法。
>   - 没有设置Timeout参数的Thread.join()方法。
>   - LockSupport.park()方法。
> 
> - 3.2 限期等待（TimedWaiting）：处于这种状态的线程也不会被分配CPU执行时间，不过无须等待被其他线程显式地唤醒，在一定时间之后它们会由系统自动唤醒。以下方法会让线程进入限期等待状态：
>
>   - Thread.sleep()方法。
>   - 设置了Timeout参数的Object.wait()方法。
>   - 设置了Timeout参数的Thread.join()方法。
>   - LockSupport.parkNanos()方法。
>   - LockSupport.parkUntil()方法。
> 
> - 4.阻塞（Blocked）：线程被阻塞了，“阻塞状态”与“等待状态”的区别是：“阻塞状态”在等待着获取到一个排他锁，这个事件将在另外一个线程放弃这个锁的时候发生；而“等待状态”则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态。
> - 5.结束（Terminated）：已终止线程的线程状态，线程已经结束执行。上述5种状态在遇到特定事件发生的时候将会互相转换，它们的转换关系如图12-6所示。


#### 2.4.5 线程安全实现方法

**1.互斥同步**

>互斥同步（MutualExclusion＆Synchronization）是常见的一种并发正确性保障手段。同步是指在多个线程并发访问共享数据时，保证共享数据在同一个时刻只被一个（或者是一些，使用信号量的时候）线程使用。而互斥是实现同步的一种手段，临界区（CriticalSection）、互斥量（Mutex）和信号量（Semaphore）都是主要的互斥实现方式。因此，在这4个字里面，互斥是因，同步是果；互斥是方法，同步是目的。在Java中，最基本的互斥同步手段就是synchronized关键字.
>
>互斥同步最主要的问题就是进行线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步（BlockingSynchronization）。


**2.非阻塞同步**

>通俗地说，就是先进行操作，如果没有其他线程争用共享数据，那操作就成功了；如果共享数据有争用，产生了冲突，那就再采取其他的补偿措施（最常见的补偿措施就是不断地重试，直到成功为止），这种乐观的并发策略的许多实现都不需要把线程挂起，因此这种同步操作称为非阻塞同步
>
>为什么笔者说使用乐观并发策略需要“硬件指令集的发展”才能进行呢？因为我们需要操作和冲突检测这两个步骤具备原子性，靠什么来保证呢？如果这里再使用互斥同步来保证就失去意义了，所以我们只能靠硬件来完成这件事情，硬件保证一个从语义上看起来需要多次操作的行为只通过一条处理器指令就能完成，这类指令常用的有：
>
>- 测试并设置（Test-and-Set）。
>- 获取并增加（Fetch-and-Increment）。
>- 交换（Swap）。
>- 比较并交换（Compare-and-Swap，下文称CAS）。
>- 加载链接/条件存储（Load-Linked/Store-Conditional，下文称LL/SC）。

----

**CAS的介绍**

>CAS的实现需要硬件指令集的支撑,CAS比较交换的过程可以通俗的理解为CAS(V,O,N)，包含三个值分别为：V 内存地址存放的实际值；O 预期的值（旧值）；N 更新的新值。当V和O相同时，也就是说旧值和内存中实际的值相同表明该值没有被其他线程更改过，即该旧值O就是目前来说最新的值了，自然而然可以将新值N赋值给V。反之，V和O不相同，表明该值已经被其他线程改过了则该旧值O不是最新版本的值了，所以不能将新值N赋给V，返回V即可。当多个线程使用CAS操作一个变量时，只有一个线程会成功，并成功更新，其余会失败。失败的线程会重新尝试，当然也可以选择挂起线程
>
>**存在的问题**
>
>ABA、自旋时间过长、只能保证一个共享变量的原子操作
>
>- [深入浅出CAS](https://www.jianshu.com/p/fb6e91b013cc)
>- [让你彻底理解Synchronized](https://www.jianshu.com/p/d53bf830fa09)(包含了对CAS的介绍)

-----

**2.无同步方案**


>要保证线程安全，并不是一定就要进行同步，两者没有因果关系。同步只是保证共享数据争用时的正确性的手段，如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性，因此会有一些代码天生就是线程安全的，笔者简单地介绍其中的两类。
>
>可重入代码（Reentrant Code）：这种代码也叫做纯代码（PureCode），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。相对线程安全来说，可重入性是更基本的特性，它可以保证线程安全，即所有的可重入的代码都是线程安全的，但是并非所有的线程安全的代码都是可重入的。
>
>线程本地存储（Thread Local Storage）：如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行？如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。
>

---

**关于synchronized实现原理**

>synchronized关键字经过编译之后，会在同步块的前后分别形成monitorenter和monitorexit这两个字节码指令，这


两个字节码都需要一个reference类型的参数来指明要锁定和解锁的对象。如果Java程序中的synchronized明确指定了对象参数，那就是这个对象的reference；如果没有明确指定，那就根据synchronized修饰的是实例方法还是类方法，去取对应的对象实例或Class对象来作为锁对象。根据虚拟机规范的要求，在执行monitorenter指令时，首先要尝试获取对象的锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1，相应的，在执行monitorexit指令时会将锁计数器减1，当计数器为0时，锁就被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到对象锁被另外一个线程释放为止。


在虚拟机规范对monitorenter和monitorexit的行为描述中，有两点是需要特别注意的。首先，synchronized同步块对同一条线程来说是可重入的，不会出现自己把自己锁死的问题。其次，同步块在已进入的线程执行完之前，会阻塞后面其他线程的进入。

**关于ReentrantLock**

>在基本用法上，ReentrantLock与synchronized很相似，他们都具备一样的线程重入特性，只是代码写法上有点区别，一个表现为API层面的互斥锁（lock()和unlock()方法配合try/finally语句块来完成），另一个表现为原生语法层面的互斥锁。不过，相比synchronized,ReentrantLock增加了一些高级功能，
>主要有以下3项：
>
>- 等待可中断
>- 可实现公平锁
>- 锁可以绑定多个条件
>
>等待可中断是指当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情，可中断特性对处理执行时间非常长的同步块很有帮助。
>
>公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁；而非公平锁则不保证这一点，在锁被释放时，任何一个等待锁的线程都有机会获得锁。synchronized中的锁是非公平的，ReentrantLock默认情况下也是非公平
的，但可以通过带布尔值的构造函数要求使用公平锁。
>
>锁绑定多个条件是指一个ReentrantLock对象可以同时绑定多个Condition对象，而在synchronized中，锁对象的wait()和notify()或notifyAll()方法可以实现一个隐含的条件，如果要和多于一个的条件关联的时候，就不得不额外地添加一个锁，而ReentrantLock则无须这样做，只需要多次调用newCondition()方法即可。
>
>多线程环境下synchronized的吞吐量下降得非常严重，而ReentrantLock则能基本保持在同一个比较稳定的水平上。与其说ReentrantLock性能好，还不如说synchronized还有非常大的优化余地。措施），JDK1.6发布之后，人们就发现synchronized与ReentrantLock的性能基本上是完全持平了。
>

### 2.5 锁优化

>- 适应性自旋（AdaptiveSpinning
>- 锁消除（LockElimination）
>- 锁粗化（LockCoarsening）
>- 轻量级锁（LightweightLocking）
>- 偏向锁（BiasedLocking）


>前面我们讨论互斥同步的时候，提到了互斥同步对性能最大的影响是阻塞的实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性能带来了很大的压力。同时，虚拟机的开发团队也注意到在许多应用上，共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复线程并不值得。如果物理机器有一个以上的处理器，能让两个或以上的线程同时并行执行，我们就可以让后面请求锁的那个线程“稍等一下”，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁。


参考13.3

