# `ThreadLocal`线程数据隔离

每个线程私有的 本地线程变量，通过get和set方法就可以得到当前线程对应的值

`ThreadLocal`实例被其创建的类持有(更顶端应该是被线程持有),他们都位于堆中，通过一些技巧，将可见性修改成了私有性##

## 结构图

![img](file:///Users/zhangsongqi/markdown/%E5%9B%BE%E7%89%87/0c724fe36bb745c295d8384d9d52d85b~tplv-k3u1fbpfcp-watermark.image?lastModify=1628149759)

## JVM中内存图

![img](file:///Users/zhangsongqi/markdown/%E5%9B%BE%E7%89%87/330d2dfaeea140edb0921a3f721d5742~tplv-k3u1fbpfcp-watermark.image?lastModify=1628149759)

## 应用场景

- Spring 实现事务隔离级别，保证单个线程的数据库操作采用同一个数据库连接
- 多线程中，`SimpleDataFormat.parse()`内部有一个Calendar对象，parse时会先调用`Calendar.add()`，再调用`Calendar.remove()`,多线程并发操作下会产生线程安全问题，使用`ThreadLocal`进行解决
- cookie，session的数据隔离

## 内存泄露

根本原因：`ThreadLocalMap`的生命周期和Thread一样长，如果没有手动remove，除非线程结束，value值一直得不到回收，就会导致内存泄露

`ThreadLocalMap`中的key用弱引用，为什么不用强引用？

> `TheadLocal`对象被创建时，会产生两个引用，1是`ThreadLocal t = new ThreadLocal<>();`的强引用，2是Entry中，key是指向当前`ThreadLocal`的弱引用
>
> 但是设计时 key如果是使用强引用指向`ThreadLocal`，当想要回收`ThreadLocal`堆中内存时，会清除掉栈中指向`ThreadLocal`堆中内存的引用( 即t=null )，但由于`ThreadLocalMap`中Entry的key使用强引用指向了`ThreadLocal`，而`ThreadLocalMap`的生命周期和Thread一样长，如果没有手动remove，除非线程结束，`ThreadLocal`的内存一直得不到回收，就会导致内存泄露。

`ThreadLocalMap`中的key用弱引用，为什么也会导致内存泄露？

> 因为即使key被回收掉了，key=null，导致value将再也无法被访问到，就会导致内存泄露((因为这个时候`ThreadLocalMap`会存在key为null但是value不为null的Entry项)

解决办法：**使用完毕及时调用remove方法避免内存泄漏。**

## 实现

> **每个Thread维护着一个`ThreadLocalMap` 类的引用`threadlocals`**
>
> ```java
> ThreadLocal.ThreadLocalMap threadLocals = null;
> ```
>
> `ThreadLocalMap`是`ThreadLocal`的静态内部类 ，为每个Thread都维护了一个Entry类型的数组table(方便一个线程通过多个`ThreadLocal`实例对象关联多个值)，里面定义了一个Entry来保存数据，而且还是继承的弱引用。在Entry内部使用`ThreadLocal`实例对象作为key，使用我们设置的value作为value来保存数据.
>
> **采用开放地址法解决hash冲突**
>
> > ```java
> > static class ThreadLocalMap {
> > // 一个线程通过多个ThreadLocal实例对象关联多个值
> > private Entry[] table;
> > // 初始容量   必须是2的幂
> > private static final int INITIAL_CAPACITY = 16;
> > private int size = 0;
> > 
> > /**
> >  * 是继承自WeakReference的一个类，该类中实际存放的key是
> >  * 指向ThreadLocal的弱引用和与之对应的value值(该value值
> >  * 就是通过ThreadLocal的set方法传递过来的值)
> >  * 由于是弱引用，当get方法返回null的时候意味着坑能引用
> >  */    
> >  static class Entry extends WeakReference<ThreadLocal<?>> {
> >      /** 与该线程本地关联的值. */
> >      Object value;
> >      // 以ThreadLocal实例对象为k，使用我们设置的value作为value
> >      Entry(ThreadLocal<?> k, Object v) {
> >          super(k);
> >          value = v;
> >   }
> > }
> >     // super(k);
> >     //WeakReference构造方法(public class WeakReference<T> extends Reference<T> )
> >     public WeakReference(T referent) {
> >         super(referent); //referent：ThreadLocal的引用
> >     }
> > 
> >     //super(referent);调用Reference 一个参数的构造方法
> >     Reference(T referent) {
> >         this(referent, null);//referent：ThreadLocal的引用
> >     }
> >   // this(referent, null);调用当前类中两个参数的构造方法
> >     Reference(T referent, ReferenceQueue<? super T> queue) {
> >         this.referent = referent;
> >         this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
> >     }
> > 
> > ```
>
> #### **set方法   向`ThreadLocalMap`中设置值**
>
> > ```java
> > public void set(T value) {
> > //  获取当前线程
> >   Thread t = Thread.currentThread();
> > //  获取当前线程的ThreadLocalMap
> >   ThreadLocalMap map = getMap(t);
> > //  判断当前线程的ThreadLocalMap是否为null
> >   if (map != null)
> >       // 不是null，以ThreadLocal实例对象为k，关联的值为value
> >       map.set(this, value);
> >   else
> >       // TheadLocalMap为null，
> >       createMap(t, value);
> > }
> > ```
> >
> > **`getMap(t)` 获取当前线程的`ThreadLocalMap`**
> >
> > > ```java
> > > ThreadLocalMap getMap(Thread t) {
> > >      return t.threadLocals;
> > >  }
> > > ```
> >
> > **`map.set(this, value)`** 
> >
> > > ```java
> > > private void set(ThreadLocal<?> key, Object value) {
> > >  //获取到table,以及数组长度 
> > >  Entry[] tab = table;
> > >   int len = tab.length;
> > >   //通过对ThreadLocal实例对象hashCode与length位运算确定出一个索引值i
> > >   int i = key.threadLocalHashCode & (len-1);
> > >   //遍历tab,判断该索引值处的Entry对象是否存在
> > > 
> > >   for (Entry e = tab[i];
> > >        e != null;
> > >        e = tab[i = nextIndex(i, len)]) {
> > >       // 获取Entry的key
> > >       ThreadLocal<?> k = e.get();
> > >   //判断k与传进来的引用对象key是否一致 一致则更新值
> > >       if (k == key) {
> > >           e.value = value;
> > >           return;
> > >       }
> > >   // 不一致，判断Entry的key是否为null(ThreadLocal弱链接已被GC)，为null则初始化一个Entry替换旧的Entry
> > >       if (k == null) {
> > >           replaceStaleEntry(key, value, i);
> > >           return;
> > >       }
> > >   }
> > >   //没有遍历成功(1. Entry对象不存在 Entry=null 2.Entry！=null且k不为null，但是k不等于当前实例化的ThreadLocal,发生hash冲突)则找到一个空位,创建新值
> > >   tab[i] = new Entry(key, value);
> > >   int sz = ++size;
> > >   //满足条件数组扩容
> > >   if (!cleanSomeSlots(i, sz) && sz >= threshold)
> > >       rehash();
> > >  }
> > > ```
> > >
> > > **`Entry e = tab[i] ; e.get();` 获取table[i]处的Entry对象弱连接的`ThreadLocal`实例对象**
> > >
> > > > ```java
> > > > public T get() {
> > > >  return this.referent;
> > > > }
> > > > ```
> >
> > **`createMap(t, value);`**
> >
> > > ```java
> > > void createMap(Thread t, T firstValue) {
> > >  t.threadLocals = new ThreadLocalMap(this, firstValue);
> > > }
> > > ```
> > >
> > > **通过`ThreadLocalMap(this, firstValue)`构造函数 初始化并设置值**
> > >
> > > > ```java
> > > > ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
> > > >  // 初始化 大小为INITIAL_CAPACITY=16的Entry数组 table
> > > >  table = new Entry[INITIAL_CAPACITY];
> > > >  // 对ThreadLocal实例对象hashCode与length位运算确定出一个索引值i
> > > >  int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
> > > >  // 在table的索引位置设置一个Entry对象
> > > >  table[i] = new Entry(firstKey, firstValue);
> > > >  size = 1;
> > > >  setThreshold(INITIAL_CAPACITY);
> > > > }
> > > > ```
>
> #### **get()方法**
>
> > ```java
> > public T get() {
> >  // 获取当前线程
> >  Thread t = Thread.currentThread();
> >  // 获取当前线程的ThreadLocalMap
> >  ThreadLocalMap map = getMap(t);
> >  // 当前map不为空时
> >  if (map != null) {
> >      //获取当前实例化的 `ThreadLocal`对象对应的Entry对象
> >      ThreadLocalMap.Entry e = map.getEntry(this);
> >      // 对应的Entry对象不为null时，返回对应的value值
> >      if (e != null) {
> >          @SuppressWarnings("unchecked")
> >          T result = (T)e.value;
> >          return result;
> >      }
> >  }
> >  // 当前线程的map为空或者实例化的ThreadLocal对象无关联值时，初始化map并设置实例对象关联的value值null或者直接设置value值null 最终返回null
> >  return setInitialValue();
> > }
> > ```
> >
> > **`getEntry(this)`  获取当前实例化的 `ThreadLocal`对象对应的Entry对象**
> >
> > > ```java
> > > private Entry getEntry(ThreadLocal<?> key) {
> > >  int i = key.threadLocalHashCode & (table.length - 1);
> > >  Entry e = table[i];
> > >  // table[i]位置上的Entry不为null 且Entry的key等于实例化的ThreadLocal对象(key),直接返回该Entry
> > >  if (e != null && e.get() == key)
> > >      return e;
> > >  // table[i]位置上的Entry为null或者Entry的key不等于实例化的ThreadLocal对象,就
> > >  else
> > >      return getEntryAfterMiss(key, i, e);
> > > }
> > > 
> > > 
> > > // 
> > > private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
> > >  Entry[] tab = table;
> > >  int len = tab.length;
> > >  // 
> > >  while (e != null) {
> > >      ThreadLocal<?> k = e.get();
> > >      // 相等就直接返回
> > >      if (k == key)
> > >          return e;
> > >      if (k == null)
> > >          expungeStaleEntry(i);
> > >      // 不相等就继续查找，找到相等的位置
> > >      else
> > >          i = nextIndex(i, len);
> > >      e = tab[i];
> > >  }
> > >  return null;
> > > }
> > > ```
> >
> > **`setInitialValue();`给map设置初始值或者将实例化的`ThreadLocal`对象关联null值**
> >
> > > ```java
> > > private T setInitialValue() {
> > >  T value = initialValue();
> > >  Thread t = Thread.currentThread();
> > >  ThreadLocalMap map = getMap(t);
> > >  // map不为空，直接设置实例对象关联的value值null
> > >  if (map != null)
> > >      map.set(this, value);
> > >  else
> > >  // map为空，初始化map并设置实例对象关联的value值null
> > >      createMap(t, value);
> > >  return value;
> > > }
> > > ```
>
> #### **remove()**
>
> > ```java
> > public void remove() {
> >  ThreadLocalMap m = getMap(Thread.currentThread());
> >  if (m != null)
> >      m.remove(this);
> > }
> > ```
> >
> > **`m.remove(this);`**
> >
> > > ```java
> > > private void remove(ThreadLocal<?> key) {
> > >  Entry[] tab = table;
> > >  int len = tab.length;
> > >  int i = key.threadLocalHashCode & (len-1);
> > >  for (Entry e = tab[i];
> > >       e != null;
> > >       e = tab[i = nextIndex(i, len)]) {
> > >      if (e.get() == key) {
> > >          e.clear();
> > >          expungeStaleEntry(i);
> > >          return;
> > >      }
> > >  }
> > > }
> > > 
> > > // e.clear()
> > > public void clear() {
> > >      this.referent = null;
> > >  }
> > > 
> > > 
> > > private int expungeStaleEntry(int staleSlot) {
> > >          Entry[] tab = table;
> > >          int len = tab.length;
> > > 
> > >          // expunge entry at staleSlot
> > >          tab[staleSlot].value = null;
> > >          tab[staleSlot] = null;
> > >          size--;
> > > 
> > >          // Rehash until we encounter null
> > >          Entry e;
> > >          int i;
> > >          for (i = nextIndex(staleSlot, len);
> > >               (e = tab[i]) != null;
> > >               i = nextIndex(i, len)) {
> > >              ThreadLocal<?> k = e.get();
> > >              if (k == null) {
> > >                  e.value = null;
> > >                  tab[i] = null;
> > >                  size--;
> > >              } else {
> > >                  int h = k.threadLocalHashCode & (len - 1);
> > >                  if (h != i) {
> > >                      tab[i] = null;
> > > 
> > >                      // Unlike Knuth 6.4 Algorithm R, we must scan until
> > >                      // null because multiple entries could have been stale.
> > >                      while (tab[h] != null)
> > >                          h = nextIndex(h, len);
> > >                      tab[h] = e;
> > >                  }
> > >              }
> > >          }
> > >          return i;
> > >      }
> > > ```

 	