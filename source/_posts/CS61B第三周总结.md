---
title: CS61B第三周总结
date: 2020-01-06 10:00:36
tags: 
    - 总结
    - CS61B
catalog: true
categories:
    - CS61B
---
# 第三周总结笔记

CS61B的第三周主要是关于Java的单元测试，双向链表这两个部分

## 一，Java的单元测试与Junit

Junt的一些基本用法。没啥好说的。详细的内容之后才有。



## 二，Java的接口，实现接口以及动态静态类型

### （1）接口实现interface implement和默认方法default关键字

简单来说就是接口原先不能有方法体，但是从Java8开始接口可以先实现一个默认的方法体。采用`default`关键字。

![](https://raw.githubusercontent.com/StoicLD/Photos/master/Other/20200105201126.png)



### （2）Static Type和Dynamic Type以及Dynamic Method Selection

如下所示的一个例子。可以看到最后

```java
a.flatter(d);
```

输出的是"u r cool animal"。这就涉及到所谓的**Static Type**和**Dynamic Type**以及**Dynamic Method Selection**。

![](https://raw.githubusercontent.com/StoicLD/Photos/master/Other/20200105201402.png)



**Static Type**和**Dynamic Type**定义如下

![](https://raw.githubusercontent.com/StoicLD/Photos/master/Other/20200105201759.png)

声明的类型是**Static Type**，而实例化的时候的类型是**Dynamic Type**

然后我们明确一下overriding重写和overload重载

overriding重写指的是子类或者接口的实现类对于签名一致（既参数与函数名一致）的父类函数或者接口函数的实现。

overload指的是函数名一致但是参数不同的另一种函数实现方式。

---

最后我们再来看下**Method Selection Algorithm**

![](https://raw.githubusercontent.com/StoicLD/Photos/master/Other/20200105202006.png)

以此为例：
解析a.flatter(d)
a的**static type**是Animal，**dynamic type**是Dog
d的**static type**是Dog，**dynamic type**是Dog

a.flatter(d); 在编译期，记录下a的**static type**的方法中可以处理d的**static type**类型参数的方法，此处就是Animal的flatter方法。（d继承自Animal，尽管d的static type和dynamic type都是Dog，但还是可以作为Animal类型的参数被接受）

因为Dog的flatter方法是**重载**了其父类Animal的flatter方法
而不是**重写overriding**，所以运行时没有改变Animal的flatter
方法改为Dog版本的（因为不是重写！！！）
所以最后还是采用了Animal版本的flatter方法

![](https://raw.githubusercontent.com/StoicLD/Photos/master/Other/20200105201402.png)



## 三，双端队列

参见链表目录下的[Deque文章](/Users/Ld19980608/OneDrive/Typora/CS61B/链表/Deque.md)



## 四，参考资料

【1】[Josh Hug's Book](https://joshhug.gitbooks.io/hug61b/content/chap2/chap25.html)

【2】[Alist Slides](https://docs.google.com/presentation/d/1LGQeMHb8-HFKdvJi5nGKRIPZt4on18fZe-cIyTJv8_4/edit)

【3】[Testing Slides](https://docs.google.com/presentation/d/1g2RwCFKvbv2x0lkFW9hwK1IcmNTHEqZloOLRgF4clhA/edit#slide=id.g3996ea012_0156)

【4】[inheritance](https://docs.google.com/presentation/d/1b-Ue_mWWMI2CeHfaU_GO6PWhIpAr_SyZge9JQJQFpXc/edit#slide=id.g1c60d50676_54_6)

【5】[Project1A](https://sp19.datastructur.es/materials/proj/proj1a/proj1a#1-linked-list-deque)