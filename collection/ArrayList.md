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
- fast-fail事件产生的条件：当多个线程对Collection进行操作时，
若其中某一个线程通过iterator去遍历集合时，该集合的内容被其他
线程所改变；则会抛出ConcurrentModificationException异常。

- 解决fail-fast办法：使用java.util.concurrent中对应类即可，
如：CopyOnWriteArrayList

- ArrayList fail-fast原理分析

1.在ArrayList的父类AbstractList中存在一个 modCount变量
```java
protected transient int modCount = 0;
```
这个变量会在ArrayList执行修改操作的时候modCount++

2.

```java
private class Itr implements Iterator<E> {
        int cursor;       //初始化为0 index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        //使用ArrayList的modCount
        int expectedModCount = modCount;

        Itr() {}
        /**
         * @param
         * @return
         * @Description 判断当前索引与数组大小是否相等
         * @auth like
         * @date 2019-05-29 00:39
         */
        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            //判断外部是否有线程修改了ArrayList
            checkForComodification();
            int i = cursor;
            //判断索引是否已经到达末尾
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            //加强判断当前索引会不会越界
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            //判断是否移动过索引
            if (lastRet < 0)
                throw new IllegalStateException();
            //判断外部是否有线程修改了ArrayList
            checkForComodification();
            //尝试移除当前索引对象，如果越界了则还是在外部有修改ArrayList
            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        //java8特性，使用Consumer消费item
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            //如果ArrayList的modCount与Itr的expectedModCount不相同了
            //说明外部有线程修改了ArrayList
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```




