title: 通过AspectJ代码注入来实现scheme跳转条件的检查判断
tags: 
- Android
categories: 
- 工程开发
- Android
comments: true
date: 2017-3-21 22:48:58
---

scheme跳转是Android通过外部链接打开APP指定页面的一种常见的实现方式，如果只是简单的跳转，那么不需要做什么额外的判断就可以打开指定页面了，但我们的产品中有一个需求就是：在通过scheme跳转打开某一个页面的时候，需要判断一些前置条件，如果前置条件满足的情况下，才能执行跳转，如果前置条件不满足，那么需要缓存本次跳转，直到需要满足的条件被触发时，才去执行跳转。

简单说来就是下面这张图：

![](/img/17_03_21/001.png)

一开始我想到的处理方式是通过在Activity的基类里的`onCreate()`方法之前做判断逻辑，如果符合条件，则正常执行，如果条件不满足，则执行`finish()`。

虽然可以满足需求，但这样的代码侵入性太高，逻辑必须侵入到Activity的基类中，容易与基类中其他逻辑产生耦合。

因此为了解决这个方式，我想到的解决方式就是使用AOP的方式，在`onCreate()`之前注入我们的判断逻辑的代码，最后的使用方式可以简化到仅仅使用一行注解来添加判断条件：

```java
@SchemeCheck(conditions = {Condition.LOGIN})
public class TargetActivityA extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
    ...
}
```

这样一来，我们就不用考虑在Activity的基类中写这些可能产生冗余的逻辑了。

最终实现效果如下：

- 判断**登录**条件

<video width='300px' autoplay="autoplay" loop="loop" id="video" controls="" preload="none" >
  <source id="mp4" src="/img/17_03_21/002.mp4" type="video/mp4">
  <p>Your user agent does not support the HTML5 Video element.</p>
</video>
    
- 判断**下载**条件

<video width='300px' autoplay="autoplay" loop="loop" id="video" controls="" preload="none" >
  <source id="mp4" src="/img/17_03_21/003.mp4" type="video/mp4">
  <p>Your user agent does not support the HTML5 Video element.</p>
</video>

- 判断**登录并且下载**

<video width='300px' align="center" autoplay="autoplay" loop="loop" id="video" controls="" preload="none" >
  <source id="mp4" src="/img/17_03_21/004.mp4" type="video/mp4">
  <p>Your user agent does not support the HTML5 Video element.</p>
</video>

- 判断登录或者下载

<video width='300px' autoplay="autoplay" loop="loop" id="video" controls="" preload="none" >
  <source id="mp4" src="/img/17_03_21/005.mp4" type="video/mp4">
  <p>Your user agent does not support the HTML5 Video element.</p>
</video>

接下来介绍一下我是如何实现的。

## 如何实现

首先关于**依赖注入**以及**AspectJ**的相关使用，我参考了以下文章以及代码，具体使用方式我就不再赘述：

- [【翻译】Android中的AOP编程](http://www.jianshu.com/p/0fa8073fd144)  这一篇对AOP概念进行了介绍，并且通过AspectJ仿照[Hugo](https://github.com/JakeWharton/hugo)实现了一个AOP的Demo。
- JakeWharton大神的[hugo](https://github.com/JakeWharton/hugo) 项目，通过AspectJ实现的日志工具。
- [使用AspectJ在Android中实现Aop](http://blog.csdn.net/crazy__chen/article/details/52014672)这一篇文章是对上面的文章和hugo项目的总结。

---

下面是我的实现：

首先，定义注解：

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface SchemeCheck {
    Condition[] conditions();
}
```

值得提醒的是，这里`@Retention`要定义为`RetentionPolicy.RUNTIME`，因为我们要在运行时检查注解的参数，来判断scheme的触发条件。如果你写成了`RetentionPolicy.CLASS`或者`RetentionPolicy.SOURCE`就检查不到了。

`Condition`是我们定义的需要判断的条件枚举：

```
public enum Condition {
    NULL(null),
    LOGIN(new LoginCondition()),
    DOWNLOAD_BOOK(new BookDownloadCondition()),
    LOGIN_OR_DOWNLOAD_BOOK(new LoginOrBookDownloadCondition());

    BaseCondition condition;
    Condition(BaseCondition condition) {
        this.condition = condition;
    }

    public BaseCondition getCondition() {
        return condition;
    }
}
```

`Condition`可以通过`getCondition()`来获取到具体的继承自`BaseCondition`的Condition对象：

```
public abstract class BaseCondition {
    public abstract boolean isSatisfied();

    public abstract String unSatisfiedInfo();
}
```

由于是Demo，我们的登录条件暂时写死，到时候换成你具体的业务逻辑即可：

```
public class LoginCondition extends BaseCondition {

    public static boolean isLogin = false;

    @Override
    public boolean isSatisfied() {
        return isLogin;
    }

    @Override
    public String unSatisfiedInfo() {
        return "请先登录";
    }
}
```

有了这些，我们就可以写注入代码了：

```
@Aspect
public class SchemeCheckAspect {
    @Pointcut("within(@demo.com.aj.anno.SchemeCheck *)")
    public void withinAnnotatedClass() {
    }

    @Pointcut("execution(!synthetic * *(..)) && withinAnnotatedClass()")
    public void methodInsideAnnotatedType() {
    }

    @Pointcut("execution(@demo.com.aj.anno.SchemeCheck * *(..)) || methodInsideAnnotatedType()")
    public void method() {
    }

    @Around("method()")
    public Object weaveJoinPoint(ProceedingJoinPoint joinPoint) throws Throwable {
        Signature signature = joinPoint.getSignature();
        String methodName = signature.getName();

        if (TextUtils.equals(methodName, "onCreate")) {
            SchemeCheck anno = (SchemeCheck) signature.getDeclaringType().getAnnotation(SchemeCheck.class);
            if (anno != null) {
                Condition[] conditions = anno.conditions();
                Object point = joinPoint.getThis();
                if (point != null && point instanceof Activity) {
                    Activity activity = (Activity) point;
                    handleScheme(activity, conditions);
                }
            }
        }
        return joinPoint.proceed();
    }

    /**
     * 检查前置条件
     *
     * @return 是否通过检查
     */
    private boolean checkCondition(Condition[] conditions) {
        if (conditions != null) {
            for (Condition condition : conditions) {
                BaseCondition conditionObj = condition.getCondition();
                if (conditionObj != null
                        && !conditionObj.isSatisfied()) {
                    Toast.makeText(App.getContext(), conditionObj.unSatisfiedInfo(), Toast.LENGTH_SHORT).show();
                    SchemeManager.Cache.save(condition);
                    return false;
                }
            }
        }
        return true;
    }

    /**
     * 通过验证
     */
    private void passSatisfy() {
        SchemeManager.Cache.clear();
    }

    /**
     * 处理前置条件的检查结果
     *
     * @param isPass 检查是否通过
     */
    private void handleCheck(Activity activity, boolean isPass) {
        if (isPass) {
            passSatisfy();
        } else {
            if (activity != null) {
                activity.finish();
            }
        }
    }

    /**
     *
     * 处理scheme相关的事情
     *
     * @param activity
     * @param conditions
     * @return 是否通过验证
     */
    private boolean handleScheme(Activity activity, Condition[] conditions) {
        boolean passCheck = true;
        if (isStartByScheme(activity)) {
            passCheck = checkCondition(conditions);
            handleCheck(activity, passCheck);
        } else {
            passCheck = true;
            passSatisfy();
        }
        return passCheck;
    }

    /**
     * 检查是否是由scheme开启
     *
     * @return
     */
    private boolean isStartByScheme(Activity activity) {
        boolean isScheme = false;
        if (activity != null) {
            Intent intent = activity.getIntent();
            if (intent != null) {
                isScheme = intent.getBooleanExtra(KEY_IS_SCHEME, false);
            }
        }
        return isScheme;
    }
}
```

可以看到，我们在`onCreate()`方法开始之前，我们注入了scheme的判断逻辑，当条件满足时，直接执行了后面的逻辑；当条件不满足时，将不满足的条件进行缓存，并且`finish()`当前Activity。当正常执行了scheme跳转之后，清空缓存。

其中`SchemeManager`的逻辑如下：

```
public class SchemeManager {
    public static final String KEY_IS_SCHEME = "isScheme";

    public static class Cache {
        public static String next = "";
        public static Condition unSatisfiedCondition;

        public static void save(Condition condition) {
            unSatisfiedCondition = condition;
        }

        public static void clear(){
            next = "";
            unSatisfiedCondition = null;
        }
    }

    public static void reExecuteScheme(Activity activity, Condition condition) {
        if (Cache.unSatisfiedCondition != null) {
            // 当缓存的条件与当前重复执行时触发的条件一致时，再次执行scheme
            if (Cache.unSatisfiedCondition == condition) {
                String scheme = Cache.next;
                executeScheme(activity, scheme);
            }
        }
    }

    public static void executeScheme(Activity activity, String schemeStr) {
        Cache.next = schemeStr;
        Class<? extends Activity> target = null;
        switch (schemeStr) {
            case "startA":
                target = TargetActivityA.class;
                break;
            case "startB":
                target = TargetActivityB.class;
                break;
            case "startC":
                target = TargetActivityC.class;
                break;
            case "startD":
                target = TargetActivityD.class;
                break;
        }

        if (target != null) {
            // 如果当前页和目标页面是同一个页面  则清空缓存  防止递归调用产生死循环
            if (target.equals(activity.getClass())) {
                Cache.clear();
                return;
            }
            Intent intent = new Intent(activity, target);
            intent.putExtra(KEY_IS_SCHEME, true);
            activity.startActivity(intent);
        }
    }
}
```

这里包含有缓存管理的逻辑，以及**执行scheme**和**重复执行scheme**的逻辑。

其中，由于是Demo，执行scheme的逻辑用固定的字符串来代表scheme链接，这里需要你来替换为你自己的业务逻辑，因为你可能有解析scheme参数并且传递到目标Activity的逻辑。

scheme缓存保存了scheme链接和最后一次没有通过的条件枚举值，当下一次条件被触发时，通过`reExecuteScheme`来执行缓存中的scheme，达到继续跳转的效果。

例如，在登录成功的地方，可以执行:

```
reExecuteScheme(this, Condition.LOGIN);
```

来达到登录成功时继续触发**因未登录导致的scheme跳转失败的scheme跳转**（好绕啊...）。

## 愉快的调用

接下来我们在需要加入scheme跳转判断的Activity的类上加入注解判断即可，例如我们在下面的Activity上加入**登录**以及**词书下载**的条件判断：

```
@SchemeCheck(conditions = {Condition.LOGIN, Condition.DOWNLOAD_BOOK})
public class TargetActivityC extends Activity {

    public static void start(Activity activity) {
        activity.startActivity(new Intent(activity, TargetActivityC.class));
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_c);
    }

}
```

是不是很简单呢？

## 注意！这里有坑

目前发现的一个坑就是我们的加入scheme跳转判断的子类Activity中必须有`onCreate`方法才可以正常执行，因为`AspectJ`只能判断当前子类中所触发的子类的方法。也就是说如果即使你对Activity的基类的`onCreate`进行了一层封装，完成了`onCreate`的所有工作，子类也需要复写一下`onCreate`。就像这样：

```
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
```

否则会crash。
