### HashMap

- HashMap是一个散列表，存储内容是key映射value
- HashMap是Map，Cloneable，Serializable的实现类，是AbstractMap的子类
- HashMap不是一个线程安全的类，HashMap中的映射不能保证是有序的
- Key和Value都可以为Null

#### HashMap成员变量与初始值

```java
  //默认初始化容量（必须是2的幂）
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
  //最大容量（2的幂并且小于等于2的30次幂）
  static final int MAXIMUM_CAPACITY = 1 << 30;
  //默认负载因子
  static final float DEFAULT_LOAD_FACTOR = 0.75f;
  //链表转换成红黑树的阈值
  static final int TREEIFY_THRESHOLD = 8;
  //红黑树退化成链表的阈值
  static final int UNTREEIFY_THRESHOLD = 6;
  //链表转红黑树的阈值（为了避免在哈希表建立初期，多个键值对恰好被放入了同一个链表中而导致不必要的转化。）
  static final int MIN_TREEIFY_CAPACITY = 64;
  //HashMap内部链表
  transient Node<K,V>[] table;
  //价值对集合
  transient Set<Map.Entry<K,V>> entrySet;
  //HashMap 大小
  transient int size;
  //HashMap修改次数
  transient int modCount;
  //可存储容量
  int threshold;
  //负载因子
  final float loadFactor;
```

#### HashMap构造函数

- public HashMap(int initialCapacity, float loadFactor)

```java
public HashMap(int initialCapacity, float loadFactor) {
		 //参数校验
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
		 //设置容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //校验负载因子
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
}
//计算阈值(传入cap的最小2的幂)
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

- public HashMap(Map<? extends K, ? extends V> m)

```java
public HashMap(Map<? extends K, ? extends V> m) {
		 //设置默认负载因子
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
}
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        //传入map有K-V
        if (s > 0) {
        	  //如果当前HashMap没有放过值
            if (table == null) { // pre-size
            	//计算负载因子
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
				//
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            //传入map大小大于当前HashMap大小
            else if (s > threshold)
            //重新计算size
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                //放Key Value 
                putVal(hash(key), key, value, false, evict);
            }
        }
}
//重新计算HasMap内部结合大小
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
        	//当旧map容量已经到达最大值是 不在扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //扩容两倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
        //默认初始化的时候使用DEFAULT_INITIAL_CAPACITY（16）
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //初始化新的内部数组
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
	        //将每个元素移动到新的哈希桶中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                    //如果oldTab的第j个元素的next为空，则直接复制到新数组中
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                    //红黑树节点直接放入
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            //计算是否为0或者1
                            if ((e.hash & oldCap) == 0) {
                            //为0，放入原来的位置
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                            //为1，放入原来位置+oldCap
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                        //放入原来的位置
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                        //放入原来位置+oldCap
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
    //实际初始化HashMap中的Node数组
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //步骤1：table为null则创建table
        if ((tab = table) == null || (n = tab.length) == 0)
        //table为null，执行resize()方法
            n = (tab = resize()).length;
        //步骤2：当前索引下元素为null，直接插入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
        //步骤3：节点存在，直接覆盖。
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
     	  //步骤4：判断时候为红黑树结构
            else if (p instanceof TreeNode)
            //红黑树插入元素
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
            //步骤5：将元素出入链表
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                         //链表长度大于8时转化为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //节点已存在，则覆盖
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //覆盖已存在节点的逻辑 HashMap 没有此处逻辑
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
         //步骤6：判断是否要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
}
```

- HashMap的hash算法

> （一）高16 bit 不变，低16 bit 和高16 bit 做了一个异或（得到的 hashcode 转化为32位二进制，前16位和后16位低16 bit和高16 bit做了一个异或）。

>（二）(n-1) & hash = -> 得到下标。

```java
static final int hash(Object key) {
        int h;
        //if(key==null){
        //key为null直接返0
        //	return 0;
        //}else{
        //   计算key的hashCode
        //	h = key.hashCode();
        //   
        //	return h^h>>>16;
        //}
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```



- HashMap的fail-fast(原理同ArrayList)

> 如果在使用迭代器的过程中有其他线程修改了map，那么将抛出ConcurrentModificationException。

- 为什么 String、Integer 这样的 wrapper 类适合作为键

> 因为 String 是 final，而且已经重写了 equals() 和 hashCode() 方法了。不可变性是必要的，因为为了要计算 hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的 hashcode 的话，那么就不能从 HashMap 中找到你想要的对象。

- 拉链法导致的链表过深，为什么不用二叉查找树代替而选择红黑树？为什么不一直使用红黑树？

> 之所以选择红黑树是为了解决二叉查找树的缺陷：二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成层次很深的问题），遍历查找会非常慢。而红黑树在插入新数据后可能需要通过左旋、右旋、变色这些操作来保持平衡。引入红黑树就是为了查找数据快，解决链表查询深度的问题。我们知道红黑树属于平衡二叉树，为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少。所以当长度大于8的时候，会使用红黑树；如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢。
