title: HelloDjango!--Django开发极速入门
date: 2016-1-12 17:26:55
tags:
  - Python
categories:
  - 工程开发
  - Python
comments: true
---

## About Django

> **[Django](https://www.djangoproject.com/)**是一个开源的python web应用框架，最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站，即是CMS（内容管理系统）软件。于2005年7月在BSD许可证下发布。
> 
> 截至目前，Django的最新版本是1.9.1。Django的名字来源于上世纪五十年代的一位吉普赛吉他手的名字Django Reinhardt，其中‘d’是不发音的，为此django官方还录制了一个[音频](http://red-bean.com/~adrian/django_pronunciation.mp3)来更正大家的读音。
> 

### Django的理念
django是一个高级的python web框架，鼓励开发人员快速开发出**干净务实**的设计。由于是由有经验的web开发者创建的，所以它避免了很多web开发的麻烦。

可以使得开发人员更加专注的写应用程序而不需要重新造轮子。而且django是免费开源的。

**三个特点：**

- 快速：帮助开发人员快速将想法变成现实
- 安全：可以帮助开发人员避免很多常见的安全方面的问题
- 可伸缩性：一些很复杂繁琐的web项目也可以由django来构建

### 都有谁用过Django呢？
Disqus, Instagram, Pinterest以及Mozilla 都已经使用Django很多年了，而且Django目前已经可以承受每秒最高5万的访问量。

## 环境配置

在配置好python2.7运行环境之后，执行：

```
pip install Django==1.9.1
```

即可自动下载并安装Django，如果提示权限不足，可以尝试`sudo pip install Django==1.9.1`。

如果选择通过[下载zip包](https://www.djangoproject.com/download/1.9.1/tarball/)安装，下载成功后，解压zip包，命令行进入到Django-1.9.1目录下，执行：

```
python setup.py install
```

## 创建第一个django项目

命令行执行：

```
django-admin startproject mysite
```

创建mysite项目，`cd` 进入到mysite，我们可以看到生成了如下几个文件：

```
mysite/
    manage.py			＃这是一个与django交互的命令行工具
    mysite/
        __init__.py
        settings.py		＃配置设置
        urls.py			＃声明一系列django用来处理分发的url
        wsgi.py			＃PythonWeb服务器网关接口，相当于一种协议，使得python的web应用可以部署到所有遵循这个协议点服务器上。用于部署django项目的
```

在这里，我们执行

```
python manage.py startapp polls
```

来创建一个名叫polls的app。

我们会发现，在mysite下会多产生一个polls的目录，这个目录的结构如下：

```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py 		#定义moudle
    tests.py
    views.py 		＃定义view
```

### 配置url，让我们的项目run起来

django通过urls.py文件作为路由文件，将获取到的url通过正则匹配去分发到各个app下的urls.py，然后再由app内的urls.py通过正则匹配分发到各个view层中，并执行相关方法。

这里我们分别配置我们主项目和子项目中的urls.py文件：

mysite/urls.py:

```
from django.conf.urls import include
	url(r'^polls/', include('polls.urls')), 
```

polls/urls.py:

```
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```

可以看到子项目中的url指向了views中的index方法，那么我们把views的index进行完善：

polls/views.py:

```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello Django")
```

这个时候我们在命令行中输入：

```
python manage.py runserver
```

把项目跑起来之后，在浏览器输入：http://localhost:8000/polls/ 就可以看到以下输出结果了。

![Hello Django](/img/django_01_01.png)

### 定义models，创建数据库

django遵循DRY原则（Don’t repeat yourself），其实道理很简单，避免数据冗余，它认为冗余的就是坏的。
所以django中定义的每一个model，都应该是单一的，明确的，包含了所有必要信息的。
这样做的目的就是，你在一处定义好model，在任何其他的地方，就可以通过这个model来获取到由他派生出的信息。

修改models.py:

```python
from django.db import models
from django.utils import timezone

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

	#当前model的字符串表示
    def __str__(self):
        return self.question_text

	#判断这条Question是否是当天发出的
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```

mysite下的settings.py:

的INSTALLED_APPS中加入：

```
'polls.apps.PollsConfig',
```

现在，我们可以执行`python manage.py makemigrations`,这条命令会去遍历所有在settings.py中INSTALLED_APPS下配置的app，如果发现对应app的models文件有改动，就会在这个app下的`migrations`目录下生成一个数据库的scheme文件，这里我们可以看到生成了一个`0001_initial.py`文件，这个文件描述了model对应的数据库操作，这个文件是可编辑的。

我们还可以通过执行`python manage.py sqlmigrate polls 0001`命令，查看上面那个scheme文件所对应的SQL语句。

这个时候，其实我们的sql语句并没有执行，数据库也并没有创建，我们需要执行下面这条语句，来让我们的scheme文件生效：

```
python manage.py migrate
```
这下，我们的表就创建成功了。

看似繁琐的操作，其实我们只要记住三步走的套路，就很容易掌握：

- 1.修改models.py文件
- 2.执行python manage.py makemigrations
- 3.执行python manage.py migrate

这里我有一个疑问，就是2，3的操作明明可以一步执行完，为什么还需要分成两条命令呢？

这里官方的解释是：

> The reason that there are separate commands to make and apply migrations is because you’ll commit migrations to your version control system and ship them with your app; they not only make your development easier, they’re also useable by other developers and in production.

我表示依然不理解。

### 通过django的shell来插入数据

django 给我们提供了一个shell，通过执行`python manage.py shell`进入。

下面通过这个shell来插入一些数据，也顺便体验一下django的orm数据库操作。

```
>>> from polls.models import Question, Choice 
>>> from django.utils import timezone
>>> Question.objects.all()
>>> []
>>> q = Question(question_text="What's new?", pub_date=timezone.now())
>>> q.save()
>>> q.id
>>> 1
>>> q.question_text
>>> "What's new?"
>>> q.choice_set.create(choice_text='Not much', votes=0)
>>> <Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
>>> <Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)
>>> c.question
>>> <Question: What's new?>
>>> q.choice_set.all()
>>> [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]
>>> exit()
```

至此，我们已经添加了一部分数据到db中了，我们可以通过django下的一个后台管理模块来查看我们插入的数据。

执行：

```
python manage.py createsuperuser
```
创建用户。

将我们的app注册到admin中。

polls/admin.py：

```
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

输入http://localhost:8000/admin/ 就可以访问后台了。

![admin page](/img/django_01_02.png)

### 配置views 和 urls

views写入：

```
from django.shortcuts import render,HttpResponse

def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

polls/urls.py：

```
from django.conf.urls import url

from . import views

urlpatterns = [
    # ex: /polls/
    url(r'^$', views.index, name='index'),
    # ex: /polls/5/
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    # ex: /polls/5/results/
    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    # ex: /polls/5/vote/
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```

其中：
?P<question_id>定义的名称，是用于识别和匹配模式；[0-9]是一个正则表达式匹配一个数字序列（即一个数）

当然如果你想要的话，也可以写死一个url：
`url(r'^polls/latest\.html$', views.index),`
但是django不建议这么做。

接下来我们可以将models的数据信息显示在view界面上了：

views.py:

```
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

但是这样写前台界面实在太不友好，我们要引入模版机制，将数据信息渲染到html页面上：

创建polls/templates/polls/index.html

用django特有的模版语言写入：

```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

这些模版语言都是比较易懂的流程控制语句，相信不用解释也可以理解。

修改views.py:

```
from django.http import HttpResponse
from django.template import loader

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

还有一种更快捷的写法，使用django封装好的render()方法：

```
from django.shortcuts import render

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

创建polls/templates/polls/detail.html,并写入：

```
{{ question }}
```

polls/views.py：

```
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

这里我们可以看到404的处理都单独封装了一个方法，django这么处理的原因是为了防止模型层和视图层的耦合，因为404是比较常见的一种错误，而且处理404的判断如果放在了界面层，那么对models的处理就和view层耦合太重。

现在访问：`http://localhost:8000/polls`就能看到载入模版之后的样子了。

![add template](/img/django_01_03.png)

内容页面：

![details](/img/django_01_04.png)

注意这里有个细节：

polls/index.html中的a标签的链接我们用了硬编码，为了更好的实现可扩展性，我们将其中的:

```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

改为：

```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

可以看出来新的写法会去找`name`属性为`ditail`的url进行匹配。

### form的实现

修改我们的
`polls/templates/polls/detail.html`:

```
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
{% endfor %}
<input type="submit" value="Vote" />
</form>
```

view界面加入投票逻辑：

```
from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.core.urlresolvers import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.core.urlresolvers import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

其中：

reverse()方法  用于返回一个url，这里直接返回polls/1/vote/

定义投票的结果页面：

```
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

以及模版页面：
`polls/results.html`:

```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

### 静态文件的加载

创建一个css文件：`polls/static/polls/style.css`并写入：

```
li a {
    color: green;
}
```

在polls/templates/polls/index.html文件中引入这个css

```
{% load staticfiles %}
<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />
```

接下来添加一个图片：
`polls/static/polls/images/background.gif.`

修改`polls/static/polls/style.css`加入：

```
body {
    background: white url("images/background.gif") no-repeat right bottom;
}
```

可看到执行效果：

![css](/img/django_01_05.png)

### 自定义后台页面

Django经常被用来开发后台应用，因为它有一个非常强大的，可灵活配置的后台应用。这个admin应用就是`settings.py`中的`INSTALLED_APPS`下的`django.contrib.admin`。

前面我们在polls项目下的admin.py中将我们的`Question`注册到了admin中，我们在后台页面可以看到了Question表的相关信息。那么如果我们想把`Choice`的相关信息也显示出来需要怎么处理呢？

有一种做法就是也仿照`Question`将`Choice`注册进入admin：

```
from django.contrib import admin
from .models import Question,Choice

admin.site.register(Question)
admin.site.register(Choice)
```

但这样并不好，因为Choice和Question存在一对多的外键关联，我们直接显示出来，并不能很好的体现出这层含义。为了解决这个问题，我们可以引入`ModelAdmin`，将Question注册到一个自定义的`ModelAdmin`上，定义其内联元素为一个Choice。

```

from .models import Question,Choice

class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 0

class QuestionAdmin(admin.ModelAdmin):
    inlines = [ChoiceInline]

admin.site.register(Question,QuestionAdmin)
```

在Question的详情页面里，我们就可以看到作为内联样式展示的Choices了：

![内联](/img/django_01_06.png)

接下来我们可以通过对`QuestionAdmin`简单的配置，实现一个功能更加强大的后台界面：

```
class QuestionAdmin(admin.ModelAdmin):
	# 详情页加入更多字段的显示
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]
    
    # 列表页面加入更多字段的显示
    list_display = ('question_text', 'pub_date', 'was_published_recently')
    
    # 在列表页面右侧加入过滤功能
    list_filter = ['pub_date']
    # 加入搜索功能
    search_fields = ['question_text']
```

实现效果如下：

![完整后台](/img/django_01_07.png)