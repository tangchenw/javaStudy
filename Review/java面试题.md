# Java面试题

将字符串反转使用 StringBuilder 或者 stringBuffer 的 reverse() 方法。

**普通类和抽象类有哪些区别：**

普通类不能包含抽象方法，抽象类可以包含抽象方法。

抽象类不能直接实例化，普通类可以直接实例化。

**接口和抽象类有什么区别：**

实现：抽象类的子类使用 extends 来继承；接口必须使用 implements 来实现接口。

构造函数：抽象类可以有构造函数；接口不能有。

main 方法：抽象类可以有 main 方法，并且我们能运行它；接口不能有 main 方法。

实现数量：类可以实现很多个接口；但是只能继承一个抽象类。

访问修饰符：接口中的方法默认使用 public 修饰；抽象类中的方法可以是任意访问修饰符。

字节流和字符流的区别是：字节流按 8 位传输以字节为单位输入输出数据，字符流按 16 位传输以字符为单位输入输出数据。

**BIO、NIO、AIO 有什么区别？**

BIO：Block IO 同步阻塞式 IO，就是我们平常使用的传统 IO，它的特点是模式简单使用方便，并发处理能力低。

NIO：New IO 同步非阻塞 IO，是传统 IO 的升级，客户端和服务器端通过 Channel（通道）通讯，实现了多路复用。

AIO：Asynchronous IO 是 NIO 的升级，也叫 NIO2，实现了异步非堵塞 IO ，异步 IO 的操作基于事件和回调机制。

**Files的常用方法都有哪些？**

- Files.exists()：检测文件路径是否存在。
- Files.createFile()：创建文件。
- Files.createDirectory()：创建文件夹。
- Files.delete()：删除一个文件或目录。
- Files.copy()：复制文件。
- Files.move()：移动文件。
- Files.size()：查看文件个数。
- Files.read()：读取文件。
- Files.write()：写入文件。

**HashMap 和 Hashtable 有什么区别？**

hashMap去掉了HashTable 的contains方法，但是加上了containsValue（）和containsKey（）方法。

hashTable同步的，而HashMap是非同步的，效率上逼hashTable要高。

hashMap允许空键值，而hashTable不允许。

**ArrayList 和 LinkedList 的区别是什么？**

最明显的区别是 ArrrayList底层的数据结构是数组，支持随机访问，而 LinkedList 的底层数据结构是双向循环链表，不支持随机访问。
使用下标访问一个元素，ArrayList 的时间复杂度是 O(1)，而 LinkedList 是 O(n)。

**如何实现数组和 List 之间的转换？**

List转换成为数组：调用ArrayList的toArray方法。

数组转换成为List：调用Arrays的asList方法。

**Array 和 ArrayList 有何区别？**

Array可以容纳基本类型和对象，而ArrayList只能容纳对象。 

Array是指定大小的，而ArrayList大小是固定的。 

Array没有提供ArrayList那么多功能，比如addAll、removeAll和iterator等。

**迭代器 Iterator 是什么？**

迭代器是一种设计模式，它是一个对象，它可以遍历并选择序列中的对象，而开发人员不需要了解该序列的底层结构。
迭代器通常被称为“轻量级”对象，因为创建它的代价小。

**Iterator 怎么使用？有什么特点？**

Java中的Iterator功能比较简单，并且只能单向移动：

1. 使用方法iterator()要求容器返回一个Iterator。第一次调用Iterator的next()方法时，它返回序列的第一个元素。
   注意：iterator()方法是java.lang.Iterable接口,被Collection继承。
2. 使用next()获得序列中的下一个元素。
3. 使用hasNext()检查序列中是否还有元素。
4. 使用remove()将迭代器新返回的元素删除。

Iterator是Java迭代器最简单的实现，为List设计的ListIterator具有更多的功能，它可以从两个方向遍历List，也可以从List中插入和删除元素。

Iterator可用来遍历Set和List集合，但是ListIterator只能用来遍历List。 

**通过Callable和Future创建线程：**

创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。

创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。

使用FutureTask对象作为Thread对象的target创建并启动新线程。

调用FutureTask对象的get()方法来获得子线程执行结束后的返回值。

**runnable 和 callable 有什么区别？**

Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已；

Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果。

**sleep() 和 wait() 有什么区别？**

sleep()：方法是线程类（Thread）的静态方法，让调用线程进入睡眠状态，让出执行机会给其他线程，
等到休眠时间结束后，线程进入就绪状态和其他线程一起竞争cpu的执行时间。因为sleep() 是static静态的方法，
他不能改变对象的机锁，当一个synchronized块中调用了sleep() 方法，线程虽然进入休眠，但是对象的机锁没有被释放，
其他线程依然无法访问这个对象。

wait()：wait()是Object类的方法，当一个线程执行到wait方法时，它就进入到一个和该对象相关的等待池，
同时释放对象的机锁，使得其他线程能够访问，可以通过notify，notifyAll方法来唤醒等待的线程。

**notify()和 notifyAll()有什么区别？**

如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。

当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争。

优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。

**ThreadLocal 是什么？有哪些使用场景？**

Java提供ThreadLocal类来支持线程局部变量，是一种实现线程安全的方式。

**说一下 synchronized 底层实现原理？**

synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性。

Java中每一个对象都可以作为锁，这是synchronized实现同步的基础：普通同步方法，锁是当前实例对象;静态同步方法，锁是当前类的class对象;同步方法块，锁是括号里面的对象

**synchronized 和 volatile 的区别是什么？**

volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的。

**synchronized 和 Lock 有什么区别？**

首先synchronized是java内置关键字，在jvm层面，Lock是个java类；synchronized无法判断是否获取锁的状态，Lock可以判断是否获取到锁；synchronized会自动释放锁(a 线程执行完同步代码会释放锁 ；b 线程执行过程中发生异常会释放锁)，Lock需在finally中手工释放锁（unlock()方法释放锁），否则容易造成线程死锁；

用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去，而Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了；

synchronized的锁可重入、不可中断、非公平，而Lock锁可重入、可判断、可公平（两者皆可）；

Lock锁适合大量同步的代码的同步问题，synchronized锁适合代码少量的同步问题。

**什么是反射？**

反射主要是指程序可以访问、检测和修改它本身状态或行为的一种能力

**Java反射：**

在Java运行时环境中，对于任意一个类，能否知道这个类有哪些属性和方法？对于任意一个对象，能否调用它的任意一个方法
Java反射机制主要提供了以下功能：

- 在运行时判断任意一个对象所属的类。
- 在运行时构造任意一个类的对象。
- 在运行时判断任意一个类所具有的成员变量和方法。
- 在运行时调用任意一个对象的方法。 

**什么是 java 序列化？什么情况下需要序列化？**

简单说就是为了保存在内存中的各种对象的状态（也就是实例变量，不是方法），并且可以把保存的对象状态再读出来。虽然你可以用你自己的各种各样的方法来保存object states，但是Java给你提供一种应该比你自己好的保存对象状态的机制，那就是序列化。

什么情况下需要序列化：

1. 当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；
2. 当你想用套接字在网络上传送对象的时候；
3. 当你想通过RMI传输对象的时候；

**动态代理是什么？有哪些应用？**

当想要给实现了某个接口的类中的方法，加一些额外的处理。比如说加日志，加事务等。

可以给这个类创建一个代理，故名思议就是创建一个新的类，这个类不仅包含原来类方法的功能，而且还在原来的基础上添加了额外处理的新类。这个代理类并不是定义好的，是动态生成的。具有解耦意义，灵活，扩展性强。
动态代理的应用：

- Spring的AOP
- 加事务
- 加权限
- 加日志

**怎么实现动态代理？**

首先必须定义一个接口，还要有一个InvocationHandler(将实现接口的类的对象传递给它)处理类。再有一个工具类Proxy(习惯性将其称为代理类，因为调用他的newInstance()可以产生代理对象,其实他只是一个产生代理对象的工具类）。利用到InvocationHandler，拼接代理类源码，将其编译生成代理类的二进制码，利用加载器加载，并将其实例化产生代理对象，最后返回。

**为什么要使用克隆？**

想对一个对象进行处理，又想保留原有的数据进行接下来的操作，就需要克隆了，Java语言中克隆针对的是类的实例。