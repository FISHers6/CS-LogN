# JUC

## 分类

### 线程管理

- 线程池相关类

	- Executor、Executors、ExecutorService
	- 常用的线程池：FixedThreadPool、CachedThreadPool、ScheduledThreadPool、SingleThreadExecutor

- 能获取子线程的运行结果

	- Callable、Future、FutureTask

### 并发流程管理

- CountDwonLatch、CyclicBarrier、Semaphore、Condition

### 实现线程安全

- 互斥同步(锁)

	- Synchronzied、及工具类Vector、Collections
	- Lock接口的相关类：ReentrantLock、读写锁

- 非互斥同(原子类)

	- 原子基本类型、引用类型、原子升级、累加器

- 并发容器

	- ConcurrentHashMap、CopyOnWriteArrayList、BlockingQueue

- 无同步与不可变方案

	- final关键字、ThreadLocal栈封闭

## 线程池

### 使用线程池的作用好处

- 降低资源消耗

	- 重复利用已创建的线程降低线程创建和销毁造成的消耗

- 提高响应速度

	- 任务到达，可以不需要等到线程创建就能立即执行

- 提高线程的可管理性

	- 使用线程池可以进行统一的分配，调优和监控

### 线程池的参数

- corePoolSize、maximumPoolSize、keepAliveTime、workQueue、threadFactory、handler
- 图示

	- 

### 常用线程池的创建与规则

- 线程添加规则

	- 1.如果线程数量小于corePoolSize，即使工作线程处于空闲状态，也会创建一个新线程来运行新任务，创建方法是使用threadFactory
	- 2.如果线程数量大于corePoolSize但小于maximumPoolSize，则将任务放入队列
	- 3.如果workQueue队列已满，并且线程数量小于maxPoolSize，则开辟一个非核心新线程来运行任务
	- 4.如果队列已满，并且线程数大于或等于maxPoolSize，则拒绝该任务，执行handler
	- 图示(分别与3个参数比较)

		- 

- 常用线程池

	- newFixedThreadPool

		- 创建固定大小的线程池，使用无界队列会发生OOM

	- newSingleThreadExecutor

		- 创建一个单线程的线程池，线程数为1

	- newCachedThreadPool

		- 创建一个可缓存的线程池，60s会回收部分空闲的线程。采用直接交付的队列 SynchronousQueue ，队列容量为0，来一个创建一个线程

	- newScheduledThreadPool

		- 创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求

- 如何设置初始化线程池的大小？

	- 可根据线程池中的线程
处理任务的不同进行分别估计

		- CPU 密集型任务

			- 大量的运算，无阻塞
通常 CPU 利用率很高
应配置尽可能少的线程数量
设置为 CPU 核数 + 1

		- IO 密集型任务

			- 这类任务有大量 IO 操作
伴随着大量线程被阻塞
有利于并行提高CPU利用率
配置更多数量： CPU 核心数 * 2

- 使用线程池的注意事项

	- 1.避免任务堆积(无界队列会OOM)、2.避免线程数过多(cachePool直接交付队列)、3.排查线程泄露

### 线程池的状态和常用方法

- 线程池的状态

	- RUNNING(接受并处理任务中)、
SHUTDOWN(不接受新任务但处理排队任务)、
STOP(不接受新任务 也不处理排队任务 并中断正在进行的任务)、
TIDYING、TEMINATED(运行完成)

- 线程池停止

	- shutdown

		- 通知有序停止，先前提交的任务务会执行

	- shutdownNow

		- 尝试立即停止，忽略队列里等待的任务

### 线程池的源码解析

- 线程池的组成

	- 1.线程池管理器
2.工作线程
3.任务队列：无界、有界、直接交付队列
4.任务接口Task
	- 图示

		- 

- Executor家族

	- Executor顶层接口，只有一个execute方法
	- ExecutorService继承了Executor，增加了一些新的方法，比如shutdown拥有了初步管理线程池的功能方法
	- Executors工具类，来创建，类似Collections

		- 

	- 图示

		- 

- 线程池实现任务复用的原理

	- 线程池对线程作了包装，不需要启动线程，不需要重复start线程，只是调用已有线程固定数量的线程来跑传进来的任务run方法
	- 添加工作线程

		- 
		- 
		- 4步：1. 获取线程池状态、4.判断是否进入任务队列 3.根据状态检测是否增加工作线程4.执行拒绝handler

	- 重复利用线程执行不同的任务

		- 

### 面试题

- 为什么要使用线程池？
- 如何使用线程池？
- 线程池有哪些核心参数？
- 初始化线程池的大小的如何算？
- shutdown 和 shutdownNow 有什么区别？

## ThreadLocal

### ThreadLocal的作用好处

- 为每个线程提供存储自身独立的局部变量，实现线程间隔离
- 即：达到线程安全，不需要加锁节省开销，减少参数传递

### ThreadLocal的使用场景

- 1.每个线程需要一个独享的对象，如 线程不安全的工具类，(线程隔离)
- 2.每个线程内需要保存全局变量，如 拦截器中的用户信息参数，让不同方法直接使用，避免参数传递过多，(局部变量安全，参数传递）

### ThreadLocal的实现原理

- 每个 Thread 维护着一个 ThreadLocalMap 的引用；ThreadLocalMap 是 ThreadLocal 的内部类，用 Entry 来进行存储；key就对应一个个ThreadLocal
- get方法：取出当前线程的ThreadLocalMap，然后调用map.getEntry方法，把ThreadLocal作为key参数传入，取出对应的value
- set方法：往 ThreadLocalMap 设置ThreadLocal对应值
initalValue方法：延迟加载，get的时候设置初始化
- 图示

	- 

### 缺陷注意

- value内存泄漏

	- 原因：ThreadLocal 被 ThreadLocalMap 中的 entry 的 key 弱引用。如果 ThreadLocal 没有被强引用， 那么 GC 时 Entry 的 key 就会被回收，但是对应的 value 却不会回收，就会造成内存泄漏
	- 解决方案：每次使用完 ThreadLocal，都调用它的 remove () 方法，清除value数据。
	- 源码图示

		- 

### 面试题

- ThreadLocal 的作用是什么？
- 讲一讲ThreadLocal的实现原理(组成结构)
- ThreadLocal有什么风险？

## Callable与Future

### Callable

- 引入目的

	- 解决Runnable的缺陷

		- 1.没有返回值，因为返回类型为void
		- 2.不能抛出异常，因为没有继承Execption接口

- 是什么如何使用

	- Callable是类似于Runnable的接口，实现Callable接口的类和实现Runnable的类都是可被其它线程执行的任务。
	- 实现Call方法，可以有返回值

### Future

- 引入目的

	- Future的核心思想是：一个方法的计算过程可能非常耗时，一直在原地等待方法返回，显然不明智。可以把该计算过程放到子线程去执行，并通过Future去控制方法的计算过程，在计算出结果后直接获取该结果。

- 常用方法

	- get方法：获取结果，在没有计算出结果前，会进入阻塞态

- 使用场景

	- 用法1：线程池的submit方法返回Future对象
	- 用法2：用FutureTask来创建Future

- 注意点

	- 当for循环批量获取future的结果时，容易block，get方法调用时应使用timeout限制
	- Future和Callable的生命周期不能后退

- Callable和Future的关系

	- Future相当于一个存储器，它存储未来call()任务方法的返回值结果
	- 可以用Future.get方法来获取Callable接口的执行结果，在call()未执行完毕之前没调用get的线程会被阻塞
	- 线程池传入Callable，submit返回Future，get获取值

		- 

- FutureTask

	- FutureTask是一种包装器，可以把Callable转化成Future和Runnable，它同时实现了二者的接口。所以既可以作为Runnable任务被线程执行，又可以作为Future得到Callable的返回值
	- 图示

		- 

## final与不变性

### 什么是不变性（Immutable）

- 如果对象在被创建后，状态就不能被修改，那么它就是不可变的。
- 具有不变性的对象一定是线程安全的，我们不需要对其采取任何额外的安全措施，也能保证线程安全。

### final的作用

- 类防止被继承、方法防止被重写、变量防止被修改
- 天生是线程安全的(因为不能修改)，而不需要额外的同步开销

### final的3种用法：修饰变量、方法、类

- final修饰变量

	- 被final修饰的变量，意味着值不能被修改。
如果变量是对象，那么对象的引用不能变，但是对象自身的内容依然可以变化。
	- 赋值时机

		- 属性被声明为final后，该变量则只能被赋值一次。且一旦被赋值，final的变量就不能再被改变，如论如何也不会变。
		- 区分为3种

			- final instance variable（类中的final属性）

				- 等号右侧、构造函数、初始化代码块

			- final static variable（类中的static final属性）

				- 等号右侧、静态初始化代码块

			- final local variable（方法中的final变量）

				- 使用前复制即可

		- 为什么规定时机

			- 根据JVM对类和成员变量、静态成员变量的加载规则来看：如果初始化不赋值，后续赋值，就是从null变成新的赋值，这就违反final不变的原则了！

- final修饰方法（构造方法除外）

	- 不可被重写，也就是不能被override，即便是子类有同样名字的方法，那也不是override，与static类似*

- final修饰类

	- 不可被继承，例如典型的String类就是final的

### 栈封闭 实现线程安全

- 在方法里新建的局部便咯，实际上是存储在每个线程私有的栈空间，线程栈不能被其它线程访问，所以不会有线程安全问题，如ThreadLocal

### 面试题

- 

## CAS

### 什么是CAS

- 我认为V的值应该是A，如果是的话那我就把它改成B，如果不是A（说明被别人修改过了），那我就不修改了，避免多人同时修改导致出错。
- CAS有三个操作数：内存值V、预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才将内存值修改为B，否则什么都不做。最后返回现在的V值。
- 最终执行CPU处理机提供的的原子指令

### 缺点

- ABA问题

	- 我认为 V的值为A，有其它线程在这期间修改了值为B，但它又修改成了A，那么CAS只是对比最终结果和预期值，就检测不出是否修改过

- CAS+自旋，导致自旋时间过长
- 改进：通过版本号的机制来解决。每次变量更新的时候，版本号加 1，如AtomicStampedReference的compareAndSet () 

### 应用场景

- 1 乐观锁：数据库、git版本号； 自旋 2 concurrentHashMap：CAS+自旋
3 原子类

### CAS底层实现

- 通过Unsafe获取待修改变量的内存递增，
比较预期值与结果，调用汇编cmpxchg指令

### 以AtomicInteger为例，分析在Java中是如何利用CAS实现原子操作的？

- 1.使用Unsafe类拿到value的内存递增，通过偏移量 直接操作内存数据
- 2.Unsafe的getAndAddInt方法，使用CAS+自旋尝试修改数据
- CAS的参数通过 预期值 与 实际拿到的值进行比较，相同就修改，不相同就自旋
- Unsafe提供硬件级别的原子操作，最终调用原子汇编指令的cmpxchg指令

## 锁

### 锁的分类

- 

### Lock锁接口

- 简介

	- Lock锁是一种工具，用于控制对共享资源的访问
	- 如：ReentrantLock

- Lock和Synchronized的异同点

	- 相同点

		- 都能达到线程安全的目的

	- 不同点

		- Lock 有比 synchronized 更精确的线程语义和更好的性能；高级功能
		- 1 实现原理不同

			- Synchronized 是关键字，属于 JVM 层面，底层是通过 monitorenter 和 monitorexit 完成，依赖于 monitor 对象来完成；
			-  Lock 是 java.util.concurrent.locks.lock 包下的，底层是AQS

		- 2 灵活性不同

			- Synchronized 代码完成之后系统自动让线程释放锁；ReentrantLock 需要用户手动释放锁，加锁解锁灵活

		- 3 等待时是否可以中断

			- Synchronized 不可中断，除非抛出异常或者正常运行完成；ReentrantLock 可以中断。一种是通过 tryLock，另一种是 lockInterruptibly () 放代码块中，调用 interrupt () 方法进行中断；

- 可见性

	- happens-before规则约定；Lock与Synchronized一致都可以保证可见性
	- 即下一个线程加锁时可以看到上一个释放锁的线程发生的所有操作

### 乐观锁与悲观锁

- 悲观锁(互斥同步锁)

	- 思想

		- 锁住数据，让别人无法访问，确保数据万无一失

	- 实例

		- Synchronized、Lock相关类
		- 应用实例：select 把库锁住，属于悲观锁，更新期间其它人不能修改

	- 缺点

		- 在阻塞和唤醒性能开销大(用户态核心态切换、上下文切换、检查是否有线程被唤醒)
		- 持有锁的线程被阻塞时无法释放，有可能造成永久阻塞

- 乐观锁

	- 思想

		- 认为自己在操作数据时不会有其它线程干扰，所以不需要锁住被操作对象
		- 在更新数据的时候，去对比修改期间有没有被其它人改变过，没改过就正常修改(类似CAS思想)
		- 乐观锁一般由CAS实现：CAS在一个原子操作内把数据对比且交换，在此期间不能被打断的

	- 实例

		- 原子类、并发容器
		- 应用实例：数据库版本号控制、git版本号

	- 优缺点对比

		- 悲观锁一旦切换就不用再考虑切换CPU等操作了，一劳永逸，开销固定
		- 乐观锁，会一步步尝试自旋来获取锁，自旋开销

- 对比

	- 

### 可重入锁与非可重入锁

- 什么是可重入

	- 拿到锁的线程又请求这把锁，允许通过

- 可重入的好处

	- 避免死锁(拿到锁的线程内部又请求该锁)
	- 提升封装性，避免一次次加锁

- 可重入锁ReentrantLock与非可重入锁ThreadPoolExecutor的Worker类对比

	- 

### 公平锁和非公平锁

- 公平锁

	- 介绍

		- 公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁

	- 优点

		- 公平锁的优点是公平执行，等待锁的线程不会饿死

	- 缺点

		- 缺点是整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大

- 非公平锁

	- 介绍

		- 多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取锁的场景

	- 优点

		- 减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程

	- 缺点

		- 处于等待队列中的线程可能会饿死，或者等很久才会获得锁

- 优缺点对比

	- 

- 源码分析

	- 

### 共享锁和排他锁

- 排他锁

	- 介绍

		- 排他锁，获取锁后，既能读又能写，但是此时其它线程不能获取这个锁了，只能由当前线程修改数据独享锁，保证了线程安全，synchronized
		- 又称为 独占锁，写锁

- 共享锁

	- 介绍

		- 获取共享锁后，其它线程也可以获取共享锁完成读操作，但都不能修改删除数据
		- 又成为 读锁

- ReentrantReadWriteLock

	- 读写锁的作用

		- 共享锁减少了多个读都加锁的开销，线程也安全
		- 在读的地方使用读锁，在写的地方写锁；在没有写锁的情况下，读操作无阻塞，提高程序效率

	- 读写锁的规则

		- 要么可以多读，要么只能一写
		- 读写锁只是一把锁，可以通过两个方式锁定：读锁定 或 写锁定

	- 一把锁两种方式锁定

		- readLock() 读锁
		- writeLock() 写锁

	- 读线程插队策略(非公平下)

		- 写锁可以随时插队，参与竞争
		- 读锁仅在等待队列头节点为写的时候不允许插队；当队头为读的时候可以去插队。

	- 锁升级

		- 引入场景

			- 假如一开始持有写锁，但我写需求完了，后面都是读的需求了，如果还占用写锁就浪费资源开销

		- 策略

			- 只允许降级，不允许升级

	- 适合场景

		- 读多写少，提高并发效率

### 自旋锁和阻塞锁

- 阻塞锁

	- 思想

		- 没拿到锁之前，会直接把线程阻塞，直到被唤醒

	- 开销缺陷

		- 阻塞或唤醒一个线程需要操作系统切换CPU状态来完成，恢复现场等需要消耗处理机时间；如果同步代码块的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长，得不偿失

- 自旋锁

	- 思想

		- 让当前抢锁失败的线程进行自旋，如果在自旋完成后前面锁定同步资源的线程已经释放了锁，那么当前线程就可以不必阻塞而是直接获取同步资源，从而避免切换线程的开销

	- 开销缺陷

		- 自旋占用时间长，起始开销低，但消耗CPU资源开销会线性增长

- 源码分析

	- atomic包下的类基本都是自旋锁的实现
	- AtomicInteger的实现：自旋锁实现原理是CAS，Atomic调用Unsafe进行自增add的源码中的do-while循环就是一个自旋操作，使用CAS如果修改过程中遇到其它线程修改导致没有秀嘎四成功，就在while里死循环，直至修改成功
	- 图示

		- 

- 适用场景

	- 多核、临界区短小

### 可中断锁

- 介绍

	- 线程B等待线程A释放锁时，线程B不想等待了，想处理其它事情，我们可以中断它

- 使用场景

	- synchronized是不可中断锁，Lock是可中断锁(tryLock(time) 和 lockInterruptibly)响应中断

### 锁优化

- JDK1.6 后对synchronized锁的优化

	-  JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。
	- 偏向锁

		- 无竞争条件下,消除整个同步互斥，连CAS都不操作；即这个锁会偏向于第一个获得它的线程

	- 轻量级锁

		- 无竞争条件下,通过CAS消除同步互斥，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。

	- 重量级锁

		- 互斥同步锁

	- 自旋锁

		- 为了减少线程状态改变带来的消耗,不停地执行当前线程

	- 自适应自旋锁

		- 自旋的时间不固定了，如设置自旋次数

	- 锁消除

		- 不可能存在共享数据竞争的锁进行消除；

	- 锁粗化

		- 锁粗化就是增大锁的作用域；如解决加锁操作在循环体内的频开销

- 写代码时的优化

	- 缩小同步代码块、如不要锁住方法
	- 减少锁的请求次数， 如一批一批请求
	- 参考LongAdder的思想，每个段有自己的计数器，最后才合并

### 面试题

- 什么是公平锁？什么是非公平锁？
- 自旋锁解决什么问题?自旋锁的原理是什么?自旋的缺点?
- 说说 JDK1.6 之后的synchronized 关键字底层做了哪些优化，可以详细介绍一下这些优化吗？
- 说说 synchronized 和 java.util.concurrent.locks.Lock 的异同？

## 原子类atomic包

### 原子类的作用

- 原子类的作用和锁类似，都是为了保证并发下线程安全
- 粒度更细，变量级别
- 效率更高，除了高度竞争外

### 原子类的种类

- Atomic*基本类型原子类：AtomicInteger、AtomicLong、AtomicBoolean
- Atomic*Array数组类型原子类：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
- Atomic*Reference 引用类型原子类：AtomicReference等
- AtomicIntegerFiledUpdate等升级类型原子类
- Adder累加器、Accumlator累加器

### AtomicInteger

- 常用方法

	- get、getAndSet、getAndIncrement、compareAndSet(int expect,int update)

- 实现原理

	-  AtomicInteger 内部使用 CAS 原子语义来处理加减等操作。CAS通过判断内存某个位置的值是否与预期值相等，如果相等则进行值更新
	- CAS 是内部是通过 Unsafe 类实现，而 Unsafe 类的方法都是 native 的，在 JNI 里是借助于一个 CPU 指令完成的，属于原子操作。

- 缺点

	- 循环开销大。如果 CAS 失败，会一直尝试
	- 只能保证单个共享变量的原子操作，对于多个共享变量，CAS 无法保证，引出原子引用类
	- 用CAS存在 ABA 问题

### Adder累加器

- 引入目的/改进思想

	- AtomicLong在每一次加法都要flush和refresh主存，与JMM内存模型有关。工作线程之间不能直接通信，需要通过主内存间接通信

- 设计思想

	- Java8引入，高并发下LongAdder比AtomicLong效率高，本质是空间换时间
	- 竞争激烈时，LongAdder把不同线程对应到不同的Cell单元上进行修改，降低了冲突的概率，是多段锁的理念，提高了并发性
	- 每个线程都有自己的一个计数器，不存在竞争
	- sum源码分析：最终把每一个Cell的计数器与base主变量相加

### 面试题

- AtomicInteger 怎么实现原子操作的？
- AtomicInteger 有哪些缺点？

## 并发容器

### ConcurrentHashMap

- 集合类历史

	- Vector的方法被synchronizd修饰，同步锁；不允许多个线程同时执行。并发量大的时候性能不好
	- Hashtable是线程安全的HashMap，方法也是被synchronized修饰，同步但并发性能差
	- Collections工具类，提高的有synchronizedList和synchronizedMap，代码内使用sync互斥变量加锁

- 为什么需要

	- 为什么不用HashMap

		- 1.多线程下同时put碰撞导致数据丢失
		- 2.多线程下同时put扩容导致数据丢失
		- 3.死循环造成的CPU100%

	- 为什么不用Collection.synchronizedMap

		- 同步锁并发性能低

- 数据结构与并发策略

	- JDK1.7

		- 数组+链表，拉链法解决冲突
		- 采用分段锁，每个数组结点是一个独立的ReentrantLock锁，可以支持同时并发写

	- JDK1.8

		- 数组+链表+红黑树，拉链法和树化解决冲突
		- 采用CAS+synchronized锁细化

	- 1.7到1.8改变后有哪些优点

		- 1.数据结构由链表变为红黑树，树查询效率更高
		- 2.减少了Hash碰撞，1.7拉链法
		- 3.保证了并发安全和性能，分段锁改成CAS+synchronized
		- 为什么超过8要转为红黑树，因为红黑树存储空间是结点的两倍，经过泊松分布，8冲突概率低

- 注意事项

	- 组合操作线程不安全，应使用putIfAbsent提供的原子性操作

### CopyOnWriteArrayList

- 引入目的

	- Vector和SynchronizedList锁的粒度太大并发效率低，并且迭代时无法编辑exceptMod!=Count

- 适合场景

	- 读多写少，如黑名单管理每日更新

- 读写规则

	- 是对读写锁的升级：读取完全不用加锁，读时写入也不会阻塞。只有写入和写入之间需要同步

- 实现原理

	- 创建数据的新副本，实现读写分离，修改时整个副本进行一次复制，完成后最后再替换回去；由于读写分离，旧容器不变，所以线程安全无需锁
	- 在计算机内存中修改不直接修改主内存，而是修改缓存(cache、对拷贝的副本进行修改)，再进行同步(指针指向新数据)。

- 缺点

	- 1.数据一致性问题，拷贝不能保证数据实时一致，只能保证数据最终一致性
	- 2.内存占用问题，写复制机制，写操作时内存会同时驻扎两个对象的内存

### 并发队列

- 为什么使用队列

	- 用队列可以在线程间传递数据，缓存数据
	- 考虑锁等线程安全问题的重任转移到了“队列”上

- 并发队列关系图示

	- 

- BlockingQueue阻塞队列

	- 阻塞队列是局由自动阻塞功能的队列，线程安全；take方法移除队头，若队列无数据则阻塞直到有数据；put方法插入元素，如果队列已满就无法继续插入则阻塞直到队列里有了空闲空间
	- ArrayBlockQueue

		- 有界可指定容量、可公平
		- Put源码加锁，可中断的上锁方法。没满才可以入队，否则一直await等待。

	- LinkedBlockingQueue

		- 无界容量为MAX_VALUE，内部结构Node
		- 使用了两把锁take锁和put锁互补干扰

	- PriorityBlockingQueue

		- 支持优先级，无界队列

	- SynchronousQueue

		- 直接传递的队列，容量0，效率高线程池的CacheExecutorPool使用其作为工作队列

	- DelayQueue

		- 无界队列，根据延迟时间排序

- 非阻塞队列

	- ConcurrentLinkedQueue

		- 使用链表作为队列存储结构
		- 使用Unsafe的CAS非阻塞方法来实现线程安全，无需阻塞，适合对性能要求较高的并发场景

- 选择合适的队列

	- 边界上看

		- ArrayBlockQueue有界；LinkedBlockQueue无界适合容量大容量激增

	- 内存上看

		- ArrayBlockQueue内部结构是array，从内存存储上看，连续存储更加整齐。而LinkedBlockQueue采用链表结点，可以非连续存储。

	- 吞吐量上看

		- 从性能上看LinkedBlockQueue的put锁和锁分开，锁粒度更细，所以优于ArrayBlockQueue

### 总结并发容器对比

- 分为3类：Concurrent*、CopyOnWrite*、Blocking*
- Concurrent*的特定是大部分使用CAS并发；而CopyOnWrite通过复制一份元数据写加锁实现；Blocking通过ReentLock锁底层AQS实现

## 并发流程控制工具类

### 控制并发流程工具类的作用

- 控制并发流程的工具类，作用是帮助程序员更容易让线程之间相互配合，来满足业务逻辑
- 并发工具类图示

	- 

### CountDownLatch倒计时门闩

- 作用(事件)

	- 一个线程等多个线程、或多个线程等一个线程完成到达，才能继续执行

- 常用方法

	- 构造函数中传入倒数值、await、countDown

### Semaphore信号量

- 作用

	- 用来限制管理数量有限的资源的使用情况，相当于一定数量的“许可证”

- 常用方法

	- 构造函数中传入数量、acquire、release

### Condition条件对象

- 作用

	- 等待条件满足才放行，否则阻塞；一个锁可以对应多个条件

- 常用方法

	- lock.newCondition、await、signal

### CyclicBarrier循环栅栏

- 作用(线程)

	- 多个线程互相等待，直到达到同一个同步点（屏障），再继续一起执行

- 常用方法

	- 构造函数中传入个数、await

## AQS

### AQS的作用

- AQS是一个用于构建锁、同步器、协作工具类的框架，有了AQS后，更多的协作工具类都可以很方便的写出来

### AQS的应用场景

- Exclusive(独占)

	- ReentrantLock 公平和非公平锁

- Share(共享)

	- Semaphore/CountDownLatch/CyclicBarrier

### AQS原理解析

- 核心三要素

	- 1.sate

		- 使用一个 int 成员变量来表示同步状态 state，被volatile修饰，会被并发修改，各方法如getState、setState等使用CAS保证线程安全
		- 在ReentrantLock中，表示可重入的次数
		- 在Semaphore中，表示剩余许可证信号的数量
		- 在CountDownLatch中，表示还需要倒数的个数

	- 2.控制线程抢锁和配合的FIFO队列

		- 获取资源线程的排队工作

	- 3.期望协作工具类去实现的“获取/释放”等唤醒分配的方法策略

- AQS的用法

	- 第一步：写一个类，想好协作的逻辑，实现获取/释放方法
	- 第二步：内部写一个Sync类继承AbstractQueueSynchronizer
	- 第三步：Sync类根据独占还是共享重写tryAcquire/tryRelease或tryAcquireShared和tryReleaseShared等方法，在之前写的获取/释放方法中调用AQS的acquire/release或则Shared方法

### AQS应用实例源码解析

- AQS在CountDownLatch的应用

	- 内部类Sync继承AQS
	- 1.state表示门闩倒数的count数量，对应getCount方法获取
	- 2.释放方法，countDown方法会让state减1，直到减为0时就唤醒所有线程。countDown方法调用releaseShared，它调用sync实现的tryReleaseShared，其使用CAS+自旋锁，来实现安全的计数-1
	- 3.阻塞方法，await会调用sync提供的aquireSharedInterruptly方法，当state不等于0时，最终调用LockUpport的park，它利用Unsafe的park，native方法，把线程加入阻塞队列
	- 总结

		- 

- AQS在Semphore的应用

	- state表示信号量允许的剩余许可数量
	- tryAcquire方法，判断信号量大于0就成功获取，使用CAS+自旋改变state状态。如果信号量小于0了，再请求时tryAcquireShared返回负数，调用aquireSharedInterruptly方法就进入阻塞队列
	- release方法，调用sync实现的releaseShared，会利用AQS去阻塞队列唤醒一个线程
	- 总结

		- 

- AQS在ReentrantLock的应用

	- state表示已重入的次数，独占锁权保存在AQS的Thread类型的exclusiveOwnerThread变量中
	- 释放锁: unlock方法调用sync实现的release方法，会调用tryRelease，使用setState而不是CAS来修改重入次数state，当state减到0完全释放锁
	- 加锁lock方法：调用sync实现的lock方法。CAS尝试修改锁的所有权为当前线程，如果修改失败就要调用acquire方法再次尝试获取，acquire方法调用了AQS的tryAcquire，这个实现在ReentantLock的里面，获取失败加入到阻塞队列

### 通过AQS自定义同步器

- 自定义同步器在实现时只需要根据业务逻辑需求，实现共享资源 state 的获取与释放方式策略即可
- 至于具体线程等待队列的维护（如获取资源失败入队 / 唤醒出队等），AQS 已经在顶层实现好了
