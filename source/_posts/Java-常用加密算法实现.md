title: Java 常用加密算法实现
author: Answer
tags:
  - 加密算法
categories:
  - Java 核心
date: 2019-01-27 17:23:00
---
[toc]

> 常用加密算法 Java 实现

### 算法种类
- 单向加密
- 对称加密
- 非对称加密

### 1. 单向加密

**Base64**，Base64 编码是从二进制到字符的过程，用 64 个字符来表示任意的二进制数据，常用于在 HTTP 加密，图片编码传输等。
- Java 8 内置实现

```java
package com.cn.singleway;

import java.io.UnsupportedEncodingException;
import java.util.Base64;

public class Base64Demo {

    public static void main(String[] args) {
        try {
            // 编码
            String encode = Base64.getEncoder().encodeToString("testBase64".getBytes("UTF-8"));
            System.out.println(encode);
            // 解码
            byte[] decode = Base64.getDecoder().decode("dGVzdEJhc2U2NA==");
            System.out.println(new String(decode, "UTF-8"));
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
}

```
- JDK 1.8 之下，引入第三方 jar 包

```java
package com.cn.singleway;

import org.apache.commons.codec.binary.Base64;
import java.io.UnsupportedEncodingException;

public class Base64Demo2 {

    public static void main(String[] args) {
        try {
            String encodeBase64String = Base64.encodeBase64String("test".getBytes("UTF-8"));
            System.out.println(encodeBase64String);

            byte[] decodeString = Base64.decodeBase64(encodeBase64String);
            System.out.println(new String(decodeString,"UTF-8"));

        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
}

        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.11</version>
        </dependency>

```

**MD5**，Message Digest algorithm 5，信息摘要算法，一般用于确保信息的传输完整一致性，校验传输的数据是否被修改，一旦原始信息被修改，生成的 MD5 值将会变得很不同

```java
package com.cn.singleway;

import java.io.UnsupportedEncodingException;
import java.math.BigInteger;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class MD5Demo {

    public static void test1(String s){
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            byte[] after = messageDigest.digest(s.getBytes("UTF-8"));
            StringBuilder stringBuilder = new StringBuilder();
            for (int i = 0; i < after.length; i++) {
                if ((0xff & after[i]) < 0x10) {
                    stringBuilder.append("0" + Integer.toHexString((0xFF & after[i])));
                } else {
                    stringBuilder.append(Integer.toHexString(0xFF & after[i]));
                }
            }
            System.out.println(stringBuilder.toString());
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }

    public static void test2(String s){
        try {
            // 生成一个MD5加密计算摘要
            MessageDigest md = MessageDigest.getInstance("MD5");
            // 计算md5函数
            md.update(s.getBytes());
            // digest()最后确定返回md5 hash值，返回值为8为字符串。因为md5 hash值是16位的hex值，实际上就是8位的字符
            // BigInteger函数则将8位的字符串转换成16位hex值，用字符串来表示；得到字符串形式的hash值
            System.out.println(new BigInteger(1, md.digest()).toString(16));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        String s = "this is MD5 test demo";
        test1(s);
        test2(s);
    }

}

```


- SHA 家族，是一个密码散列函数家族，是 FIPS 所认证的安全散列算法，和 MD5 类似，都是对文本进行散列，产生一定长度的散列值
SHA1 与 SHA2

```java

```


- HMAC
Hash Message Authentication Code，散列消息鉴别码，HMAC运算利用哈希算法，以一个密钥和一个消息为输入，生成一个消息摘要作为输出。

```Java

```
---

### 2. 对称加密
**对称加密的意思就是信息收发都有相同的一把钥匙，消息的加密解密都用这进行**
- DES，Data Encryption Standard，数据加密标准，速度较快，适用于加密大量数据的场合。DES算法提供CBC, OFB, CFB, ECB四种模式，MAC是基于ECB实现的。

- AES
- Advanced Encryption Standard，高级加密标准，是下一代的加密算法标准，速度快，安全级别高；

---

### 3. 非对称加密
**非对称加密算法是一种密钥的保密方法。 非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。 公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。**

- RSA，名称来源于发明这个算法的三个人的姓氏组成，算法大致内容就是对极大整数进行因式分解。这种算法非常可靠，密钥越长，它就越难破解。根据已经披露的文献，目前被破解的最长 RSA密钥是768个二进制位。也就是说，长度超过768位的密钥，还无法破解（至少没人公开宣布）。因此可以认为，1024位的RSA密钥基本安全，2048位的密钥极其安全。

[http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)

- DSA，Digital Signature Algorithm，数字签名算法，是一种标准的 DSS（数字签名标准）；

- ECC，Elliptic Curves Cryptography，椭圆曲线密码编码学。一种建立公开密钥加密的算法，基于椭圆曲线数学。ECC的主要优势是在某些情况下它比其他的方法使用更小的密钥——比如RSA加密算法——提供相当的或更高等级的安全。ECC的另一个优势是可以定义群之间的双线性映射，基于Weil对或是Tate对；双线性映射已经在密码学中发现了大量的应用，例如基于身份的加密。不过一个缺点是加密和解密操作的实现比其他机制花费的时间长。
 
## Base64


## MD5 和 SHA 家族
```java
public static void main(String[] args) {
    
    String content = "you are my son"; // 原文
    try {
        byte[] a;
        MessageDigest messageDigest = MessageDigest.getInstance("SHA-1");
        a = messageDigest.digest(content.getBytes());
        System.out.println(byte2hex(a)); // 333a9634d8809b5a9e8d280d82553b8fd8d4a911

        messageDigest = MessageDigest.getInstance("SHA-256");
        a = messageDigest.digest(content.getBytes());
        System.out.println(byte2hex(a)); // cdb2c97079d9a1943eea98de4201f5c4f49ecda5af2b364e1c7a5d1ae89688eb

        messageDigest = MessageDigest.getInstance("MD5");
        a = messageDigest.digest(content.getBytes());
        System.out.println(byte2hex(a)); // 6fe6b9a8f8bd29f4f4f1368a0619a7ae

        // 第三方 MD5 算法。需要添加 jar 包 org.apache.commons.codec.digest.DigestUtils
        String encodeStr=DigestUtils.md5Hex(content);
        System.out.println(encodeStr); // 6fe6b9a8f8bd29f4f4f1368a0619a7ae

    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
}

public static String byte2hex(byte[] b) //二进制转字符串
{
    String hs = "";
    String stmp = "";
    for (int n = 0; n < b.length; n++) {
        stmp = (java.lang.Integer.toHexString(b[n] & 0XFF));
        if (stmp.length() == 1) {
            hs = hs + "0" + stmp;
        } else {
            hs = hs + stmp;
        }
    }
    return hs;
}
```
--- 

# 总结

- 现在的加密算法大部分情况下是为了验证数据的一致性，例如传递一些参数组的时候，简单的会使用 BASE64 或 MD5 进行加密生成一个签名。复杂点就是 BASE64 编码之后再用 对称密钥再加密一次，达到比较不容易被人篡改的目的

- 对于一些支付场景，一般使用 非对称加密算法 实现，这样的场景需要的安全性更高。

- 其它博客参考
  - [java加解密之RSA使用](https://blog.csdn.net/qq_18870023/article/details/52596808)
