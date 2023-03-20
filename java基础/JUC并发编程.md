# JUC

```JAVA
public class TestRunTime {
    public static void main(String[] args) {
        //获取CPU的核数
        //CPU密集型，IO密集型
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}
```

**线程有几个状态**

- 新生
- 运行
- 阻塞
- 等待
- 超时等待
- 死亡（终止）

```java
public enum State {
    /**
     * Thread state for a thread which has not yet started.
     */
    NEW,

    /**
     * Thread state for a runnable thread.  A thread in the runnable
     * state is executing in the Java virtual machine but it may
     * be waiting for other resources from the operating system
     * such as processor.
     */
    RUNNABLE,

    /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     */
    BLOCKED,

    /**
     * Thread state for a waiting thread.
     * A thread is in the waiting state due to calling one of the
     * following methods:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     *
     * <p>A thread in the waiting state is waiting for another thread to
     * perform a particular action.
     *
     * For example, a thread that has called <tt>Object.wait()</tt>
     * on an object is waiting for another thread to call
     * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
     * that object. A thread that has called <tt>Thread.join()</tt>
     * is waiting for a specified thread to terminate.
     */
    WAITING,

    /**
     * Thread state for a waiting thread with a specified waiting time.
     * A thread is in the timed waiting state due to calling one of
     * the following methods with a specified positive waiting time:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     */
    TERMINATED;
}
```

## LOCK的生产者消费者模型

```java
public class B {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(()->{for(int i=0;i<10;i++)data.increment();},"A").start();
        new Thread(()->{for(int i=0;i<10;i++)data.decrement();},"B").start();
        new Thread(()->{for(int i=0;i<10;i++)data.increment();},"C").start();
        new Thread(()->{for(int i=0;i<10;i++)data.decrement();},"D").start();
    }

}

class Data{

    private int count = 0;

    Lock lock = new ReentrantLock();
    final Condition condition = lock.newCondition();

    public void increment(){
        lock.lock();
        try {
            //等待
            while (count!=0){
                condition.await();
            }
            count++;
            System.out.println(Thread.currentThread().getName()+"=>"+count);
            //通知
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decrement(){
        lock.lock();
        try {
            //等待
            while (count==0){
                condition.await();
            }
            count--;
            System.out.println(Thread.currentThread().getName()+"=>"+count);
            //通知
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

## Lock的多线程指定唤醒线程

```java
public class C {
    public static void main(String[] args) {
        Data3 data = new Data3();
        new Thread(()->{for (int i=0;i<10;i++)data.printA();},"A").start();
        new Thread(()->{for (int i=0;i<10;i++)data.printB();},"B").start();
        new Thread(()->{for (int i=0;i<10;i++)data.printC();},"C").start();
    }
}

class Data3{
    private int count = 1;

    private Lock lock = new ReentrantLock();
    private final Condition condition1 = lock.newCondition();
    private final Condition condition2 = lock.newCondition();
    private final Condition condition3 = lock.newCondition();

    public void printA(){
        lock.lock();
        try {
            while (count!=1){
                condition1.await();
            }
            count = 2;
            System.out.println(Thread.currentThread().getName()+"=>"+count);
            condition2.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void printB(){
        lock.lock();
        try {
            while (count!=2){
                condition2.await();
            }
            count = 3;
            System.out.println(Thread.currentThread().getName()+"=>"+count);
            condition3.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC(){
        lock.lock();
        try {
            while (count!=3){
                condition3.await();
            }
            count = 1;
            System.out.println(Thread.currentThread().getName()+"=>"+count);
            condition1.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

## ArrayList线程安全问题

```java
public class ListTest {
    public static void main(String[] args) {
        /*
           线程不安全的解决方案:1.使用vector，底层实现就是synchronized关键字
           2.使用List<Object> list1 = Collections.synchronizedList(new ArrayList<>());
           3.使用JUC下面的类 CopyOnWriteArrayList<Object> objects = new CopyOnWriteArrayList<>();
        * */
        List<String> list = new ArrayList<>();
        for (int i = 1; i <= 10; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
```

## Callable接口

```java
public class CallableTest {
    public static void main(String[] args) {
        try {
            //new Thread(new Runnable()).start();
            //new Thread(new FutureTask<V>()).start();
            //new Thread(new FutureTask<V>(Callable)).start();
            new Thread().start();
            MyThread thread = new MyThread();
            FutureTask<String> futureTask = new FutureTask<String>(thread);
            new Thread(futureTask,"A").start();
            new Thread(futureTask,"B").start(); //结果会被缓存，效率高，所以没有再执行call方法里面的操作
            String s = futureTask.get(); //这个get方法可能会产生阻塞，把它放到最后或者使用异步通信来处理！
            System.out.println(s);
//            String call = ((Callable<String>) () -> "666666").call();
//            System.out.println(call);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
class MyThread implements Callable<String>{

    @Override
    public String call() throws Exception {
        System.out.println("2022");
        return "6666";
    }
}
```

**重点**:call方法多次被调用时结果被缓存，不会执行里面的操作；get方法获取结果可能需要等待，有可能会阻塞。

## 常用的辅助类

### CountDwonLatch（减法计数器）

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        //总数是6, 必须要执行的任务的时候才执行
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"GO OUT");
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        countDownLatch.await();//等待计数器归0再向下执行
        System.out.println("close door");
        countDownLatch.countDown();//数量-1
    }
}
```

### CyclicBarrier（加法计数器）

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        /**
         * 集齐7颗龙珠召唤神龙
         */
        //召唤龙珠的线程
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙成功");
        });
        for (int i = 1; i <= 7; i++) {
            new Thread(()->{
                System.out.println(cyclicBarrier.getNumberWaiting());;
                System.out.println("收集了"+Thread.currentThread().getName()+"龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

### Semaphore

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        //线程数量，停车位,限流的时候可以使用
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                // acquire()得到
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release(); //release()释放
                }
            },String.valueOf(i)).start();
        }
    }
}
0抢到车位
1抢到车位
2抢到车位
0离开车位
1离开车位
3抢到车位
2离开车位
4抢到车位
5抢到车位
4离开车位
3离开车位
5离开车位
```

## 读写锁

```java
/**
 * 独占锁（写锁） 一次只能被一个线程占有，
 * 共享锁（读锁） 多个线程可以同时占有
 * ReadWriteLock
 * 读-读 可以共存
 * 读-写 不可以共存
 * 写-写 不可以共存
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCacheLock myCache = new MyCacheLock();
        for (int i = 1; i <= 50; i++) {
            new Thread(()->{
                myCache.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0,5));
            },String.valueOf(i)).start();
        }
        for (int i = 1; i <= 50; i++) {
            new Thread(()->{
                myCache.get(Thread.currentThread().getName());
            },String.valueOf(i)).start();
        }
    }
}

//加锁
class MyCacheLock{
    private volatile Map<String,Object> map = new HashMap<>();
    //读写锁，更加细粒度的控制
    private ReadWriteLock lock = new ReentrantReadWriteLock();
    //存，写；写入的时候只希望同时只有一个线程写，所有线程都可以去读
    public void put(String key,Object value){
        lock.writeLock().lock();//写锁，锁上
        try{
            System.out.println(Thread.currentThread().getName()+"写入"+key);
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()+"写入完毕");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.writeLock().unlock();
        }
    }
    public Object get(String key){
        lock.readLock().lock();
        try{
            System.out.println(Thread.currentThread().getName()+"读取"+key);
            return map.get(key);
        }catch (Exception e){
            return e.getMessage();
        }finally {
            lock.readLock().unlock();
        }
    }
}


//未加锁
class MyCache{
    private volatile Map<String,Object> map = new HashMap<>();

    //存，写
    public void put(String key,Object value){
        System.out.println(Thread.currentThread().getName()+"写入"+key);
        map.put(key,value);
        System.out.println(Thread.currentThread().getName()+"写入完毕");
    }
    public Object get(String key){
        System.out.println(Thread.currentThread().getName()+"读取"+key);
        return map.get(key);
    }
}
```

## 阻塞队列（Blocking Queue)

- ArrayBlockingQueue
- LinkedBlockingQueue
- SynchronousQueue

什么时候使用阻塞队列：多线程并发处理，线程池

### ArrayBlockingQueue

学会使用队列：添加、取出

四组API：

- 抛出异常
- 不会抛出异常
- 阻塞等待
- 超时等待

```java
/**
 * 抛出异常
 */
public class Test {
    public static void main(String[] args) {
        Test test = new Test();
        test.test1();
    }
    //抛出异常 添加：add();取出：remove()，返回队首元素：element()
    public void test1(){
        //队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        //Exception in thread "main" java.lang.IllegalStateException: Queue full
        //System.out.println(blockingQueue.add("d"));
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        //Exception in thread "main" java.util.NoSuchElementException
        //System.out.println(blockingQueue.remove());
    }
    	//不抛出异常; 添加：offer(),取出：poll(),返回队首元素：peek()
        public void tesst2(){
        //队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        //false
        //System.out.println(blockingQueue.offer("d"));
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        //null
        //System.out.println(blockingQueue.poll());
    }
    
     /**
     * 一直阻塞，添加：put()，取出：take()
     * @throws InterruptedException
     */
    public void test3() throws InterruptedException {
        //队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
        //大小只有3，添加完三个之后再添加d会一直阻塞
        blockingQueue.put("d");

        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        //没有元素之后还想要取就会一直阻塞
        System.out.println(blockingQueue.take());
    }
    
    
     /**
     * 超时队列
     * @throws InterruptedException
     */
    public void test4() throws InterruptedException {
        //队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);

        blockingQueue.offer("a");
        blockingQueue.offer("b");
        blockingQueue.offer("c");
        //2秒之内不能进入队列就放弃进入
        //blockingQueue.offer("d",2, TimeUnit.SECONDS);
        blockingQueue.poll();
        blockingQueue.poll();
        blockingQueue.poll();
        //2秒之内不能获取到队列内元素就放弃获取
        blockingQueue.poll(2,TimeUnit.SECONDS);
    }
}
```

### SynchronousQueue同步队列

特点：

- 没有容量，进去一个元素，就必须等待取出来之后，才能再往里面放一个元素！

操作：

- 添加：put
- 取：take

```java
/**
 * 同步队列：put了一个元素就必须从里面取出来才能再put进去。
 * 先T1线程put a，然后T2线程3秒之后取出来a；取出来了接着T1线程立即put b,然后T2线程又得等三秒之后取出来b
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        BlockingQueue<String> blockingQueue = new SynchronousQueue<>();

        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"put a");
                blockingQueue.put("a");
                System.out.println(Thread.currentThread().getName()+"put b");
                blockingQueue.put("b");
                System.out.println(Thread.currentThread().getName()+"put c");
                blockingQueue.put("c");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T1").start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T2").start();
    }
}
```