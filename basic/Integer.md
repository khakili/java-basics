## Integer
### 1.Integer类分析
- Integer是基本类型int的一个包装类
- Integer的初值为null，int为0
### 2.拆箱与装箱
- 拆箱，装箱是JDK5增加的一个新特性
```java
    Integer a = 1;//自动装箱,Integer.valueOf()
    int b = a;//自动拆箱，此时调用的为Integer.intValue()
```
### 3.Integer比较
```java
    Integer a = 100;
    Integer b = new Integer(100);
    System.out.println(a==b);//(1)
    System.out.println("----------------------");
    Integer c = 100;
    System.out.println(a==c);//(2)
    System.out.println("----------------------");
    Integer d = 128;
    Integer e = 128;
    System.out.println(d==e);//(3)
    System.out.println("----------------------");
    int f =128;
    System.out.println(e==f);//(4)
```

- (1)b是有构造函数生成，两个对象地址不同，输出false
- (2)由于Integer中有一个静态内部类IntegerCache，在-128，127
范围内，会使用缓存中的对象，输出true
- (3)同上，输出false
- (4)由于f为int为基本类型，变量e会自动拆箱，输出true
注意，由于对象与基本类型比较时，对象会自动拆箱，所以需要保证该对象
不是Null，避免抛出NPE异常