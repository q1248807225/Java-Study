### LRU LinkedHashMap 实现

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LRUTest<K, V> {

    private LinkedHashMap<K, V> cache = null;
    private int cacheSize;

    public LRUTest() {
        this(16);
    }

    public LRUTest(int x) {
        this.cacheSize = (int)Math.ceil(cacheSize / 0.75f) + 1;
        // accessOrder = true;在访问元素后会调用afterNodeAccess() 将访问元素放在末尾
        cache = new LinkedHashMap<K, V>(this.cacheSize, 0.75f, true){
            // size > cacheSize 移除头
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > cacheSize;
            }
        };
    }

    public V get(K k) {
        return cache.get(k);
    }

    public void put(K k, V v) {
        cache.put(k, v);
    }
}

```

