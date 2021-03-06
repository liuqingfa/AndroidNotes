
**参考**

周志明. 深入理解Java虚拟机：JVM高级特性与最佳实践（第2版） (原创精品系列) (Kindle位置980). 机械工业出版社. Kindle 版本. 

- [JVM 内存模型概述](http://www.wanandroid.com/blog/show/2152)

-----

## 1.内存划分

Java虚拟机在执行Java程序的过程中会将所管理的内存划分为若干个不同的数据区域。

![Java虚拟机运行时数据区](https://github.com/sparkfengbo/AndroidNotes/blob/master/PictureRes/JVM/Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.png?raw=true)

- 1.方法区
- 2.堆
- 3.虚拟机栈
- 4.本地方法栈
- 5.程序计数器

### 1.1 Java堆

对大多数应用来说，Java堆是Java虚拟机所管理的内存中最大的一块

Java堆是**被所有线程共享的一块内存区域**，在虚拟机启动时创建。

使用`-Xmx`和`-Xms`控制大小

通过参数 `-XX:+ HeapDumpOnOutOfMemoryError` 可以让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便事后进行分析


**作用**

>此内存区域的唯一**目的**就是存放对象实例，几乎所有的对象实例都在这里分配内存。这一点在Java虚拟机规范中的描述是：所有的对象实例以及数组都要在堆上分配


**组成划分**

>从内存回收的角度来看，由于现在收集器基本都采用分代收集算法，所以Java堆中还可以细分为：新生代和老年代；再细致一点的有Eden空间、FromSurvivor空间、ToSurvivor空间等。(GC算法可参考[笔记：JVM垃圾收集算法](https://github.com/sparkfengbo/AndroidNotes/blob/master/JVM/JVM%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E7%AE%97%E6%B3%95.md))
>根据Java虚拟机规范的规定，Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘空间一样。在实现时，既可以实现成固定大小的，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的（通过-Xmx和-Xms控制）。如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。


### 1.2 方法区

`-XX:PermSize`  `-XX:MaxPermSize`

(JDK 1.7 逐步去除永久代)

和Java堆一样，也是**被所有线程共享的一块内存区域**

**目的**

它用于存储已被虚拟机加载的 **类信息、常量、静态变量、即时编译器编译后的代码** 等数据。

**运行时常量池**

运行时常量池是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池（ConstantPoolTable），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。当常量池无法再申请到内存时会抛出OutOfMemoryError异常。

### 1.3 虚拟机栈

是**线程私有**，生命周期与线程相同

使用`-Xss` 参数控制栈区大小(对于HotSpot来说，设置栈大小只能通过这个参数，因为HosSpot不缺分虚拟机栈和本地方法栈)

>虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧（StackFrame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。
>
>在Java虚拟机规范中，对这个区域规定了两种异常状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；如果虚拟机栈可以动态扩展（当前大部分的Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈），如果扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。

- [java StackOverflowError与OutOfMemoryError区别](http://blog.csdn.net/chenchaofuck1/article/details/51144223)

**局部变量表部分**

>局部变量表存放方法参数和方法内部定义的局部变量，包括编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。
>
>局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。
>
>以变量槽（Slot）为最小单位

### 1.4 本地方法栈

>本地方法栈（NativeMethodStack）与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。
>
>与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

### 1.5 程序计数器

是**线程私有**，生命周期与线程相同

>是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。
在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
>
>为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。
>
>如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Native方法，这个计数器值则为空（Undefined）。此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

