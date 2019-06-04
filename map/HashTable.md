### HashTable

- 同HashMap也是一个散列表，与HashMap相似
- 是Dictionary的子类，实现了Map，Cloneable，Serializable接口
- HashTable是线程安全的
- 锁的粒度为整个链表，在高并发时性能会很低

#### HashTable成员变量与初始值

```java
	//内部存储链表
    private transient Entry<?,?>[] table;
	//当前table长度
    private transient int count;
	//rehash阈值
    private int threshold;
	//负载因子
    private float loadFactor;
	//修改次数（fail-fast用）
    private transient int modCount = 0;
```

### HashTable构造函数

- public Hashtable(int initialCapacity, float loadFactor)

```java
public Hashtable(int initialCapacity, float loadFactor) {
		//参数校验
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);
		 //默认容量必须>0
        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        //与HashMap不同的是 HashTable直接在构造函数里初始化链表
        table = new Entry<?,?>[initialCapacity];
        //计算rehash阈值（容量与负载的积和Integer.MAX_VALUE - 8取小的）
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}
```

- public Hashtable()

```java
public Hashtable() {
		//空参构造函数会初始化链表长度为11的HashTable
        this(11, 0.75f);
}
```

- public Hashtable(Map<? extends K, ? extends V> t)

```java
public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        putAll(t);
}
public synchronized void putAll(Map<? extends K, ? extends V> t) {
        for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
            put(e.getKey(), e.getValue());
}
public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
}
private void addEntry(int hash, K key, V value, int index) {
        modCount++;

        Entry<?,?> tab[] = table;
        if (count >= threshold) {
            // Rehash the table if the threshold is exceeded
            rehash();

            tab = table;
            hash = key.hashCode();
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>) tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
}
```
