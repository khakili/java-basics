## ArrayList
### 1.ArrayList类简介

- ArrayList是一个动态数组，是数组的一个复杂版本，提供了动态添加，删除
item的方法。
- ArrayList不是一个线程安全的类
- ArrayList实现了RandomAccess接口，提供了随机访问的功能，分为
快速随机访问和通过Iterator迭代器访问

### 2.ArrayList方法分析

- add(E e)

1.判断集合是否为空，为空初始化容量为10

2.判断使用最小容量减去当前数组的长度是否大于零，
如果大于零则表示需要扩容，反之当前容量不需要扩容

3.扩容方法，扩容后容量为当前的1.5倍

4.将e放到数组size++的位置

- add(int index, E element)

1.检查index是否在集合大小内

2.同add(E e)第2，3步

3.将element放到数组index位置

4.size++

- 3.remove(int index)

1.判断传入索引是否在集合范围内

2.取出指定索引的item

3.判断删除元素是否为最后一个，若不是则拷贝index+1及以后元素到index位置

4.将数组最后一个元素设为null，帮助GC

5.返回第2步中取出item

### ArrayList fail-fast分析