一.新建项目

`django-admin startproject mysite`

这将在目录下生成一个mysite目录，也就是你的这个Django项目的根目录。它包含了一系列自动生成的目录和文件，具备各自专有的用途。

注意：在给项目命名的时候必须避开Django和Python的保留关键字，比如“django”，“test”等，否则会引起冲突和莫名的错误。对于mysite的放置位置，不建议放在传统的/var/wwww目录下，它会具有一定的数据暴露危险，因此Django建议你将项目文件放在例如/home/mycode类似的位置。

项目结构如下

`.
├── manage.py
└── mysite
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py`

各文件和目录解释：

- 外层的`mysite/`目录与Django无关，只是你项目的容器，可以任意重命名。
- `manage.py`：一个命令行工具，用于与Django进行不同方式的交互脚本，非常重要！
- 内层的`mysite/`目录是真正的项目文件包裹目录，它的名字是你引用内部文件的包名，例如：`mysite.urls`。
- `mysite/__init__.py`:一个定义包的空文件。
- `mysite/settings.py`:项目的主配置文件，非常重要！
- `mysite/urls.py`:路由文件，所有的任务都是从这里开始分配，相当于Django驱动站点的内容表格，非常重要！
- `mysite/wsgi.py`:一个基于WSGI的web服务器进入点，提供底层的网络通信功能，通常不用关心。



二.启动开发服务器

进入项目根目录，run

`python manage.py runserver`

提示如下

`Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

March 29, 2020 - 11:29:40
Django version 2.2.6, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.`

打开浏览器，访问http://127.0.0.1:8000/，看到Django的火箭欢迎界面，就成功了

三.创建投票app

在 Django 中，每一个应用（app）都是一个 Python 包，并且遵循着相同的约定。Django 自带一个工具，可以帮你生成应用的基础目录结构。

app应用与project项目的区别：

- 一个app实现某个功能，比如博客、公共档案数据库或者简单的投票系统；
- 一个project是配置文件和多个app的集合，这些app组合成整个站点；
- 一个project可以包含多个app；
- 一个app可以属于多个project！

app的存放位置可以是任何地点，但是通常都将它们放在与`manage.py`脚本同级的目录下，这样方便导入文件。

进入mysite项目根目录，确保与`manage.py`文件处于同一级，输入下述命令：

`python mnage.py startapp polls`

系统自动生成polls的目录

`.
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py`

四 编写第一个视图

在polls/views.py中，编写代码

```python
from django.http import HttpResponse

def index(request):
	return HttpResponse("Hello,world. You're at the polls index.")
```

为了调用该views,我们需要编写urlconf,也就是路由，现在，在polls目录中新建一个文件，命名为urls.py并写入代码:

```python
from django.urls import path
from . import views
urlpatterns=[
	path('',views.index,name='index'),
]
```

此时，目录如下

`.
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
├── urls.py
└── views.py`

接下来在主urls.py中添加urlpatterns条目，指向刚才polls中新建的urls文件

include语法相当于多级路由，它把接收到的url地址去除与此项匹配的部分，将剩下的字符串传递给下一级路由urlconf进行判断。在路由的章节，有更加详细的用法指导。

include的背后是一种即插即用的思想。项目根路由不关心具体app的路由策略，只管往指定的二级路由转发，实现了应用解耦。app所属的二级路由可以根据自己的需要随意编写，不会和其它的app路由发生冲突。app目录可以放置在任何位置，而不用修改路由。这是软件设计里很常见的一种模式。

建议：除了admin路由外，尽量给每个app设计自己独立的二级路由。

好了，路由设置成功后，启动服务器，然后在浏览器中访问地址`http://localhost:8000/polls/`。一切正常的话，你将看到`“Hello, world. You’re at the polls index.”`

### **path()方法：**

路由系统中最重要的path()方法可以接收4个参数，其中2个是必须的：`route`和`view`，以及2个可选的参数：`kwargs`和`name`。

**route：**

route 是一个匹配 URL 的准则（类似正则表达式）。当 Django 响应一个请求时，它会从 urlpatterns  的第一项开始，按顺序依次匹配列表中的项，直到找到匹配的项，然后执行该条目映射的视图函数或下级路由，其后的条目将不再继续匹配。因此，url路由的编写顺序非常重要！

需要注意的是，route不会匹配 GET 和 POST 参数或域名。例如，URLconf 在处理请求 `https://www.example.com/myapp/`时，它会尝试匹配 `myapp/`。处理请求 `https://www.example.com/myapp/?page=3` 时，也只会尝试匹配 `myapp/`。

**view：**

view指的是处理当前url请求的视图函数。当Django匹配到某个路由条目时，自动将封装的`HttpRequest`对象作为第一个参数，被“捕获”的参数以关键字参数的形式，传递给该条目指定的视图view。

**kwargs：**

任意数量的关键字参数可以作为一个字典传递给目标视图。

**name：**

对你的URL进行命名，让你能够在Django的任意处，尤其是模板内显式地引用它。这是一个非常强大的功能，相当于给URL取了个全局变量名，不会将url匹配地址写死。

path()方法的四个参数，每个都非常有讲究，这里先做基本的介绍，在后面有详细的论述。