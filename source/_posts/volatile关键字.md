title: volatile关键字
date: 2016-04-14 18:06:55
tags: java
categories: java
comments: true
---

## volatile

volatile

易变的，不稳定的

volatile关键字类似synchronized关键字。不同之处是volatile用来修饰的是单个进行原子操作的变量，而synchronized修饰的是一个代码块。

在java中，多线程访问主线程中的变量时，默认并不会直接操作主线程中的变量，而是从主线程中创建一个变量的拷贝，到对应的子线程中，在子线程操作的过程中去和主线程的变量进行同步，这也就意味着，子线程中操作的变量并不是最新的。

如果想要直接在子线程中访问到主线程的变量，而不是通过拷贝内存的方式，那么就需要在对应的变量之前加上**volatile**关键字，来标识这个变量是与主线程同步的，但这仅仅限于原子操作，比如：

```java
i = i ++;  //不是原子操作，因为i自身参与了运算
i = i +1;  //不是原子操作，因为i自身参与了运算
```

这两种操作都不是原子操作，只有没有自身参与的负值操作，才是原子操作，比如：

```java
i = m + n;  //这是原子操作，因为i自身没有参与运算
```

对于非原子操作的形式，可以选择使用**synchronized**关键字，其实synchronized内部也做了内存同步，不同的是，它对所括起来的区域内的变量都做了内存同步，因此synchronized相比volatile开销会大一些。

所以在不确定是否是原子操作的情况下，最好使用synchronized关键字。


参考文章：

[java中关键字volatile的作用](http://sakyone.iteye.com/blog/668091)