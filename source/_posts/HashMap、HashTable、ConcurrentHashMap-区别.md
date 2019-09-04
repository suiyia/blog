title: HashMap、HashTable、ConcurrentHashMap 区别
author: Answer
toc: true
tags:
  - 集合
categories:
  - Java基础
date: 2019-04-07 14:24:00
---
> 简单总结 HashMap、Hashtable、ConcurrentHashMap 之间的区别，基于 JDK 1.8.0_191

先说结论，暂时有以下几个需要注意的不同点：

1. 继承、实现接口不同
2. 初始大小、扩容倍数不同
3. 线程安全
4. NULL KEY，NULL VALUE 支持不同
5. 计算 Hash 值的方式不同


### 1. 继承、实现接口不同
```
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {}
    
public class Hashtable<K,V> extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {    

public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

```

- HashMap、ConcurrentHashMap 都继承于 AbstractMap 抽象类，但 Hashtable 继承于 Dictionary 抽象类。
- AbstractMap 实现了 Map 接口，而 Dictionary 没有。这使得 AbstractMap 具有更多的功能，而 Dictionary 逐渐被弃用。


### 2. 初始大小、扩容倍数不同
HashMap
```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

扩容部分代码：
newThr = oldThr << 1; // double threshold
```
Hashtable
```
public Hashtable() {
    this(11, 0.75f);
}

扩容部分代码：
int newCapacity = (oldCapacity << 1) + 1;
```

ConcurrentHashMap
```
/**
 * The default initial table capacity.  Must be a power of 2
 * (i.e., at least 1) and at most MAXIMUM_CAPACITY.
 */
private static final int DEFAULT_CAPACITY = 16;
```

- 初始大小：HashMap、ConcurrentHashMap 都是 16，Hashtable 是 11
- 扩容倍数：HashMap、ConcurrentHashMap 2 倍、Hashtable (2n + 1) 倍。

### 3. 线程安全

```
Hashtable:
public synchronized V put(K key, V value) {}

HashMap:
public V put(K key, V value) {}
```

- Hashtable 添加元素方法加上了 synchronized 关键字，HashMap 没有，说明 HashMap 不适用于多线程环境。
- ConcurrentHashMap 添加元素时，只对需要变更的地方加锁。


### 4. NULL KEY，NULL VALUE 支持不同

通过观察它们的 put 方法，得到以下结论： 

- Hashtable、ConcurrentHashMap 的 key、value 都不能为 null。
- HashMap value 可以为 null，而 key 为 null 时，该元素会放在 HashMap 第一位。

### 5. key 的索引计算方法不同

计算元素存放位置，会经过两步转化。Object -> int -> index，先将 key 使用 hash 方法转换为一个整数数字，然后对整型数字进行转化，得到这个对象在 map 中的索引。

Hashtable
```
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```
HashMap
```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

- Hashtable 直接使用对象的 hashCode，然后再使用除留余数法来获得最终的位置。
- HashMap 在对象的 hashCode 之上，将 hashCode 的高16位和低16位进行异或，得到最终的位置。

### 总结

后面学习将对 HashMap，ConcurrentHashMap 源码进行分析总结。

[ConcurrentHashMap的扩容机制（jdk1.8）](https://blog.csdn.net/varyall/article/details/81277296)

[HashMap 和Hashtable的区别](http://www.10tiao.com/html/710/201903/2650123468/2.html)

