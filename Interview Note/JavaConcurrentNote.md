# Java并发
## 线程和进程
1. 进程：资源调度的基本单位，
    - 系统运行程序的基本单位，
    - 当我们启动main函数时其实就是启动了一个JVM进程，而main函数所在的线程就是这个进程中的一个线程，也称主线程。

2. 线程：cpu调度的基本单位
    - 共享进程的堆和方法区资源，有自己独立的程序计数器，虚拟机栈（存储局部变量表、操作数栈、常量池引用等信息），本地方法栈

3. 线程进程的关系区别优缺点
    - 线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护，而进程正相反。

4. 线程的生命周期和状态
    - NEW：初始状态，线程被创建出来但没有被调用start()；
    - RUNNBALE:运行状态，线程被调用了start()等待运行的状态；
        - READY(可运行状态)获得CPU时间片后就处于RUNNING状态
        - RUNNING(运行状态)JVM中没有区分这两种状态
    - BLOCKED:阻塞状态，需要等待锁释放
    - WAITING:等待状态，表示该线程需要等待其他线程做出一定特定动作（通知或中断）
    - TIME-WAITING:超时等待，可以在指定的时间后自行返回而不是像WAITING那样一直等待。
    - TERMINATION:终止状态，表示该线程已经运行完毕。
    ![alt text](image-23.png)

5. 什么是线程死锁？如何避免死锁？如何预防和避免线程死锁？
    - 线程死锁是多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期的阻塞，因此程序不可能正常终止。
    - 产生死锁的四个必要条件：
        - **互斥条件**：该资源任意一个时刻只由一个线程占用
        - **请求与保持条件**：一个线程因请求资源而阻塞时，对已获得的资源保持不放
        - **不剥夺条件**：线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源
        - **循环等待条件**：若干线程之间形成一种头尾相接的循环等待资源关系；
    - 如何检测死锁：
        - 使用`jstack`查看JVM线程栈和堆内存的情况。如果有死锁，`jstack`输出会有`Found one Java-level deadlock：`的字样
        - 可以搭配使用top（动态查看进程状态、CPU、内存使用情况等）、df（显示文件系统的磁盘空间占用情况）、free（显示物理内存和交换分区的总量、使用量及剩余量）等命令查看操作系统的基本情况，出现死锁可能会导致CPU、内存等资源消耗过高。
    - 如何预防和避免死锁：
        - 预防死锁：破坏必要条件
            - 破坏请求与保持条件：一次性申请所有资源；
            - 破坏不剥夺条件：占用部分资源的线程进一步申请其他资源的时候，如果申请不到，可以主动释放它占有的资源；
            - 破坏循环等待条件：按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。
        - 避免死锁：银行家算法：进入**安全状态**
            - 安全状态指系统能按照某种线程**推进顺序**来为每个线程分配所需资源，直到满足每个线程对资源的最大要求，使得每个线程都可顺利完成。称这个推进顺序序列为**安全序列**。

## 乐观锁和悲观锁
1. 区别
    - 悲观锁：是假设最坏的情况，认为共享资源每次被访问的时候就会出现问题，所以每次在获取操作的时候都会上锁，这样其他线程想拿到这个资源就会阻塞直到被上一个持有者释放。
        - **共享资源每次只给一个线程使用，其他线程阻塞，用完后再把资源转让给其他线程。** eg:synchronized和Reentrantlock等独占锁
        - 多用于写比较多的情况
    - 乐观锁：是假设最好的情况，认为共享资源每次被访问的时候不会出现问题，线程可以不停执行，无需加锁也无需等待，只是在提交修改的时候去验证对应的资源是否被其他线程修改了（使用版本号机制或CAS算法）
        - 用于写比较少的情况。
        - 如何实现乐观锁：
            - 版本号机制：在数据表中加上一个数据版本号version字段，表示数据被修改的次数。
                - 当数据被修改时，version值加1。
                - 当线程A要更新数据值时，在读取数据的时候也会读取version值，在提交更新时，若刚才读取到的version值和当前数据库中的version值相等才更新，否则重试更新操作，直到更新成功。
            - CAS（Compare and swap比较与交换）算法：用一个预期值和要更新的变量值进行比较，两值相等才会进行更新。
                - CAS是原子操作
                - 三个操作数：
                    - V:要更新的变量值（Var）
                    - E:预期值（Expected）
                    - N：拟写入的新值（New）
                - 当且仅当V的值等于E时，CAS通过原子方式用新值N来更新V的值。如果不等，说明已经有其它线程更新了V，则当前线程放弃更新。
                - 当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。
2. CAS 
    - ConcurrentHashMap采用CAS+synchronized来保证并发安全
    - Java.util.concurrent.atomic包中的类通过volatile+CAS重试保证线程安全性。
    - Java中，实现CAS操作的关键类是Unsafe类：
        - sun.misc包下的Unsafe类提供了comapreAndSwapObject、comapreAndSwapInt、comapreAndSwapLong方法来实现的对Object、Int、Long类型的CAS操作。
        - Unsafe类中的CAS方法是native方法。native关键字表明这些方法是用本地代码实现的，而不是用Java实现的。这些方法直接调用底层的硬件指令来实现原子操作，通过C++内联汇编的形式实现的（通过JNI调用）。因此，CAS的具体实现与操作系统以及CPU密切相关。
        - Java.util.concurrent.atomic包提供了一些用于原子操作的类。这些类利用底层的原子指令，确保在多线程环境下的操作是线程安全的。
        - AtomicInteger（Java.util.concurrent.atomic包中）是Java的原子类之一，主要用于对int类型的变量进行原子操作， 它利用Unsafe类提供的低级别原子操作方法实现无锁的线程安全性。
        - 由于CAS操作可能会因为并发冲突而失败，因此通常会与While循环搭配使用，在失败后不断重试，直到操作成功。这就是**自旋锁机制**

3. 乐观锁存在哪些问题
    - **ABA问题**：一个变量初次读取是是A值，准备赋值的时候依然是A值，但有可能在这段时间内被改为了其他值，然后又改回A值，那CAS操作会误认为它从来没有被修改过
        - 加上版本号或者时间戳：JDK1.5之后的AtomicStampedReference类就是解决ABA问题的，其中的compareAndSet()方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，若全部相等，则用原子方式将该引用和该标志的值设置为给定的更新值。
    - **循环时间开销大**：CAS操作用到自旋操作，若长时间不成功，会给CPU带来非常大的执行开销。
        - 若JVM支持处理器提供的pause指令，那么自旋操作的效率将有所提升。pause操作有两个重要作用：
            1. 延迟流水线执行指令：pasue指令可以延迟指令的执行，从而减少CPU的资源消耗。具体的延迟时间取决于处理器的实现版本，在某些处理器上，延迟时间可能为零。
            2. 避免内存顺序冲突：在退出循环时，pause指令可以避免由于内存顺序冲突而导致的CPU流水线被清空，从而提高CPU的执行效率。
    - **只能保证一个共享变量的原子操作**：CAS操作只能对单个共享变量有效。
        - Java提供了AtomicReference类，使我们能够保证引用对象之间的原子性。通过将多个变量封装在一个对象中，我们可以使用AtomicRefernce来执行CAS操作。
        - 加锁

## synchronized和volatie
1. synchronized：Java中的一个关键字，同步的意思。可以保证被它修饰的方法或者代码块在任意时刻只能有一个执行。
    - Java早期版本中，是重量级锁，因为监视器锁（monitor）是依赖于底层的操作系统的Mutex Lock来实现的，Java的线程是映射到操作系统的原生线程之上的。线程的的挂起和唤醒都需要操作系统帮忙，而操作系统实现线程的切换需要用户态和内核态之间的转换，这个过程很耗时。
    - Java 6 之后引入了大量的优化如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等，提高了synchronized的效率。
    - 但由于偏向锁增加了JVM的复杂性，并且性能收益不明显，JDK15后就默认关闭（**仍然可以使用 -XX:+UseBiasedLocking 启用偏向锁**），JDK18后被彻底放弃。

2. 如何使用synchronized
    - 修饰实例方法：锁当前对象实例
        - 给当前对象实例加锁，进入同步代码之前要获得当前对象实例的锁；
    - 修饰静态方法：锁当前类
        - 给当前类加锁，会作用于类的所有对象实例，进入同步代码前要获得当前class的锁。
    - 修饰代码块：锁指定对象/类
        - synchroonized（object）表示进入同步代码库前要获得给定对象的锁。
        - synchroonized（类.class）表示进入同步代码前要获得给定Class的锁。

3. synchronized原理
    - 同步语句块的实现使用的是monitorenter和monitorexit指令，其中monitorenter指令指向同步代码块的开始位置，monitorexit指令则指明同步代码块的结束位置
        - 在执行monitorenter时，会尝试获取对象的锁（获取对象监视器monitor），如果锁的计数器为0则表示锁可以被获取，获取后将锁计数器设为1也就是加1；
        - 对象锁的拥有者线程才可以执行monitorexit指令来释放锁。在执行monitorexit指令后，将锁计数器设为0，表明锁被释放，其他线程可以尝试获得锁。
        - 如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。
    - 修饰方法的情况，使用的时ACC_SYNCHRONIZED标识，该标识指明了该方法是一个同步方法。JVM通过该ACC_SYNCHRONIZED访问标志来辨认一个方法是否为同步方法，从而执行相应的同步调用。
        - 如果是实例方法，JVM会尝试获取实例对象的锁。
        - 如果是静态方法，JVM会尝试获取当前class的锁。 
    - 两者本质都是对对象监视器monitor的获取。
    - 锁升级：无锁状态->偏向锁状态->轻量级锁->重量级锁

4. volatile：保证变量的可见性，将变量声明为volatile，这就指示JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。
    - 禁用CPU缓存。
    - 禁止指令重排序：插入特定的内存屏障的方式来禁止指令重排序。
    - 双重校验锁实现对象单例（线程安全）：
    ```Java
    public class Singleton{
        
        private volatile static Singleton uniqueInstance;

        private SingleTon(){
        }

        public static Singleton getUniqueInstance(){
            //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
    ```
    - uniqueInstance 采用 volatile 关键字修饰也是很有必要的， uniqueInstance = new Singleton(); 这段代码其实是分为三步执行：
        1. 为 uniqueInstance 分配内存空间
        2. 初始化 uniqueInstance
        3. 将 uniqueInstance 指向分配的内存地址
    - 但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

    - synchronized和volatile的区别：两个互补
        - volatile关键字是线程同步的轻量级实现，所以volatile性能比synchronized关键字好。但volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块。
        - volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。
        - volatile关键字主要用于解决变量在多个线程之间的可见性，而synchronized关键字解决的是多个线程之间访问资源的同步性。

- ReentrantLock:实现了Lock接口，是一个可重入且独占式的锁。
    - synchronized和ReentrantLock区别：
        - 两者都是可重入锁，递归锁，线程可以再次获取自己的内部锁。
        - synchronized是依赖JVM实现的，JDK1.6为synchronized关键字进行了很多升级，但这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。
        - ReentrantLock是JDK层面实现的（也就是API层面，需要lock()和unlock()方法配合try/finally语句块来完成）
        - 增加了一些高级功能
            - 等待可中断：当前线程在等待获取锁的过程中，如果其他线程中断当前线程，当前线程就会抛出InterruptException异常，可以捕捉该异常进行相应处理。
            - 可实现公平锁：公平锁就是先等待的线程先获得锁。
            - 可实现选择性通知（锁可以绑定多个条件）
            - 支持超时：可以指定等待获取锁的最长等待时间
        - ![alt text](image-30.png)



## ThreadLocal
1. ThreadLocal的作用：
    - ThreadLocal 类允许每个线程绑定自己的值，可以将其形象地比喻为一个“存放数据的盒子”。每个线程都有自己独立的盒子，用于存储私有数据，确保不同线程之间的数据互不干扰。当你创建一个 ThreadLocal 变量时，每个访问该变量的线程都会拥有一个独立的副本。这也是 ThreadLocal 名称的由来。线程可以通过 get() 方法获取自己线程的本地副本，或通过 set() 方法修改该副本的值，从而避免了线程安全问题。

2. 原理：
    - 最终的变量是放在了当前线程的ThreadLocalMap中，并不是存在ThreadLocal上， ThreadLocal可以理解为只是ThreadLocalMap的封装，传递了变量值。ThreadLocal类可以通过Thread.currentThread()获取到当前线程对象，直接通过getMap（Thread t）可以访问到该线程的ThreadLocalMap对象。
    - 每个Thread中都有一个ThreadLocalMap，而ThreadLocalMap可以存储以ThreadLocal为key，Object对象为value的键值对。
    - 比如我们在同一个线程中声明了两个ThreadLocal对象的话，Thread内部都是用仅有的哪个ThreadLocalMap存放数据的，ThreadLocalMap的key就是ThreadLocal对象，value就是ThreadLocal对象调用set方法设置的值。
    - ![alt text](image-24.png)‘

## 线程池
1. 为什么要使用线程池？
    - 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
    - 提高响应速度：当任务到达时，任务可以不需要等到线程创建就能立即执行。
    - 提高线程的可管理性：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

2. 为什么不推荐按使用内置线程池：
    - FixedThreadPool（固定数量线程池）和SingleThreadExecutor（只有一个线程的线程池）使用LinkedBlockingQueue（阻塞队列）,任务队列长度为Integer.MAX_VALUE（看作无界），可能导致OOM。
    - CachedThreadPool(可以根据实际情况调整线程数量的线程池)使用SynchronousQueue（同步队列），线程数量可达Integer.MAX_VALUE，任务过多时可能导致OOM。
    - ScheduledThreadPool（给定的延迟后运行任务或者定期执行任务的线程池 ）和SingleThreadScheduledExecutor使用DelayedWorkQueue（无界的延迟阻塞队列）,任务队列长队为Integer.MAX_VALUE，可能导致OOM。

3. 线程池常见参数有哪些？使用ThreadPoolExecutor构造函数来创建
    - **corePoolSize**:核心线程池大小：未达到队列容量时，最大可以同时运行的线程数量
    - **maximumPoolSize**:任务队列中存放的任务达到队列容量时，当前可以同时运行的线程数量变为最大线程数。
    - **workQueue**：任务队列：新任务来的时候先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
    - keepAliveTime:非核心线程空闲后，等待时间超过了keepAliveTime会被回收销毁。
    - unit:keepAliveTime的时间单位
    - threadFactory: executor创建新线程的时候会用到
    - handler:拒绝策略
    - ![alt text](image-25.png)

4. 线程池的拒绝策略
    - ThreadPoolExecutor.AbortPolicy:抛出RejectedExecutionException来拒绝新任务的处理。
    - ThreadPoolExecutor.CallerRunsPolicy:调用执行者自己的线程运行任务，如果运行程序关闭，则丢弃任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。
    - ThreadPoolExecutor.DiscardPolicy:不处理新任务，直接丢弃掉
    - ThreadPoolExecutor.DiscardOldestPolicy:将丢弃最早的未处理的任务请求。

5. 线程池的核心线程会被回收吗？（如果我想杀死核心线程怎么做）
    - ThreadPoolExecutor默认不会回收核心线程，即使它们已经空闲了。这是为了减少创建线程的开销，因为核心线程通常是要长期保持活跃的。
    - 但是，如果线程池是被用于周期性使用的场景，且频率不高（周期之间有明显的空闲时间），可以考虑将 allowCoreThreadTimeOut(boolean value) 方法的参数设置为 true，这样就会回收空闲（时间间隔由 keepAliveTime 指定）的核心线程了。

6. 线程池处理任务的流程
    ![alt text](image-26.png)

7. 线程池中线程异常后，销毁还是复用
    - 使用execute（）提交任务的：如果这个异常没有在任务内被捕获，那么该异常会导致当前线程终止，并且异常会被打印到控制台或日志文件中。线程池会检测到这种线程终止，并**创建一个新线程来替换它**，从而保持配置的线程数不变。
    - 使用submit()提交任务：异常会被封装在由submit()返回的Future对象中。当调用Future.get()方法时，可以捕获到一个ExecutionException。在这种情况下，线程不会因为异常而终止，它会**继续存在于线程池中，准备执行后续的任务**

8. 如何设计一个能够根据任务的优先级来执行的线程池
    - 可以使用PriorityBlockingQueue （优先级阻塞队列）作为任务队列（ThreadPoolExecutor 的构造函数有一个 workQueue 参数可以传入任务队列）。
    - PriorityBlockingQueue 是一个支持优先级（优先级可能导致低优先级饥饿）的无界阻塞队列（无界可能导致OOM），可以看作是线程安全的 PriorityQueue，两者底层都是使用小顶堆形式的二叉堆，即值最小的元素优先出队。不过，PriorityQueue 不支持阻塞操作。
    - ![alt text](image-27.png)

## AQS
1. AQS原理：AQS是抽象队列同步器。
    - 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
    - 如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒是锁分配的机制，这个机制AQS是基于CLH锁进一步优化实现的
    - CLH锁对自旋锁进行了改进，是基于单链表的自旋锁。在多线程场景下，会将请求锁的线程组织成一个单向队列，每个等待的线程会通过自旋访问前一个线程节点的状态，前一个节点释放锁后，当前节点才可以获取锁。![alt text](image-28.png)
    - AQS中使用的等待队列是CLH锁队列的变体：
        - CLH变体队列是一个双向队列，会暂时获取不到锁的线程将被加入到该队列中，与CLH原队列的区别有：  
            - 由自旋优化为自旋+阻塞：自旋操作的性能很高，但大量的自旋操作比较占用 CPU 资源，因此在 CLH 变体队列中会先通过自旋尝试获取锁，如果失败再进行阻塞等待。
            - 由单项队列优化为双向队列：因为会对线程进行阻塞操作，所以前面的线程释放锁后需要对后面的线程进行唤醒，因此需要next指针。
            ![alt text](image-29.png)
    - AQS使用int成员变量state（由volatile修饰，用于展示当前临界资源的获锁情况）表示同步状态，通过内置的线程等待队列来完成资源线程的排队工作

2. Semaphore
    - 作用：synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，而Semaphore(信号量)可以用来控制同时访问特定资源的线程数量。
        - 公平模式：调用acquire()方法的顺序就是获取许可证的顺序，FIFO
        - 非公平模式：抢占式的
    
    - 原理：Semaphore 是共享锁的一种实现，它默认构造 AQS 的 state 值为 permits，你可以将 permits 的值理解为许可证的数量，只有拿到许可证的线程才能执行。
        - 调用semaphore.acquire()，线程尝试获取许可证
            - 如果state>=0，则表示可以获取成功。如果获取成功的话，使用CAS操作去修改state的值state=state-1.
            - 如果state<=0,则表示许可证数量不足。创建一个Node节点加入阻塞队列，挂起当前线程。
            
        - 调用semaphore.release(); ，线程尝试释放许可证，并使用 CAS 操作去修改 state 的值 state=state+1。
            - 释放许可证成功之后，同时会唤醒同步队列中的一个线程。被唤醒的线程会重新尝试去修改 state 的值 state=state-1 ，如果 state>=0 则获取令牌成功，否则重新进入阻塞队列，挂起线程。

3. CountDownLatch
    - 作用：
        - CountDownLatch 允许 count 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。
        - CountDownLatch 是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 CountDownLatch 使用完毕后，它不能再次被使用。CountDownLatch 的原理是什么？
    - 原理：
        - 是共享锁的一种实现，它默认构造AQS的state值为count。
        - 当线程使用 countDown() 方法时,其实使用了tryReleaseShared方法以 CAS 的操作来减少 state,直至 state 为 0 。
        - 当调用 await() 方法的时候，如果 state 不为 0，那就证明任务还没有执行完毕，await() 方法就会一直阻塞，也就是说 await() 方法之后的语句不会被执行。
        - 直到count 个线程调用了countDown()使 state 值被减为 0，或者调用await()的线程被中断，该线程才会从阻塞中被唤醒，await() 方法之后的语句得到执行。
    - 使用场景：使用多线程读取文件场景：
        - 我们要读取处理 6 个文件，这 6 个任务都是没有执行顺序依赖的任务，但是我们需要返回给用户的时候将这几个文件的处理的结果进行统计整理。
        - 为此我们定义了一个线程池和 count 为 6 的CountDownLatch对象 。使用线程池处理读取任务，每一个线程处理完之后就将 count-1，调用CountDownLatch对象的 await()方法，直到所有文件读取完之后，才会接着执行后面的逻辑。
        - ![alt text](image-31.png)

4. CyclicBarrier
    - CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。
    - CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是：让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。
    - CyclicBarrier 内部通过一个 count 变量作为计数器，count 的初始值为 parties 属性的初始化值，每当一个线程到了栅栏这里了，那么就将计数器减 1。如果 count 值为 0 了，表示这是这一代最后一个线程到达栅栏，就尝试执行我们构造方法中输入的任务。

