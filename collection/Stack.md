### Stack

- Stack是一个实现后进先出的对象（LIFO)
- Stack继承自Vector

#### Stack API 

```java
//判断Stack是否为空
boolean empty()
//获取Stack最后一个元素
synchronized E peek()
//获取Stack最后一个元素，并删除它
synchronized E pop()
//添加一个元素到Stack末尾
E push(E object)
//获取 item 在Stack中的索引
synchronized int search(Object o)
```
