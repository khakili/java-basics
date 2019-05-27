## String
### 1.String类分析
- String类是final的，也就是说这个类不能被继承，类里的成员方法也不能被重写。
- String类实现了java.io.Serializable, Comparable<String>, CharSequence
- private final char value[];<br>
使用了char数组保存保存字符串的，由于value为final，所以才有了字符串是不能修改的对象。
- String类中方法返回的都是新的String对象
### 2.String类与字符串常量池
```java
    String a ="abc";
    String b = "abc";
    System.out.println(a==b);//true(1)
    System.out.println("----------------------");
    String c = "abcabc";
    String d = "abc"+"abc";
    System.out.println(c==d);//true(2)
    System.out.println("----------------------");
    String e = a+b;
    System.out.println(c==e);//false(3)
    System.out.println("----------------------");
    String f = new String("abc");
    System.out.println(a==f);//false(4)
    System.out.println("----------------------");
    String g = new String("abc");
    System.out.println(g==f);//false(5)
```
- (1)在b变量初始化的时候a变量已经初始化了，在常量池中已经包含字符串abc，
这样b变量直接使用a变量的地址，所以输出true
- (2)d变量由于在编译期已经可以确定结果，所以c和d使用的是同一个常量池中的变量
- (3)编译期无法确定e的值
- (4)由于f是通过构造函数创建，会产生一个新的对象
- (5)两个对象都是由构造函数创建

### 3.为什么不要使用+拼接字符串

```java
    public static void main(String[] args) {
        String a ="abc";
        String b = "abc";
        String c = a+b;
    }
```
以上代码反编译结果如下：

```
 public static void main(java.lang.String[]);
    Code:
       0: ldc           #4                  // String abc
       2: astore_1
       3: ldc           #4                  // String abc
       5: astore_2
       6: new           #5                  // class java/lang/StringBuilder
       9: dup
      10: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
      13: aload_1
      14: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      17: aload_2
      18: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      21: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      24: astore_3
      25: return

```

显而易见，c变量是由两个StringBuilder，append出来后用toString()输出的，
相当于每使用+拼接一次字符串就会生成一个新的StringBuilder对象。

