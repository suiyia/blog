title: Java字符串拼接效率对比
author: Answer
tags:
  - String
categories:
  - Java基础
date: 2019-03-17 15:30:00
---
![image](https://note.youdao.com/yws/public/resource/27422d1c5ab3fa95a4e668c46ec74791/E06E2A8F7F51469DAFA40E6A9D43EA2B?ynotemdtimestamp=1552808544738)
用 Google photo 搜索关键词「String」，出现了上图，有兴趣的朋友可以试试，感觉发现了新大陆...

Java 中有 3 种字符串的拼接方式，了解这三种拼接方式的实现，将有益于提高自己的代码质量。

本文主要讲解 String 对象的三种拼接方式，以及它们之间的效率对比。

### 三种方式

```
// 方式 1
String s = "Hello";
s += "Hello";

// 方式 2
StringBuilder stringBuilder = new StringBuilder("Hello");
stringBuilder.append("Hello");

// 方式 3
StringBuffer stringBuffer = new StringBuffer("Hello");
stringBuffer.append("Hello");
```

自己写个主函数，将上面 3 个方法循环执行 10000 次；

**执行时间** t1 > t3 > t2，也就是说 StringBuilder.append() 拼接最快，String += 最慢。

### 源码解读

- StringBuilder.append() 拼接

StringBuilder.java
```
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence

@Override
public StringBuilder append(String str) {
    super.append(str);  // 调用父类 append 方法
    return this;
}
```

AbstractStringBuilder.java
```
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);  // 调用 String 的 getChars 方法
    count += len;
    return this;
}
```
String.java
```
public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {
    if (srcBegin < 0) {
        throw new StringIndexOutOfBoundsException(srcBegin);
    }
    if (srcEnd > value.length) {
        throw new StringIndexOutOfBoundsException(srcEnd);
    }
    if (srcBegin > srcEnd) {
        throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
    }
    System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);  // 本地方法
}
```

**从上面 3 段代码可知，StringBuilder.append 方法调用的是父类 AbstractStringBuilder.append，父类 append 方法调用了 String 的本地方法 System.arraycopy 实现**


- StringBuffer.append() 拼接

StringBuilder.java

```
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence

@Override
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```
**StringBuffer.append 方法与 StringBuilder.append 方法类似，不同的是，append 方法加上了 synchronized 锁，说明该方法适用于多线程环境，但是加锁的过程需要耗时，所以执行时间比 StringBuilder.append 慢。**


- String += 拼接

对下面代码进行编译，生成的 class 文件使用命令 javap -c Main.class，可以反编译得到图片中的字节码。

可以看到 += 实际调用的是 StringBuilder.append() 方法，所以速度会比 StringBuilder 慢。
```
public static void main(String[] args) {
    String s = "Hello";
    s = s + " World";
    System.out.println(s);
}
```
![image](https://note.youdao.com/yws/public/resource/27422d1c5ab3fa95a4e668c46ec74791/06E6477B2E6F4209B3DA88D5BF43E5F8?ynotemdtimestamp=1552808544738)

### 总结

所以在需要拼接字符串的场合，尽量适用 StringBuilder.append() 方法，多线程环境下则推荐 StringBuffer.append() 方法，而 String += 拼接方式任何场合都不建议使用。