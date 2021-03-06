# Volatile关键字

> * **禁止指令重排序**(运行时，添加lock前缀指令，相当于添加了内存屏障 ) **保证了有序性**
>
>   >**重排序操作不会对存在数据依赖关系的操作进行重排序**
>   >
>   >比如：a=1;b=a; 这个指令序列，由于第二个操作依赖于第一个操作，所以在编译时和处理器运行时这两个操作不会被重排序。
>   >
>   >**重排序是为了优化性能，但是不管怎么重排序，单线程下程序的执行结果不能被改变**
>
> * **保证线程之间的可见性，不保证原子性**
>
>   **原理底层**：
>
>   ​	变量经过volatile修饰后，对此变量进行写操作时，汇编指令中会有一个**LOCK前缀指令**，加了这个**lock前缀指令**后，会引发两件事情：
>
>   * **当写一个volatile变量时，JVM会把该线程本地内存中的变量强制刷新到主内存中去 ；**在刷新到主内存的过程中，会锁定内存区域内的缓存(缓存行锁定),直到写入完成。
>
>    * **这个写会操作会导致其他线程中的缓存无效（原因：lock前缀指令会触发总线的缓存一致性协议(MESI)和总线嗅探机制）**
>
>   ​	对volatile变量进行读操作时，由于线程中的缓存已经失效，会到主内存中读取
>
>      > 在多处理器下，为了保证各个处理器的缓存一致性，就会实现**缓存一致性协议**，每个处理器通过**嗅探总线**上传播的数据来检查自己缓存是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就将当前处理器的缓存行设置为无效状态，当处理器对这个数据进行修改操作时，会重新从内存系统中把数据读到处理器缓存里。
>
>   **意思就是说当一个共享变量被volatile修饰时，它会保证 修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值，这就保证了可见性。**

# Volatile原理

volatile可以保证线程可见性且提供了一定的有序性，但是无法保证原子性。在JVM底层volatile是采用“内存屏障”来实现的。观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令，lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：

- 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
- 它会强制将对缓存的修改操作立即写入主存；
- 如果是写操作，它会导致其他CPU中对应的缓存行无效。



