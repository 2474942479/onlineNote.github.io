# CAS

## CAS实现原子操作的三大问题

1. ### ABA问题

​	ABA问题，即一个值从A -> B -> A，这时候CAS检测不出来该值已经发生改变，但实际该值已经更新两次。

**解决方案**：采用**版本号/时间戳**的方式进行解决。从JDK 1.5开始，JDK的atomic包里提供了一个类`AtomicStampedReference`类的`compareAndSet`方法来解决ABA问题。

```java
/**
AtomicStampedReference维护一个对象引用以及一个可以自动更新的整数"标记"
实现说明：此实现通过创建表示"装箱"【引用，整数】对的内部对象来维护标记引用。
自从：1.5
作者：道格.利亚
类型参数：<V> - 此引用所引用的对象类型
*/
public class AtomicStampedReference<V> {
    private static class Pair<T> {
            final T reference;
            final int stamp;
            private Pair(T reference, int stamp) {
                this.reference = reference;
                this.stamp = stamp;
            }
            static <T> Pair<T> of(T reference, int stamp) {
                return new Pair<T>(reference, stamp);
            }
        }


    /**
    如果当前引用==到预期引用并且当前标记等于预期标记，则原子地将引用和标记的值设置为给定的更新值。
    参数：expectedReference-参考的预期值
         newReference-引用的新值
         expectedStamp-邮票的期望值
         newStamp-邮票的新值
    返回：如果成功则为true
    */

    public boolean compareAndSet(V   expectedReference,
                                     V   newReference,
                                     int expectedStamp,
                                     int newStamp) {
            Pair<V> current = pair;
            return
                expectedReference == current.reference &&
                expectedStamp == current.stamp &&
                ((newReference == current.reference &&
                  newStamp == current.stamp) ||
                 casPair(current, Pair.of(newReference, newStamp)));
        }
}
```

2. ### 循环时间长开销大

   CAS多与自旋结合。如果自旋CAS长时间不成功，会占用大量的CPU资源。

   解决思路是让JVM支持处理器提供的**pause指令**。

   pause指令能让自旋失败时cpu睡眠一小段时间再继续自旋，从而使得读操作的频率低很多,为解决内存顺序冲突而导致的CPU流水线重排的代价也会小很多。

3. ### 只能保证一个共享变量的原子操作

   这个问题你可能已经知道怎么解决了。有两种解决方案：

   - 使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作；
   - 使用锁。锁内的临界区代码可以保证只有当前线程能操作。

   # 