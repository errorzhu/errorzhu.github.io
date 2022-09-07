# java基础

## String

### 原理

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
......
```
实际存储的是char数组
ps: java中的char占2个字节,unicode(码点范围U+0000到U+FFFF，不包含替换区域)用char表示

### equals()方法

```
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
```
String 重写了Object的equals方法,先判断被比较者是否是string类型,再转换成char数组依次比较


### 问题

1. == 和 equals 的区别
== 对于基本数据类型来说，是用于比较 "值"是否相等的；而对于引用类型来说，是用于比较引用地址是否相同的。
Object 中的 equals() 方法其实就是 ==，而 String 重写了 equals() 方法把它修改成比较两个字符串的值是否相等

2. String类 为什么被final修饰
使用 final 修饰的第一个好处是安全；第二个好处是高效，相同的字符串值可以在jvm中共享常量池中的值

3. String 和 StringBuilder、StringBuffer 的区别
因为 String 类型是不可变的，所以在字符串拼接的时候如果使用 String 的话性能会很低。
StringBuffer，它提供了 append 和 insert 方法可用于字符串的拼接，它使用 synchronized 来保证线程安全。
StringBuilder，它提供了 append 和 insert 方法可用于字符串的拼接,但没有加锁,线程不安全但性能更好。

4. 直接赋值和new String()
直接赋值: 先去字符串常量池中查找是否已经有此值，如果有则把引用地址直接指向此值，否则会先在常量池中创建，然后再把引用指向此值
new的方式: 先在堆上创建String对象,然后再去常量池中查询此字符串的值是否已经存在，如果不存在会先在常量池中创建此字符串

```
String hello = new String("hello");
String hello2 = new String("hello");
String hello3 = "hello";
String hello4 = hello.intern();
String hello5 = hello2.intern();
System.out.println(hello == hello2); //false
System.out.println(hello == hello3); //false
System.out.println(hello3 == hello4); //true
System.out.println(hello3 == hello5); //true
String hello6 = new String("hello") + new String("6");
hello6.intern(); //const pool point to hello6
String hello7 = "hello6";
System.out.println(hello7 == hello6); //true

```



