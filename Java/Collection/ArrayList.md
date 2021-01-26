```java
private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Default initial capacity.
     */
		// 默认容量大小
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
		// 构造时当size==0，返回的数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
		// 构造时调用无参构造时，返回的数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
		// 存放元素的数组
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```

```java
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
      // 新容量=老容量 * 1.5
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity <= 0) { // 扩容的size不够
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)// 初始化第一次调用
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity; // 设置为所需最小容量
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE)
            ? Integer.MAX_VALUE
            : MAX_ARRAY_SIZE;
    }
```

Q: 谈谈ArrayList扩容？

A：1. 初始计算新容量为老容量的1.5倍。2. 如果新容量不够时，先判读是否第一次调用。3. 是否为达到容量最大阈值。

Q: ArrayList实现？

A：ArrayList底层数据结构利用数组，在无参构造方法中，数组初始化大小为0.当add的时候会扩容，初始扩容大小为10。其余每次扩容为原来的容量的1.5倍。

Q：ArrayList是线程安全的么？

A：当然不是。在ArrayList的add中并没有加入任何锁。所以在并发add的过程中会出现线程安全的问题。但可以利用CopyOnWriteArrayList来代替ArrayList解决线程安全问题。

Q：介绍下CopyOnWriteArrayList？

A：首先先介绍一下CopyOnWrite。 cow是计算机程序设计领域中的一种通用优化策略。当多个调用者访问相同资源时，每个调用者访问的是资源的副本。修改副本后，将副本覆盖到资源上。再来介绍下CopyOnWriteArrayList。CopyOnWriteArrayList是利用cow概念，当add时，CopyOnWriteArrayList会复制一个数组进行add，在去更新原数组。在操作过程中加入了synchronized来解决多个副本的问题。







