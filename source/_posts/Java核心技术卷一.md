---
title: Java核心技术卷一
date: 2020-08-19 20:22:53
summary: Java核心技术卷一笔记，从零开始摸爬滚打~
categories: Java
tags: 
	- Java
	- Java核心技术卷
---

# Day01

- [x] P164

### 安装库源文件和文档

库源文件在jdk/src.zip中

使用```jar xvf  jdk/src.zip```解压

更多源代码访问[网站](http://jdk8.java.net)

文档独立于JDK，需要单独下载

```java
a += b += c //等价于a += (b += c)  右结合运算符
```

不要使用`==`来检测两个字符串是否相等，该运算符只能确定两个字符串是否放置在同一位置上

```
String s1 = "Hello";
if(s1=="Hello")...//可能为true
if(s1.substring(0,3)=="Hel")...//可能为false
```

#### 多态与动态绑定

```java
//Manager是Employee的子类
Manager boss = new Manager();
Employee employee = new Employee();
Employee[] staff = new Employee[2];
staff[0] = boss;
staff[1] = employee;
for(Employee e:staff){
    System.out.plintln(e.getSalaty())
}
//当e引用Employee对象是，调用Employee类中的getSalary()
//当e引用Manager对象是，调用Manager类中的getSalary()
//一个对象变量(如：e)可以指示多种实际类型的现象叫多态(polymorphism)
//在运行时能够自动选择调用哪个方法的现象叫做动态绑定(dynamic binding)

//对象的类型转换
Manager boss = (Manager)staff[0];  //
Manager boss = (Manager)staff[1];  //由上向下的转换 error
//可以在转换之前进行判断
if(staff[1] instanceof Manager){
    boss = (Manager)staff[1];
    ...
}
```

##### 一般情况下，应该尽量少用类型转换和instanceof

### hashCode

```java
//hashCode方法定义在Object类中，因此每个对象都有一个默认的散列码，其值为对象的存储地址
String s = "OK";
StringBuilder sb = new StringBuilder(s);
System.out.println(s.hasCode()+" "+sb.hashCode());
String t = new String("OK");
StringBuilder tb = new StringBuilder(t);
System.out.println(t.hasCode()+" "+tb.hashCode());
//结果:   对象     散列码
//		  s       2556
//        sb      20526976
//        t       2556
//        tb      20527144
//s和t的散列码相同，因为字符串的散列码是由内容导出的
//sb和tb散列码不同，因为StringBuilder没有定义hashCode方法，其散列码由Object类默认的hashCode方法导出的对象存储地址
```

##### 建议自定义类中添加toString方法

### 自动装箱与自动拆箱

```java
//自动装箱
ArrayList<Integer> list = new ArrayList<>();
list.add(3);//将会自动变成
list.add(Integer.valueOf(3));
//自动拆箱
int n = list.get(i);//将会自动变成
int n = list.get(i).intValue();
```

- ##### 自动装箱规范要求boolean,byte,char<=127,介于 -128~127之间的short和int被包装到固定的对象中。

- ##### 如果在一个表达式中混合使用Integer和Double类型，Integer就会自动拆箱，提升为double，再装箱为Double

```java
try{
    ...
}catch(Exception e){
    e.printStackTrace();
}
```

- [x] P239

# Day02

### 浅拷贝与深拷贝

```java
//根据对象属性的拷贝程度（基本数据类型和引用类型）分为浅拷贝和深拷贝
//浅拷贝：创建一个新对象，拥有原始对象属性值的精确拷贝。若属性是基本数据类型，拷贝的就是属性的值；
//如果属性是引用类型，拷贝的就是内存地址
//因此当一个对象改变了这个地址就会影响到另外的对象
//即默认拷贝构造函数只是对对象进行浅拷贝复制（逐个成员一次拷贝），即只复制对象空间而不复制资源
//实现浅拷贝的类需要实现Cloneable接口，覆盖clone方法

//深拷贝：在拷贝引用类型属性时，为其另辟独立的内存空间，实现真正内容上的拷贝
//逐层实现Cloneable接口
```

- [x] 258

### 内部类

### 代理

### 异常

```java
//所有的异常都是由Throwable继承而来，在下一层分解为Error和Exception两个分支
//Error类描述Java运行时系统的内部错误和资源耗尽错误
//Exception又分解为两个分支
//一个分支派生于RuntimeException，另一个分支包含其他异常
//由程序错误导致的异常属于RuntimeException
//程序本身没有问题，但由于像I/O错误这类问题导致的异常属于其他异常
```

##### 派生于RuntimeException的异常包含以下情况：

- 错误的类型转换
- 数组访问越界
- 访问null指针

##### 不是派生于RuntimeException的异常：

- 试图在文件尾部后面读取数据
- 试图打开一个不存在的文件
- 试图根据给定的字符串查找Class对象，但这个字符串表示的类并不存在

##### 规则：如果出现RuntimeException异常，那么就一定是你的问题

- 数组下标越界异常：ArrayIndexOutOfBoundsException
- null指针异常：NullPointerException

##### Java语言规范将派生于Error类或者RuntimeException类的所有异常称为*非受查异常*(unchecked),其他异常称为受查异常(checked)

运行时异常,不可查，不需要显示捕捉

可查异常，必须显示捕捉或者抛出

#### 断言和日志

### 泛型

# Java核心技术与面试指南

- [ ] P96
- [ ] P87
- [ ] P107

定义基本数据类型的变量，对象的引用，还有函数调用的现场保存都使用`JVM`栈空间

通过`new`和构造器创建的对象放在堆空间，堆事垃圾收集器管理的主要区域

栈用光StackOverflowError

堆和常量池不足OutOfMemoryError





- 两个对象相同(equals返回true)，则对象的hashCode值一定相同
- 当两个对象的hashCode相同，它们并不一定相同
- equals：
  - 自反性(x.equals(x)==true)
  - 对称性(x.equals(y)==true)(y.equals(x)==true)
  - 传递性(x.equals(y)==true)(y.equals(z)==true)(x.equals(z)==true)
  - 一致性(x,y引用的对象信息没有改变时，x.equals(y)返回值不变)

sleep()进入阻塞态 对应TIMED_WAITING

yield()进入就绪态 

join() 对应WAITING

##### 两阶段终止模式

InterruptException会重置isInterrupted为false

isInterrupted不会清除打断标记

Interrupted会清除打断标记



##### Transient:阻止用此关键字修饰的变量序列化，当对象反序列化时该变量不会被持久化和恢复，只能修饰变量

##### IO流

- 按流向分：输入流和输出流
- 按操作单元分：字节流和字符流
- 按流的角色分：节点流和处理流

- InputStream/Reader:所有输入流的基类，前者字节输入流，后者字符输入流
- OutputStream/Writer：所有输出流的基类，前者字节输出流，后者字符输出流

- Reader-字符读取
  - 节点流
    - FileReader(文件操作)
    - PipedReader(管道操作)
    - CharArrayReader(数组操作)
  - 处理流
    - BufferedReader(缓冲操作)
    - InputStreamReader(转化控制)
- Writer-字符写出
  - 节点流
    - FileWriter
    - PipedWriter
    - CharArrayWriter
  - 处理流
    - BufferedWriter
    - OutputStreamWriter
    - PrintWriter(打印控制)
- InputStream-字节读取
  - 节点流
    - FileInputStream
    - PipedInputStream
    - ByteArrayInputStream
  - 处理流
    - BufferedInputStream
    - DataInputStream(基本数据类型操作)
    - ObjectInputStream(对象序列化操作)
    - SequenceInputStream
- OutputStream-字节读出
  - 节点流
    - FileOutputStream
    - PipedOutputStream
    - ByteArrayOutputStream
  - 处理流
    - BufferedOutputStream
    - DataOutputStream
    - ObjectOutputStream
    - PrintStream

- BIO：同步阻塞I/O
- NIO：同步非阻塞I/O
- AIO：异步非阻塞I/O（NIO2）

synchronized

语法

```java
synchronized(对象){
    临界区
}
```

```java
    public synchronized void test(){
        count++;
    }
	//等价于
    public void increment(){
        synchronized (this){
            count++;
        }
    }
    //锁方法相当于锁this对象
//-----------------------------------------------
    public synchronized static void test(){
        
    }
    //等价于
    public static void test(){
        synchronized (Test.class){
            
        }
    }
    //锁静态方法相当于锁类对象
    
```

private与final提供安全的意义所在，开闭原则中的闭

常见的线程安全类

- String
- Integer等包装类
- StringBuffer
- Random
- Vector
- HashTable
- java.util.concurrent包下的类

这里线程安全是指，多个线程调用同一个实例的某个方法时，是线程安全的，即

- 它们的每个方法是原子的
- 但它们多个方法的组合不是原子的

**==与equals()**:

**compareTo()**:



## 关于JVM、JDK、JRE

Java虚拟机(JVM)是运行Java字节码的虚拟机，JVM针对不同系统有不同的实现。

Java程序从源代码到运行一般有三步：

`*.java文件`-->`*.class文件`-->机器可执行的二进制代码

JDK(Java Develpoment Kit)，是功能齐全的Java SDK，拥有JRE的一切并且还有编译器(javac)和工具(如javadoc和jdb)，能够创建和编译程序。

JRE是Java运行时环境，是运行已编译Java程序所需所有内容的集合，包括JVM，Java类库，Java命令和其他的基础构件，不能创建新程序。

## 关于重写(override)与重载(overload)

**重载**

完整描述一个方法，需要指出方法名和参数类型，这就是方法的签名。当多个方法有相同名字、不同参数就会产生重载。返回类型不是方法签名的一部分。

**重写**

子类对父类的允许访问的方法的实现进行重新编写，方法名、参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。

## 关于封装、继承、多态

**封装**

封装是把一个对象的属性私有化，并提供一些可以被外部访问的属性的方法。

**继承**

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或功能，也可以使用父类的功能。

> 需要注意的三点：
>
> 1. 子类拥有父类对象所有的属性和方法(包括私有属性和私有方法)，但父类中的私有属性和方法子类无法访问，只是拥有。
> 2. 子类可以对父类进行扩展。
> 3. 子类可以用自己的方式实现父类的方法。

**多态**

多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量到底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在程序运行期间才能决定。

在Java中有两种方式实现多态：继承（多个子类对同一方法的重写）和接口（实现接口并覆盖接口中同一方法）。

**继承与实现**

**继承与组合**

**Java只有值传递**



**成员变量和方法作用域**

- default：表明该成员变量或者方法只有自己和其位于同一个包的内可见,其他包内的类不能访问,即便是它的子类。
- public：表明该成员变量或者方法是对所有类或者对象都是可见的,所有类或者对象都可以直接访问。
- protected：表明成员变量或者方法对类自身,与同在一个包中的其他类可见,其他包下的类不可访问,除非是他的子类。
- private：表明该成员变量或者方法是私有的,只有当前类对其具有访问权限,除此之外其他类或者对象都没有访问权限.子类也没有访问权限。

## 关于接口和抽象类

- 在Java中，接口不是类，而是对类的一组需求描述，并且不能通过new实例化一个接口，但是可以声明接口的变量，接口变量必须引用实现了接口的类对象。

- 接口中方法默认标记为`public`，域标记为`public static final`

- 每个类只能继承一个抽象类，但是可以实现多个接口

## 关于==与equals

==：作用是判断两个对象的地址是否相等。若对象是基本数据类型时比较对象的值，若对象是引用数据类型则比较内存地址。

equals：作用是判断两个对象是否相等，若类没有覆盖equals方法，此时equals相当于==；若类覆盖了equals方法，则通过equals来比较对象的内容是否相等。

## 关于hashCode与equals

只有在HashMap、HashSet、HashTable等本质是散列表的数据结构中使用对象时，hashCode和equals才有关系，此时需要重写hashCode和equals。

> 对象相同，hashCode一定相同
>
> hashCode相同，对象不一定相同