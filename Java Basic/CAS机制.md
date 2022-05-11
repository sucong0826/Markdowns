### 示例

启动两个线程，在每个线程中让静态变量`count`循环累加100次。

```java
public class CASTest {
    public static int count = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                for (int j = 0; j < 100; j++) {
                    count++;
                }
            }).start();
        }

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("count=" + count);
    }
}
```

最终输出的结果不一定时200，因为以上的代码时非线程安全的，可能小于200，也可能等于200。可以加上`synchronized(CASTest.class)`进行同步操作来保证线程的安全。

使用`synchronized`可以解决线程安全的问题，但是会带来性能问题。`synchronized`关键字会让没有得到锁资源的线程进入**Blocked**状态，而后在争夺到锁资源后恢复为**Runnable**状态，这个过程中可能涉及到操作系统内核模式和用户模式的相互转换，代价比较高。

> 在一个给定的线程中，可能存在内核模式和用户模式的切换。

尽管Java在`sychronized`上做了很多优化，增加了从偏向锁到轻量级锁再到重量级锁的过度，但是在最终变化为重量级锁之后，性能仍然比较低。面对这种情况，我们可以使用Java中的原子操作类。所谓的原子操作类，指的是java.util.concurrent.atomic包下的类，如AtomicBoolean/AtomicInteger等，而这些Atomic类的底层正是用到了CAS机制。



### 什么是CAS？

CAS是Compare and Swap的缩写，简单理解为比较并且替换。

> 在CAS机制中，使用了3个基本操作数：
>
> - 内存地址V
> - 旧预期值A
> - 要修改的新值B
>
> 更新一个变量的时候，只有当变量的旧预期值A和内存地址V当中的实际值相同时，才会将内存地址V对应的值修改为B。



1. 在内存地址V中，存储着值为10的变量。

<img src="F:\Projects\Markdowns\pngs\Snipaste_2022-03-09_23-31-14.png" alt="初始状态" style="zoom:50%;" />



2. 此时，线程1想把变量的值增加1，对于线程1来说，旧的期望值A为10，要修改的新值B为11。

<img src="F:\Projects\Markdowns\pngs\Snipaste_2022-03-09_23-38-08.png" style="zoom:50%;" />

3. 线程1在要提交更新前，线程2先执行完成，将内存地址V中的值更新为11。

<img src="F:\Projects\Markdowns\pngs\Snipaste_2022-03-09_23-44-08.png" style="zoom:50%;" />

4. 此时，线程1继续执行，开始提交更新，首先进行旧期望值A与地址V中的值进行比较，如果相等才可以更新。发现不同，则提交失败。

5. 线程1重新获取当前地址V的值，并重新计算想要修改的值，此时对于线程1来说，旧的期望值A为11，而要修改的值B为12。这个**重新尝试的过程被称为自旋**。

<img src="F:\Projects\Markdowns\pngs\Snipaste_2022-03-09_23-48-49.png" style="zoom:50%;" />

6. 假设这一次没有其他线程修改地址V的值，线程1进行比较，发现旧期望值A与地址V上的值都为11，相同，执行自增加1，然后线程1进行交换，把地址V上的值替换为新值B，也就是12。

<img src="F:\Projects\Markdowns\pngs\Snipaste_2022-03-09_23-52-30.png" style="zoom:50%;" />



从思想上来说，`synchronized`属于悲观锁，悲观的认为程序中的并发情况严重，所以严防死守。CAS属于乐观锁，乐观的认为程序中的并发情况不那么严重，所以让线程不断的去重试更新。



### CAS的优点与缺点

- 缺点1：**CPU开销过大**
  - 在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复会给CPU带来很大的压力。
- 缺点2：**不能保证代码块的原子性**
  - CAS机制所能保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。
- 缺点3：ABA问题



### CAS的底层实现

我们先看下AtomicInteger中常用的`incrementAndGet`方法的实现：

```java
public final int incrementAndGet() {
     for(;;) {
          int current = get();
          int next = current + 1;
          if (compareAndSet(current, next)) {
               return next;
          }
     }
}

private volatile int value;
public final int get() {
     return value;
}
```

这段代码是一个无限循环，也就是CAS的自旋，循环中做了三个事情：

1. 获取当前值（旧预期值A）
2. 当前值 + 1， 计算出目标值B
3. 进行CAS操作，如果成功则跳出循环；如果失败，则重复上述步骤

`get()`方法获取当前变量的返回值。如果保证获取的当前值是内存中的最新值？很简单，用`volatile`关键字来保证（保证线程间的可见性）。

```java
public final boolean compareAndSet(int expect, int update) {
     return unsafe.compareAndSwapInt(this, valueOffset, expect, update)
}

private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
     try {
          valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
     } catch(Exception e) {
          ...
     }
}
```

`Unsafe`类是JVM为应用层提供了一个可以直接访问底层操作系统的后门，且它为我们提供了**硬件级别的原子操作**。至于`valueOffset`值，是通过`unsafe.objectFieldOffset()`方法得到的，所代表的是AtomicInteger对象value成员变量所在内存中的偏移量，**可以简单的把`valueOffset`理解为`value`变量的内存地址**。

而`unsafe.compareAndSwapInt()`方法的参数包括了这三个基本元素：

- `valueOffset`：代表了变量的内存地址V
- `expect`：代表了旧预期值A
- `update`：代表了新的更新值B

正是`unsafe.compareAndSwapInt(...)`保证了compare和swap操作的原子性操作。



##### 参考：

https://blog.csdn.net/qq_32998153/article/details/79529704?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164675119516780366544976%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164675119516780366544976&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-79529704.pc_search_result_control_group&utm_term=CAS&spm=1018.2226.3001.4187