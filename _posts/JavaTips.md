---
title: JavaTips
date: 2017-07-25 11:27
tags:
	- Tips
	- Java
---

> 这篇文章主要是个人在二次学习 Java 过程中, 对 Java 的一些常见特性误解的分析. 
> 
> 主要是以测试代码加测试结果的形式来记录. 这里只做收集整理和简单分析, 详情请见参考链接, 支持原作者.

## 目录

- 基本类型
  + == 和 equals
  + String, StringBuffer 和 StringBuilder
  + Array
  + ArrayList
  + LinkedList
  + Exception
  + Collction
- 基本概念
  + 类变量和局部变量

<!-- more -->

### 基本类型

#### == 和 equals

```java
TestCode:
Integer a = 100;
Integer b = 100;
System.out.println(a == b);

Integer c = 1000;
Integer d = 1000;
System.out.println(c == d);

Output:
true
false
```

> 其中 == 结果不一致是因为 Integer 类对在 -128 到 127 之间的数值进行了缓存. 

```java
TestCode:
Integer e = new Integer(100);
Integer f = new Integer(100);
System.out.println(e == f);

Output:
false
```

> `Integer a = 100;`涉及到自动装箱问题, 反编译之后就是 `Integer a = Integer.valueOf(100);`.
> 而`Integer e = new Integer(100);` 反编译不改变. 所以在 a 和 e 这两个对象引用的堆区域一个是通过`valueOf()`获得, 另一个是通过`new`获得. 毫无疑问 `new` 出来的肯定不会是同一块区域. 而 `valueOf()`
> 
> ```java
> public static Integer valueOf(int i) {
> 	if (i >= IntegerCache.low && i <= IntegerCache.high)
> 		return IntegerCache.cache[i + (-IntegerCache.low)];
> 	return new Integer(i);
> }
> ```
> 这里就解释了前面的`Integer 类对在 -128 到 127 之间的数值进行了缓存`. 详情见源码.

```java
TestCode:
String s1 = "1234";
String s2 = "1234";
System.out.println(s1 == s2);

String s3 = new String("1234");
String s4 = new String("1234");
System.out.println(s3 == s4);

Output:
true
false
```
这个比较简单, 就是在 JVM 中存在常量池. 还有要注意的是两种初始化方式的不同才造成了这个差异. 

下面分析 equals ,详情见参考.这里只提几点易混淆的地方.
> equals 是属于类方法, 而且是 Object 类的. 而 Object 类的实现就是判断引用是否相等.
> 
> 而对于某个类来说判断相等一般是判断里面的某些成员变量是否相等. 所以就要重写父类方法.这里要区分重写和重载.
> 
> ```java
> 重写:
> public boolean equals(Object obj) {}
> 重载:
> public boolean equals(MyObject myObj) {}
> ```
> 
> 所以要想实现自己想要的 equals 方法应该是重写.
> 
> 其实这里面的主要内容就是了解了自动拆装箱和 JVM 内分配就不难了.
> 
> - 第一类：整型 byte Byte | short Short | int Integer | long Long
> - 第二类：浮点型 float Float | double Double
> - 第三类：逻辑型 boolean Boolean
> - 第四类：字符型 char Character
> 
> 在支持 equals 特性的时候, 往往还要支持 hashCode 

参考: 

- [让人疑惑的Java代码](https://zhuanlan.zhihu.com/p/27562748)
- [Java字符串那些事儿](https://zhuanlan.zhihu.com/p/27570687)
- [说说Java里的equals（上）](https://zhuanlan.zhihu.com/p/27573287)
- [Java自动装箱/拆箱](https://zhuanlan.zhihu.com/p/27657548)
- [说说Java里的equals（中）](https://zhuanlan.zhihu.com/p/27741179)

#### String, StringBuffer 和 StringBuilder

- String

> final 类.
> 
> String s="sss";	会在静态常量池中查找
> 
> String s = new String("sss");	直接在堆中开辟内存
> 
> 使用场合: 在字符串不经常变化的场景中可以使用 String 类, 如: 常量的声明, 少量的变量运算等

- StringBuffer

> 线程安全, 可自身修改
> 
> 必须通过构造函数初始化
> 
> 使用场合: 在频繁进行字符串的运算(拼接, 替换, 删除等), 并且运行在多线程的环境中, 则可以考虑使用 StringBuffer, 例如 XML 解析, HTTP 参数解析和封装等

- StringBuilder

> 线程不安全, 操作最快
> 
> 使用场合: 和 StringBuffer 类似的不要求线程安全场景, 效率比 StringBuffer 高

#### Array

> 是一个在 JVM 中的特殊对象, 可以使用反射查看
> 
> 拷贝时注意深拷贝还是浅拷贝
> 
> 

#### ArrayList
> 首先要知道的是这是一个 Array 所以在进行修改操作时十分不方便的. 特别是增加或者删除元素, 扩大存储等方式相比链表结构来说麻烦太多.知道这个为出发点, 对于源码实现的理解就理所当然了.
> 应该了解的几个点
> 
> - 在 ArrayList 中从有元素开始就会分配 10 的对象大小容量. 接下来以 `currentSize * 1.5` 的大小扩容.
> - 对于修改空间是以 System.arraycopy() 形式修改的. 为了提高效率采用的是 native 方法.
> 
> ```java
> public static native void arraycopy(Object src, 
> 		int  srcPos, Object dest, int destPos, int length);
> ```
> 
> 详情源码分析见参考.

参考: 

- [ArrayList初探](https://zhuanlan.zhihu.com/p/27873515)
- [再探ArrayList（ArrayList的扩容）](https://zhuanlan.zhihu.com/p/27878015)
- [三顾ArrayList](https://zhuanlan.zhihu.com/p/27938717)

#### LinkedList

> 实现方式是双向链表

#### Exception

![](http://img.my.csdn.net/uploads/201212/02/1354439580_6933.PNG)

- 尽可能的减小try块
- 保证所有资源都被正确释放, 充分运用finally关键词
- catch语句应当尽量指定具体的异常类型, 而不应该指定涵盖范围太广的 Exception 类. 不要一个Exception试图处理所有可能出现的异常
- 既然捕获了异常, 就要对它进行适当的处理
- 在异常处理模块中提供适量的错误原因信息, 组织错误信息使其易于理解和阅读
- 不要在 finally 块中处理返回值
- 不要在构造函数中抛出异常


参考: 

- [树上月](http://www.cnblogs.com/chenssy/category/525010.html)


#### Collction

![](collection.jpg)

##### List 列表(可重复)

- ArrayList (数组)
- LinkedList (链表)

##### Map 映射(key-value)

- HashMap

> extends AbstractMap<K,V> implements Map<K,V>
> 
> table 数组 + Enter 节点
> 
> 默认初始容量(16) 默认加载因子(0.75)
> 
> 查找采用 static int indexFor(int h, int length) { return h & (length-1); } 提高效率, 而不是取模.

- TreeMap

> extends AbstractMap<K,V> implements NavigableMap<K,V> 
> 
> interface NavigableMap<K,V> extends SortedMap<K,V>
> 
> interface SortedMap<K,V> extends Map<K,V>
> 
> 红黑树 + Enter 节点


##### Set 集合(不能重复)

- HashSet

> extends AbstractSet<E> implements Set<E>
> 
> 内部基于 HashMap 实现 

- TreeSet

> extends AbstractSet<E> implements NavigableSet<E> 
> 
> interface NavigableSet<E> extends SortedSet<E>
> 
> interface SortedSet<E> extends Set<E>
> 
> 基于 TreeMap 实现

### 基本概念
#### 类变量和局部变量

```java
public static void main(String[] args) {
	int a;
	System.out.println(a);
}
```
类变量：准备阶段、初始化阶段都可赋值。
局部变量：准备阶段不能检测，初始化阶段未初始化不能使用。