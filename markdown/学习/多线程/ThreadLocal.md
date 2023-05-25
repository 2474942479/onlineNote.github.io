# ThreadLocal以及子类

![image-20210115110034515](../../markdown/图片/image-20210115110034515.png)

## Thread类

​	包含  threadLocals (当前线程私有的ThreadLocal)以及  inheritableThreadLocals  (当前线程可被继承的ThreadLocal)两个属性

​	

```java
	public	class Thread implements Runnable {
        
        
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * 与此线程相关的InheritableThreadLocal(可继承的ThreadLocal)值. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
        
    }
```

## InheritableThreadLocal  

​		设计目的: 当前线程的ThreadLocal能够被子线程使用

## TransmittableThreadLocal