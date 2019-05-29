## LinkedList
### 1.LinkedList类简介

- LinkedList是一个双向链表
- 它作为堆栈，队列，从两头操作
- LinkedList也不是一个线程安全的类
- 实现了Cloneable接口，可以浅拷贝
 
### 2.LinkedList方法分析

- E get(int index)

```java
//获取index索引的item
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
//检查索引index是否越界
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
    throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
//获取节点的值
Node<E> node(int index) {
    // assert isElementIndex(index);
    //二分法 获取item（如果索引小于size的一半从头循环，否则从结尾倒过来循环）
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

- boolean add(E e)

```java
//新增item
public boolean add(E e) {
        linkLast(e);
        return true;
}
//将item追加到链表结尾
void linkLast(E e) {
		  //当前最后一个Node
        final Node<E> l = last;
        //插入新Node
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        //如果当前Node是Null则将新Node设为第一个节点，否则追加到最后一个Node的next
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        //集合大小++
        size++;
        //修改次数++
        modCount++;
}
//Node构造函数
Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
}
```

