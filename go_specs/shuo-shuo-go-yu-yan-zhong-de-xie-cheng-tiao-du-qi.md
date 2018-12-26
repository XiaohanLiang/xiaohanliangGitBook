# 说说Go语言中的协程调度器

> Introduction
>
> 上一节有一个地方提到了：“如果一个协程发生了堵塞，那么Go语言会帮你自动切换到另一支协程上“
>
> 但是在操作系统课程里学到的是：“协程是一种用户级线程，操作系统不可见的，如果协程堵塞，整个程序就将堵塞”。 我一开始还以为我记错了， 其实这是因为课上学的是Linux OS的Scheduler， 而Go语言却自带一个Scheduler
>
> Go语言完成了语言级并发，使得开启并发变得非常容易  
> 出现这种事的原因就是Go语言有自己的Scheduler



## 为什么Go需要设置一个自己的Scheduler

【GC的原因】Go的垃圾回收需要stop the world，也就是只有当所有的goroutine先停止，这种时候下内存才能保持在一个一致的状态下。垃圾回收的时间根据内存情况的变化是不同的。  也就是说：

{% hint style="warning" %}
**我们必须等所有goroutine停止工作，才能展开回收**
{% endhint %}

可是，操作系统根本感知不到你自己定义的协程啊。那么使用操作系统的Scheduler就会一团混乱。这种时候下，我们必须定义一个自己的Scheduler，来管理自己定义出的这些协程。

既然Go语言已经有了自己的Scheduler，能够自主的切换协程，而不会出现因“协程”堵塞而导致整个进程堵塞，那么站在OS-Scheduler的角度来看，OS只需要一直等待这个线程运行到时间片满就可以了



## Scheduler究竟是如何管理协程的呢 

### 1. 先定义一下G/M/P

每一个go程序都会自带以下内容：  

*   **runtime - 负责与底层操作系统交互**
*   **scheduler：负责调度自己创造出的协程**

{% code-tabs %}
{% code-tabs-item title="Scheduler中关于G/M/P三种概念的定义" %}
```text
// Goroutine scheduler
// The scheduler's job is to distribute ready-to-run goroutines over worker threads.
//
// The main concepts are:
// G - goroutine.
// M - worker thread, or machine.
// P - processor, a resource that is required to execute Go code.
//     M must have an associated P to execute Go code, however it can be
//     blocked or in a syscall w/o an associated P.
//
// Design doc at https://golang.org/s/go11sched.
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 2. Scheduler系统做出的内存管理策略 - SplitStack & Runtime

想要自己做出一套调度系统，避免不了的一个问题就是栈的管理。每一个协程都会有一个自己的函数栈，创建一个协程的同时，相应的函数栈也必须被创建。 那么可知随着函数的执行，函数栈会不断的增长。

在操作系统课程里我们学过，栈的增长都是连续的，并且多个协程之间会共享内存空间。每次我们给这些协程先分配一些起始地址不同的内存，来给他们用作栈。 那么一上来到底分配多大内存用作栈呢？ 于是我们创造了Split Stack技术：

* 一上来只分配一个很小的栈
* 如果因为某次函数调用导致了栈空间不足的问题，就会在另一个地方分配一块内存，关于这次函数调用所用到的参数拷贝到这块新栈空间里面去，接下来这个函数就会在新栈里执行下去
* runtime的栈管理方式于此类似，也是上来分配一个固定大小的内存，但是在栈空间不足的情况下会把旧栈里的所有内容拷贝到更大的新栈里去，避免了Split Stack产生的频繁的内存分配与释放效率问题

### 3. Scheduler系统做出的调度策略 - 抢占式调度策略

协程的执行是可以被抢占的，如果一个协程持续占用着CPU，长时间没有被切换过，runtime就会抢占CPU并把控制权交给其他协程。  runtime在程序开始运行之初，会创建一个系统线程运行`sysmon( )`函数，`sysmon( )`会在go程序的整个生命周期一直运行，用于达成调度目的：

1. 判断当前是否需要回收垃圾
2. 监视所有协程当前的运行状态，判断是否需要执行调度

#### 抢占过程执行细节

`sysmon( )`调用`retake( )`，retake遍历当前所有协程，如果一个协程正在处于执行状态并且已经执行了超长时间没有切换，这个协程就将被抢占：

1. 首先retake\( \)函数修改核心的参数，这使得核心中正在执行的协程，如果想执行下一次函数调用，结果直接就会是栈空间检查失败，   这个协程即将无法再执行函数调用了。  
2. 此时这条协程即将与机器解绑，停止执行，而后将这条协程加入queue list
3. 通过挑出一个新的协程加入处理器中，再将核心参数修改回来，继续执行下一个协程。





### References:

[https://zhuanlan.zhihu.com/p/27328476](https://zhuanlan.zhihu.com/p/27328476)

