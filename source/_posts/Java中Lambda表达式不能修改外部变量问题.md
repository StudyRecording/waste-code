---
title: Java中Lambda表达式不能修改外部变量问题
tags:
  - Java
categories:
  - Java
excerpt: 分析Java中Lambda表达式不能修改外部变量的问题原因以及根据原因找到相关的解决方式
thumbnail: https://pan.mwm.moe/f/pvrIk/10.WEBP
cover: https://t.mwm.moe/pc
sticky: 1
date: 2022-08-20 15:41:40
---

## 问题
在代码Lambda表达式中访问了外部的局部变量，在IDEA中会报检查错误：
![image](https://studyrecording.github.io/waste-code/images/Java%E4%B8%ADLambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%B8%8D%E8%83%BD%E4%BF%AE%E6%94%B9%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F%E9%97%AE%E9%A2%98/image.png)

## 已知前提
1. num是方法中的局部变量
2. forEach的Lambda表达式实际上是创建了一个匿名内部类
3. 该匿名内部类实现的是Consumer接口，实际上forEach中调用的是实现了Consumer接口的匿名内部类的accept方法
4. 在匿名内部类中对num进行修改

## 分析原因
1. sum是方法的局部变量，是int基本类型。而在Java的线程模型中，栈帧内的局部变量是线程私有的。如果将sum的内存地址泄露到匿名内部类中，可能会引起并发访问的问题，并且栈帧中私有变量的内存地址泄露是不安全的。
2. 在Java方法调用中，关于基本类型的传递是值传递的因此在匿名内部类中的num与外部方法中的num实际上不是同一个内存地址指向的值，因此匿名内部类中num的值的改变不会导致外部局部变量的改变，为防止开发人员将两处的num认为是同一个内存地址指向的值，因此IDEA会报错。


## 解决方案
如报错提示中一样定义成原子类便可，如下图
![image-1660980442426](https://studyrecording.github.io/waste-code/images/Java%E4%B8%ADLambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%B8%8D%E8%83%BD%E4%BF%AE%E6%94%B9%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F%E9%97%AE%E9%A2%98/image-1660980442426.png)
这里AtomicInteger实际上的作用是引用类型，在方法调用时实际上传的是指向num的内存地址的值，因此在匿名内部类中指向的值与外部变量num执行的值是同一个，当然原子类也同时保证的并发访问的安全性。当然，这里的主要作用是将数据变为引用类型，方便传递后匿名内部类的值和匿名内部类外部的值指向的是同一块内存地址的值，如下图所示改变也可以
![image-1660980918060](https://studyrecording.github.io/waste-code/images/Java%E4%B8%ADLambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%B8%8D%E8%83%BD%E4%BF%AE%E6%94%B9%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F%E9%97%AE%E9%A2%98/image-1660980918060.png)

## 注意
既然变为引用变量就可以，使用Integer类型可不可以呢？
![image-1660981012175](https://studyrecording.github.io/waste-code/images/Java%E4%B8%ADLambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E4%B8%8D%E8%83%BD%E4%BF%AE%E6%94%B9%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F%E9%97%AE%E9%A2%98/image-1660981012175.png)
如上图所示，IDEA同样会报错的，因为Integer类型在传到匿名内部类中会进行拆箱操作，因此相当于基本类型的值传递。