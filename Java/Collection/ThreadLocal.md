### ThreadLocal

1. ThreadLocal是线程局部变量，这些变量在每个线程是独立的，不受其他线程影响。

```java
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
        if (this instanceof TerminatingThreadLocal) {
            TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
        }
        return value;
    }
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

```



Q：介绍下ThreadLocal内部实现？

A：ThreadLocal含有get set方法。 利用ThreadlLocal中内部类ThreadLocalMap实现的。通过ThreadLocalMap中Entry实现存储。ThreadLocal作为key进行操作。

应用：

1. 链路追踪。将一次请求的信息例如：traceid存储在threadLocal中，其他信息通过日志方式记录，通过traceid串联起来。

2. spring事物。



Q: ThreadLocal 使用时会产生什么问题？

A： 内存泄漏。因为threadLocal是利用Thread中ThreadLocalMap进行存储的。ThreadLocal本身不存任何数据，只是最为key。而作为key的ThreadLocal是弱引用。当gc时，有可能会把ThreadLocalMap中key=ThreadLocal清除掉，这样会造成key为null的Entry。永远不会回收。

Q：为什么用弱引用？

A：首先使用强引用会出现ThreadLocal在线程运行期间一直不会回收，数据量多的情况会出现内存泄漏问题。而弱引用在gc会回收，在下次get set remove的时候自动清理key为null的value。



