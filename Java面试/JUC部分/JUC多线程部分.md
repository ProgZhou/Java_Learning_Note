# JUC多线程部分

### Java乐观锁和悲观锁

**悲观锁**

悲观锁的代表就是synchronized和Lock锁

+ 核心思想是，线程只有占有了锁，才能去操作共享变量，每次只有一个线程占锁成功，获取锁失败的线程，都得停下来等待
+ 线程从运行到阻塞，再从阻塞到唤醒，设计线程的上下文切换，如果频繁地发生，会影响性能
+ 实际上，线程在获取synchronized和Lock锁时，如果锁已经被占用，都会做几次重试操作，减少阻塞的机会

**乐观锁**

乐观锁的代表就是AtomicInteger，使用CAS操作来保证原子性

+ 其核心思想是，无需加锁，每次只有一个线程能成功修改共享变量，其他失败的线程不需要停止，不断重试直至成功
+ 由于线程一直运行，不需要阻塞，因此不涉及线程上下文切换
+ 它需要多核CPU支持，且线程数应该和CPU核数相差不大



### ConcurrentHashMap



### ThreadLocal

**ThreadLocal的作用**

ThreadLocal可以实现【资源对象】的线程隔离，让每个线程各用各的【资源对象】，避免争用引发的线程安全问题

ThreadLocal同时实现了线程内的资源共享

> 一般的多线程情况下解决共享资源的线程安全问题是通过加锁的方法实现的，不管是加synchronized悲观锁还是使用CAS乐观锁尝试
>
> ThreadLocal采用一种完全相反的解决思路，实现资源对象的隔离，也就是每个线程各用各的资源，将共享资源变得不共享

测试代码：

```java
public class TestThreadLocal {
    public static void main(String[] args) {
        test1();
        test2();
    }

    private static void test1() {
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                	//线程间获取的是不同的Connection对象
				log.debug("conn: {}", Utils.getConnection());
            }, "t" + (i + 1)).start();
        }
    }
    
    private static void test2() {
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                //线程内获取的是相同的Connection对象
			   log.debug("conn: {}", Utils.getConnection());
                log.debug("conn: {}", Utils.getConnection());
                log.debug("conn: {}", Utils.getConnection());
            }, "t" + (i + 1)).start();
        }
    }

    static class Utils {
        private static final ThreadLocal<Connection> tl = new ThreadLocal<>();

        public static Connection getConnection() {
            Connection conn = tl.get();   //再当前线程中获取资源
            if(conn == null) {
                try {
                    conn = DriverManager.getConnection("");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
                tl.set(conn);   //将资源存入当前线程
            }
            return conn;
        }
    }
}
```

**Thread的实现原理**

从代码上看，整个程序中只有一个ThreadLocal对象（使用static关键字修饰），但却能够实现资源的隔离

其实是每个线程内有一个ThreadLocalMap的成员变量，用来存储资源对象

```java
ThreadLocal.ThreadLocalMap threadLocals = null;   //ThreadLocalMap是ThreadLocal中的一个静态内部类
```

当线程调用ThreadLocal的set方法时，就是以ThreadLocal自己作为key，资源对象作为value，放入调用这个方法线程的ThreadLocalMap中

```java
//ThreadLocal中set方法的源码
public void set(T value) {
    Thread t = Thread.currentThread();  //当前线程
    ThreadLocalMap map = getMap(t);  //从当前线程中获取ThreadLocalMap
    if (map != null) {
        //将ThreadLocal对象作为键，待放入的资源作为value放入到线程的ThreadLocalMap中
        map.set(this, value);
    } else {
        //如果为空则新建一个ThreadLocalMap
        createMap(t, value);
    }
}

void createMap(Thread t, T firstValue) {
    //每个线程调用都会新建一个ThreadLocalMap
	t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

> 既然是Map就有相应的索引计算和扩容的规则，ThreadLocal内部的这个ThreadLocalMap也一样
>
> + 索引的计算是有规律的，加入第一个ThreadLocal时，hash值固定为0，之后每加入一个ThreadLocal会在前一个hash值的基础上加上一个固定的值，然后再计算桶下标
> + 扩容规则，第一次创建Map的时候，容量默认是16，当Map中的元素个数大于`capacity * 2 / 3`时，就会扩容，容量扩大为原来的两倍，并重新计算元素的桶下标
> + 当然，作为Map还需要考虑hash冲突的解决方案，ThreadLocalMap的解决方法较为简单，就单纯的拉链法，如果有冲突就寻找离冲突点最近的一个空闲位置，放入新元素，注意一点，ThreadLocalMap数组上不能出现链表的结构

当线程调用get方法时，就以ThreadLocal自己作为key，到当前线程的ThreadLocalMap中查找关联的资源值

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);  //获取当前线程中的ThreadLocalMap
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    //如果当前线程中的ThreadLocalMap还没有创建，则创建并将资源设置进去
    return setInitialValue();
}

//ThreadLocalMap中的getEntry方法
private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);  //计算桶下标
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

//setInitialValue方法
private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {  //如果已经有ThreadLocalMap了，直接设置值
            map.set(this, value);
        } else {
            //否则新建一个ThreadLocalMap并将资源put进去
            createMap(t, value);
        }
        if (this instanceof TerminatingThreadLocal) {
            TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
        }
        return value;
    }
```

**为什么ThreadLocalMap中的key（ThreadLocal）要设计为弱引用**

```java
//这是ThreadLocalMap中存储键值对的一个内部类，可以看到它继承自WeakReference
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);  //只有键是弱引用
        value = v;
    }
}
```

Thread可能需要长时间运行，如果key不再使用，需要在内存不足时释放其占用的内存

如果把key设置为强引用，那么在垃圾回收的时候，即使其余地方不再使用这些键值，GC也不会回收掉这部分内存，因为线程内部的Map还在，仍然在引用这些key，所以仍然是释放不掉这部分内存的

弱引用只要到垃圾回收的时候就会被回收，只要别的地方没有引用这些key，就会被垃圾回收器回收

但只会回收key，值仍然是强引用，不会被回收，所以这是ThreadLocalMap不同于其他Map的一种情况，key可以为null，值不为null

> 弱引用：描述一些非必须对象，被弱引用关联的对象只能生存到下一次垃圾收集发生为止，当GC开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象

**value的释放时机**

（1）调用get方法时，发现key为null，会顺便将这个位置上的value置为null

```java
//再来看get方法的最后返回值，这个方法就是如果在get方法中判断key为null时，
private T setInitialValue() {
    T value = initialValue();  //这个方法只返回null，即将value置为null
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);  //如果Map不为空，但key为null，执行set会把之前的value置为null
    } else {
        createMap(t, value);
    }
    if (this instanceof TerminatingThreadLocal) {
        TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
    }
    return value;
}
```

（2）调用set方法时，会使用启发式扫描，清除**临近**的key为null的值

但一般ThreadLocal使用时都会被设置为static的变量，一般不会被垃圾回收器回收，前两种情况发生的也比较少

（3）主动调用remove方法直接清除指定的key-value