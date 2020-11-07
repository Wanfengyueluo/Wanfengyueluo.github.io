---
title: Java异常处理
date: 2020-08-19 20:25:34
description: Java基础笔记，从零开始摸爬滚打~
categories: Java
tags: 
	- Java
	- Java异常
---

## Java异常处理手段

### try catch

#### 一般方法

```java
package exception;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;

public class TestException {

	public static void main(String[] args) {
		
		File f= new File("d:/test.exe");
		
		try{
			System.out.println("试图打开 d:/test.exe");
			new FileInputStream(f);
			System.out.println("成功打开");
		}catch(FileNotFoundException e){
			System.out.println("d:/test.exe不存在");
			e.printStackTrace();
		}
	}
}
```

#### 使用异常的父类catch

```java
package exception;
 
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
 
public class TestException {
 
    public static void main(String[] args) {
         
        File f= new File("d:/test.exe");
         
        try{
            System.out.println("试图打开 d:/test.exe");
            new FileInputStream(f);
            System.out.println("成功打开");
        }catch(Exception e){
            System.out.println("d:/test.exe不存在");
            e.printStackTrace();
        }   
    }
}
```

#### 多异常捕捉

```java
package exception;
 
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
 
public class TestException {
 
    public static void main(String[] args) {
 
        File f = new File("d:/test.exe");
 
        try {
            System.out.println("试图打开 d:/test.exe");
            new FileInputStream(f);
            System.out.println("成功打开");
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            Date d = sdf.parse("2016-06-03");
        } catch (FileNotFoundException e) {
            System.out.println("d:/test.exe不存在");
            e.printStackTrace();
        } catch (ParseException e) {
            System.out.println("日期格式解析错误");
            e.printStackTrace();
        }
    }
}
```

#### 多异常捕捉2

```java
package exception;
 
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
 
public class TestException {
 
    public static void main(String[] args) {
 
        File f = new File("d:/test.exe");
 
        try {
            System.out.println("试图打开 d:/test.exe");
            new FileInputStream(f);
            System.out.println("成功打开");
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            Date d = sdf.parse("2016-06-03");
        } catch (FileNotFoundException | ParseException e) {
            if (e instanceof FileNotFoundException)
                System.out.println("d:/test.exe不存在");
            if (e instanceof ParseException)
                System.out.println("日期格式解析错误");
 
            e.printStackTrace();
        }
    }
}
```

### finally

无论是否出现异常，finally中的代码一定会被执行

### throw和throws

> throw抛出的异常如果不处理，会传递到父方法

1.  throws 出现在方法声明上，而throw通常都出现在方法体内。
2.  throws 表示出现异常的一种可能性，并不一定会发生这些异常；throw则是抛出了异常，执行throw则一定抛出了某个异常对象。

## Java异常分类

### 可查异常CheckedException

可查异常是必须进行处理的异常，要么进行try-catch，要么抛出。如果不处理，编译器不会通过。

### 运行时异常RuntimeException

常见的运行时异常有：

- ArithmeticException
- ArrayIndexOutOfBoundsException
- NullPointerException

### 错误Error

错误是指系统级别的异常，例如OutOfMemoryError



