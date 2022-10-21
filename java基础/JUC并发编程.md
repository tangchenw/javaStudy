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