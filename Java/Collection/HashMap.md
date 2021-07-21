``` java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    private static final long serialVersionUID = 362498820763181265L;

    /*
     * Implementation notes.
     *
     * This map usually acts as a binned (bucketed) hash table, but
     * when bins get too large, they are transformed into bins of
     * TreeNodes, each structured similarly to those in
     * java.util.TreeMap. Most methods try to use normal bins, but
     * relay to TreeNode methods when applicable (simply by checking
     * instanceof a node).  Bins of TreeNodes may be traversed and
     * used like any others, but additionally support faster lookup
     * when overpopulated. However, since the vast majority of bins in
     * normal use are not overpopulated, checking for existence of
     * tree bins may be delayed in the course of table methods.
     *
     * Tree bins (i.e., bins whose elements are all TreeNodes) are
     * ordered primarily by hashCode, but in the case of ties, if two
     * elements are of the same "class C implements Comparable<C>",
     * type then their compareTo method is used for ordering. (We
     * conservatively check generic types via reflection to validate
     * this -- see method comparableClassFor).  The added complexity
     * of tree bins is worthwhile in providing worst-case O(log n)
     * operations when keys either have distinct hashes or are
     * orderable, Thus, performance degrades gracefully under
     * accidental or malicious usages in which hashCode() methods
     * return values that are poorly distributed, as well as those in
     * which many keys share a hashCode, so long as they are also
     * Comparable. (If neither of these apply, we may waste about a
     * factor of two in time and space compared to taking no
     * precautions. But the only known cases stem from poor user
     * programming practices that are already so slow that this makes
     * little difference.)
     *
     * Because TreeNodes are about twice the size of regular nodes, we
     * use them only when bins contain enough nodes to warrant use
     * (see TREEIFY_THRESHOLD). And when they become too small (due to
     * removal or resizing) they are converted back to plain bins.  In
     * usages with well-distributed user hashCodes, tree bins are
     * rarely used.  Ideally, under random hashCodes, the frequency of
     * nodes in bins follows a Poisson distribution
     * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
     * parameter of about 0.5 on average for the default resizing
     * threshold of 0.75, although with a large variance because of
     * resizing granularity. Ignoring variance, the expected
     * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
     * factorial(k)). The first values are:
     *
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
     * more: less than 1 in ten million
     *
     * The root of a tree bin is normally its first node.  However,
     * sometimes (currently only upon Iterator.remove), the root might
     * be elsewhere, but can be recovered following parent links
     * (method TreeNode.root()).
     *
     * All applicable internal methods accept a hash code as an
     * argument (as normally supplied from a public method), allowing
     * them to call each other without recomputing user hashCodes.
     * Most internal methods also accept a "tab" argument, that is
     * normally the current table, but may be a new or old one when
     * resizing or converting.
     *
     * When bin lists are treeified, split, or untreeified, we keep
     * them in the same relative access/traversal order (i.e., field
     * Node.next) to better preserve locality, and to slightly
     * simplify handling of splits and traversals that invoke
     * iterator.remove. When using comparators on insertion, to keep a
     * total ordering (or as close as is required here) across
     * rebalancings, we compare classes and identityHashCodes as
     * tie-breakers.
     *
     * The use and transitions among plain vs tree modes is
     * complicated by the existence of subclass LinkedHashMap. See
     * below for hook methods defined to be invoked upon insertion,
     * removal and access that allow LinkedHashMap internals to
     * otherwise remain independent of these mechanics. (This also
     * requires that a map instance be passed to some utility methods
     * that may create new nodes.)
     *
     * The concurrent-programming-like SSA-based coding style helps
     * avoid aliasing errors amid all of the twisty pointer operations.
     */

    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

Q: hashMap的初始容量为什么时16.

A: DEFAULT_INITIAL_CAPACITY为16， hash因用位运算。特定2^n。所以map容量需要是2^n。而16应该是比较合适的大小。



MAXIMUM_CAPACITY最大数组元素数量，2^30。



DEFAULT_LOAD_FACTOR 负载因子。默认为0.75

Q: 为什么不是1 或者 0.5？

1. 当负载因子为1时，将数组填充满才会发生扩容，虽然提高了空间利用率，但会出现很多链表已经形成红黑树的情况。会增加查询时间增大。
2. 当负载因子为0.5时，当hash后的值超过length/2.发生扩容，频繁的扩容使耗时降低，但有相当多的空间浪费。

负载因子的值时在hashCode分布较好时，降低tree结构产生。当负载因子为0.75时，桶中节点的分布频率服从参数为0.5的泊松分布。

[What is the significance of load factor in HashMap?](https://link.jianshu.com/?t=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F10901752%2Fwhat-is-the-significance-of-load-factor-in-hashmap)

计算0.5的泊松分布。

Q：为什么HashMap桶中元素个数大于8转换成红黑树？

A： TreeNode占用空间是Node的两倍。所以尽量不希望Node变成TreeNode。而随着链表越来越长。查询时间复杂度为O(n)，需要降低耗时，将Node转换成TreeNode。查询时间复杂度为O(log(n)). hash碰撞观察8层节点个概率为0.00000006。

Q: 为什么HashMap树的退化的阈值是6?

A：首先退化阈值从8递减。 阈值为8的情况：建树与退化循环进行。阈值为7的情况：当树中有8个节点时，删除一个节点就进行退化，增加一个节点就进行建树。频繁的变换形态会浪费大量的时间，降低性能。

Q：有没有大于TREEIFY_THRESHOLD时，不建树的情况？

A： 有！当容量小于MIN_TREEIFY_CAPACITY时，仍然使用链表形式存储。

```java
   final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)// 初始化数组
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) //当前桶无节点创建节点
            tab[i] = newNode(hash, key, value, null);
        else {// hash冲突
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))//k与节点相同
                e = p;
            else if (p instanceof TreeNode)//如果节点是TreeNode将value put进去
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {//链表赋值
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st 阈值大于7，建树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))//与put中key的hash值相同，跳出
                        break;
                    p = e;
                }
            }
            if (e != null) { // 如果存在key更新value
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)// 重散列
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```



Q: 说下put？

A： 1.如果未put，首先初始化数组。2.当前桶无元素，新建节点。3.1 当前节点key与put的key相同获取节点。3.2 当前node是treeNode，putTreeVal 3.3 循环链表，找到最后一个节点或者与put的key相同的节点并获取。4. 更新节点。5. 若是size大于阈值，重散列。

```java
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {// 表不为空，桶不为空
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))//桶中第一个是否匹配
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)//是否为树结构
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);//循环链表获取节点
            }
        }
        return null;
    }
```



Q：说下get？

A：1.判断表、桶是否为空。2. 判断桶中第一个元素是否匹配。3.判断是否为书结构。4. 循环链表或者查找树获取节点。



Q：hashMap有什么线程安全问题？

A：1.7 会有成环问题。因为put时先扩容再插入。1.8会有数据丢失问题。因为put时先插入后扩容。

Q：hashMap put时，元素的位置？

A： 1.7 插头。1.8插尾。

