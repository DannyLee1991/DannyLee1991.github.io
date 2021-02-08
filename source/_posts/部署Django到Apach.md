title: 部署Django到Apach
tags:
  - django
categories:
  - django
comments: true
date: 2017-1-09 14:21:58
---

以ubuntu为例，部署Django到Apache上

## 1.安装apache2

```
sudo apt-get install apache2
```

## 2.安装mod_wsgi

Python2：

```
sudo apt-get install libapache2-mod-wsgi
```

Python3:

```
sudo apt-get install libapache2-mod-wsgi-py3
```

## 3.准备一个新网站的配置文件

```
sudo vi /etc/apache2/sites-available/sitename.conf
```

内容如下：

```
<VirtualHost *:8888>
    ServerName www.yourdomain.com
    ServerAlias otherdomain.com
    ServerAdmin tuweizhong@163.com
  
    Alias /media/ /home/tu/blog/media/
    Alias /static/ /home/tu/blog/static/
  
    <Directory /home/tu/blog/media>
        Require all granted
    </Directory>
  
    <Directory /home/tu/blog/static>
        Require all granted
    </Directory>
  
    WSGIScriptAlias / /home/tu/blog/blog/wsgi.py
  
    <Directory /home/tu/blog/blog>
    <Files wsgi.py>
        Require all granted
    </Files>
    </Directory>
</VirtualHost>
```

## 4.修改Django项目中wsgi.py文件

上面的配置中写的 WSGIScriptAlias / /home/tu/blog/blog/wsgi.py

就是把apache2和你的网站project联系起来了

```
import os
from os.path import join,dirname,abspath
 
PROJECT_DIR = dirname(dirname(abspath(__file__)))#3
import sys # 4
sys.path.insert(0,PROJECT_DIR) # 5
 
os.environ["DJANGO_SETTINGS_MODULE"] = "blog.settings" # 7
 
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

> 第 3，4，5 行为新加的内容，作用是让脚本找到django项目的位置，也可以在sitename.conf中做，用WSGIPythonPath,想了解的自行搜索, 第 7 行如果一台服务器有多个django project时一定要修改成上面那样，否则访问的时候会发生网站互相串的情况，即访问A网站到了B网站，一会儿正常，一会儿又不正常（当然也可以使用 mod_wsgi daemon 模式,点击这里查看）

## 5.监听端口

```
sudo vim /etc/apache2/ports.conf
```

将网站的端口加入监听列表：

```

Listen 80
Listen 8888

<IfModule ssl_module>
	Listen 443
</IfModule>

<IfModule mod_gnutls.c>
	Listen 443
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

## 6.设置目录和文件权限

一般目录权限设置为 755，文件权限设置为 644 

假如项目位置在 /home/tu/zqxt （在zqxt 下面有一个 manage.py，zqxt 是项目名称）

```
cd /home/tu/
sudo chmod -R 644 zqxt
sudo find zqxt -type d -exec chmod 755 \{\} \;
```

**apache 服务器运行用户可以在 /etc/apache2/envvars 文件里面改，这里使用的是默认值，当然也可以更改成自己的当前用户，这样的话权限问题就简单很多，但在服务器上推荐有 www-data 用户，更安全。以下是默认设置：**

```
# Since there is no sane way to get the parsed apache2 config in scripts, some
# settings are defined via environment variables and then used in apache2ctl,
# /etc/init.d/apache2, /etc/logrotate.d/apache2, etc.
 
export APACHE_RUN_USER=www-data
export APACHE_RUN_GROUP=www-data
```

### 上传文件夹权限

media 文件夹一般用来存放用户上传文件，static 一般用来放自己网站的js，css，图片等，在settings.py中的相关设置

STATIC_URL 为静态文件的网址 STATIC_ROOT 为静态文件的根目录，

MEDIA_URL 为用户上传文件夹的根目录，MEDIA_URL为对应的访问网址

在settings.py中设置：

```
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/dev/howto/static-files/
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR,'static')
 
# upload folder
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR,'media')
```

在 Linux 服务器上，用户上传目录还要设置给 www-data 用户的写权限，下面的方法比较好，不影响原来的用户的编辑。

假如上传目录为 zqxt/media/uploads 文件夹,进入media文件夹，将 uploads 用户组改为www-data，并且赋予该组写权限:

```
cd media/ # 进入media文件夹
sudo chgrp -R www-data uploads
sudo chmod -R g+w uploads
```

> **备注**：这两条命令，比直接用sudo chown -R www-data:www-data uploads 好，因为下面的命令不影响文件原来所属用户编辑文件，fedora系统应该不用设置上面的权限，但是个人强烈推荐用ubuntu,除非你对linux非常熟悉，你自己选择。

如果你使用的是sqlite3数据库，还会提示 Attempt to write a readonly database,同样要给www-data写数据库的权限

进入项目目录的上一级，比如project目录为 /home/tu/blog 那就进入 /home/tu 执行下面的命令（和修改上传文件夹类似）

```
sudo chgrp www-data blog
sudo chmod g+w blog
sudo chgrp www-data blog/db.sqlite3  # 更改为你的数据库名称
sudo chmod g+w blog/db.sqlite3
```

> **备注**：上面的不要加 -R ,-R是更改包括所有的子文件夹和文件，这样不安全。个人建议可以专门弄一个文件夹,用它来放sqlite3数据库，给该文件夹www-data写权限，而不是整个项目给写权限，有些文件只要读的权限就够了，给写权限会造成不安全。

## 7.激活新网站

```
sudo a2ensite sitename 或 sudo a2ensite sitename.conf
```

### 8.如果遇到了错误

如果你重启apache时遇到了以下错误：

```
Job for apache2.service failed. See "systemctl status apache2.service" and "journalctl -xe" for details.
```

尝试运行

```
systemctl status apache2.service
```

得到如下结果时：

```
apache2.service - (null)
   Loaded: loaded (/etc/init.d/apache2)
   Active: failed (Result: exit-code) since Sat 2015-05-30 02:22:41 IST; 12s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 4866 ExecStart=/etc/init.d/apache2 start (code=exited, status=1/FAILURE)
```

你可以考虑完全卸载apache，并清空配置，重新再来配置一次：

重装APACHE2：

替换已经删除的配置文件，而不重新安装:

```
sudo apt-get -o DPkg::Options::="--force-confmiss" --reinstall install apache2
```

完全移除apache2

```
sudo apt-get purge apache2
```

安装apache2

```
sudo apt-get install apache2
```

参考文章：

- [Django 部署(Apache)](http://www.ziqiangxuetang.com/django/django-deploy.html)
- [Apache not able to restart](http://askubuntu.com/questions/629995/apache-not-able-to-restart)