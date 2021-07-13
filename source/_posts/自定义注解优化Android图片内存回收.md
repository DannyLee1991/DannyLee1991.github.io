title: 自定义注解优化Android图片内存回收
date: 2015-11-26 13:50:55
tags: Android
categories: 
- 工程开发
- Android
comments: true
---

java提供了垃圾回收机制，我们不用像C++一样手动去处理垃圾内存。虽然省事，但是为了对垃圾回收器友好，我们应该引导gc，告诉它什么东西是垃圾内存。简单来说，就是不再使用的对象要置空，该回收的资源要回收。

android下老生常谈的异常OOM,主要产生的原因就是加载了过多的资源到内存，导致内存溢出。而在android界面中，图片一直是吃内存的大户。一个1024x800的png图片，打开ARGB通道，占用的内存总量如下：

>	1024 x 800 x 8 x 4 / 8 = 3276800byte = 3200kb = 3.125mb

>	(内存=图片长度*图片宽度*单位像素占用的字节数)
	
而系统分配给每个应用的内存总量是一定的，通常情况下不会超过128mb,因机型而异。除非将largeHeap选项开启，会获得大幅度的内存上限提升。但不是系统有多少内存就可以申请多少，而是由dalvik.vm.heapsize限制。Android官方给的建议是，作为程序员的我们应该努力减少内存的使用，想回收和复用的方法，而不是想方设法增大内存。当内存很大的时候，每次gc的时间也会长一些，性能会下降呦。

那么我们就来通过对view图片内存的释放来优化内存。

根据android的View类的源码，可以看出，当setBackgroundResource(0)时，之前的drawable对象会被移除：

```java
	/**
     * Set the background to a given resource. The resource should refer to
     * a Drawable object or 0 to remove the background.
     * @param resid The identifier of the resource.
     *
     * @attr ref android.R.styleable#View_background
     */
    @RemotableViewMethod
    public void setBackgroundResource(int resid) {
        if (resid != 0 && resid == mBackgroundResource) {
            return;
        }

        Drawable d = null;
        if (resid != 0) {
            d = mContext.getDrawable(resid);
        }
        setBackground(d);

        mBackgroundResource = resid;
    }
```

所以，我们在Activity的onDestory()方法中，调用view的setBackgroundResource(0)的方法，内存就可以在第一时间被gc回收了。

具体效果可以参考这片文章：

[Android图片内存优化](http://dannylee1991.github.io/2015/11/19/android%E5%9B%BE%E7%89%87%E5%86%85%E5%AD%98%E4%BC%98%E5%8C%96/)

为了方便实用，我封装了一个工具:[AnnoRecycleImage](https://github.com/DannyLee1991/AnnoRecycleImage)，具体用法如下：

```java
public class TestActivity extends Activity {

    @RecycleViewBGRes	//对需要回收资源的控件添加注解
    private View root;

    @RecycleViewBGRes(isCustom = true)	//自定义控件 isCustom = true
    private CustomView cv;

    @RecycleViewBGRes
    private ImageView iv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test);
        root = findViewById(R.id.root);
        cv = (CustomView) findViewById(R.id.cv);
        iv = (ImageView) findViewById(R.id.iv);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //在onDestroy中 调用回收方法 处理标注有注解的对象
        RecycleUtils.doRecycle(TestActivity.this);
    }
    
}
```


