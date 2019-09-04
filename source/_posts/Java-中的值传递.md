title: Java中的值传递
author: Answer
tags: []
categories:
  - Java基础
toc: true
date: 2019-02-18 23:12:00
---
### 几个重要概念

- **实参、形参**

  - 形式参数：定义函数名和函数体时候使用的参数，目的用来**接收**调用该函数时**传入的参数**
  
  - 实际参数：在调用有参函数时，主调函数与被调函数之间有数据传递关系。实际参数是**调用有参方法的时候真正传递的内容**。
 
 
 ```
 public void tes(String name){ // 形式参数 name
    System.out.println(name);
}

public static void main(String[] args) {
    Test test = new Test();
    test.tes("caijicoder"); // 实际参数 caijicoder
}

 ```

---

- **值类型、引用类型：**

值类型就是**基本数据类型**，8 种基本类型**除外**的数据类型都是引用类型。

两种类型分别表示两种内存分配方式。一个**值类型**数据直接在**栈**上分配，存储所包含的值，其值就代表数据本身。一个**引用类型**指向的数据在**堆**上分配，引用类型的值是这个堆上数据的地址。

```
int num = 10;
String str = "hello";
```
num 是基本类型（值类型），值就直接保存在变量中。
str 是引用类型，变量中保存的只是实际对象的地址（0x10），而不是 Hello 这个字符串。

---


- **值传递、引用传递：**

  - 值传递（pass by value）：指在调用函数时，将实参复制一份传递到函数中，形参接收到的内容其实是实参的一个拷贝，函数对形参的修改并不会影响到实参
  
  - 引用传递（pass by reference）：指在调用函数时，将实参的地址直接传递到函数中，在函数中对参数的修改将会影响到实参
 
值传递和引用传递属于函数调用时参数的**求值策略**（Evaluation Strategy），这是对调用函数时，求值和传值的方式的描述，并不指传递的内容的类型。

也就是说，**传递内容的类型是值类型还是引用类型（地址），与值传递、引用传递无关，并不能说传入的参数类型是值类型就是值传递**。

---

**接下来重点！！！**

**对于值传递，无论是值类型还是引用类型，都会在调用栈上创建一个副本：**

  - **对于值类型而言，这个副本就是整个原始值的复制，对这个副本的操作，不影响原始值的内容。**

  - **对于引用类型而言，其副本也只是这个引用的复制，指向的仍然是同一个对象。所以对副本的操作，会影响原始值。**

![upload successful](https://note.youdao.com/yws/public/resource/5b893506b6477fb678c8a98ca5888a85/4C4F78841E6241959282C73A0BD3CD08?ynotemdtimestamp=1550503530839)

[为什么 Java 只有值传递，但 C# 既有值传递，又有引用传递，这种语言设计有哪些好处？ - Hugo Gu的回答 - 知乎](
https://www.zhihu.com/question/20628016/answer/28970414)


### 一个实例

定义 Person 类
```
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

```
public class Main {

    private int X = 123;

    public void updateVlue(int value){  // 传入基本数据类型
        value = value + 1;
        System.out.println("变量value: "+value);
    }
    
    public void updateObject(Person person){ // 传入引用类型
        Person E = person;
        E.setAge(21);
    }

    public void swapObject(Person A,Person B){  // 传入引用类型
        Person C = A;
        A = B;
        B = C;
    }
    
    public static void main(String[] args) {
        // 例子 1
        Main main = new Main();
        int X = 1;
        main.updateVlue(X);
        System.out.println("X 的值："+X); // X = 1
        
        // 例子 2
        Person A = new Person("张三",20);
        main.updateObject(A);
        System.out.println("A: "+A.toString());  // A: Person{name='张三', age=21}
        
        // 例子 3
        Person C = new Person("C",10);
        Person D = new Person("D",15);
        main.swapObject(C,D);
        System.out.println("C: "+ C.toString());  // C: Person{name='C', age=10}
        System.out.println("D: "+ D.toString());  // D: Person{name='D', age=15}
    }
}
```

例子1：函数传入基本数据类型（值类型参数），由于 value 是 X 的一个副本，对 value 进行操作，并没有改变原来实参的值。

例子2：函数传入引用类型参数，改变了原来的值。由于**值传递**的缘故，传入引用类型的参数时，其值是这个地址的拷贝，指向的仍然是同一个对象，所以发生了改变。这是值传递带来的效果，与传入的对象是值类型或者引用类型没有关系！

例子3：函数传入引用类型，如果 Java 是引用传递， 那么 swapObject(Person A,Person B) 中的形参 A，B 接收的就是 C 和 D 的地址，对 A，B 进行交换应该能成功的，事实上 C 和 D 并没有交换，这从**反面证明了 Java 不是引用传递。**

### 参考

[Java 到底是值传递还是引用传递？ - Hollis的回答 - 知乎](https://www.zhihu.com/question/31203609/answer/576030121)

[这一次，彻底解决Java的值传递和引用传递](https://juejin.im/post/5bce68226fb9a05ce46a0476)