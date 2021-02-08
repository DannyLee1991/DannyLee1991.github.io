title: Android图片内存优化
date: 2015-11-19 13:50:55
tags: Android
categories: Android
comments: true
---

首先，关于android的内存管理机制，建议阅读这篇文章：

[android垃圾回收算法](http://www.cnblogs.com/killmyday/archive/2013/06/12/3132518.html)

如果在布局文件中，设置了背景图，要在界面onDestroy()的时候，将图片回收。

```
//recycle backgroundBitmap 
View v = findViewById(id);
BitmapDrawable bd = (BitmapDrawable) v.getBackground();
if (bd != null){
    bd.getBitmap().recycle();
}
// 将资源id清空
v.setBackgroundResource(0);
```

对布局文件的背景图建议在界面销毁的时候调用上面的代码来将图片的引用置空。

虽然这样不能及时释放内存，但java虚拟机在执行gc的时候，会优先释放这些多余的内存空间。

以下是对上述代码添加前后的效果对比：

添加优化代码前，完全回收一张图片的内存，需要连续两次gc：
![未添加优化代码](/img/android_neicunyouhua2.gif)

添加优化之后，完全回收一张图片的内存，只需要调用一次gc：
![添加优化代码](/img/android_neicunyouhua1.gif)


