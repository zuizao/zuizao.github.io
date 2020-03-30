part2:模型与后台

一、数据库配置

打开`mysite/settings.py`配置文件，这是整个Django项目的设置中心。Django默认使用SQLite数据库，因为Python源生支持SQLite数据库，所以你无须安装任何程序，就可以直接使用它。当然，如果你是在创建一个实际的项目，可以使用类似PostgreSQL的数据库，避免以后数据库迁移的相关问题。

如果你想使用其他的数据库，请先安装相应的数据库操作模块，并将settings文件中DATABASES位置的`’default’`的键值进行相应的修改，用于连接你的数据库。其中：

- ENGINE（引擎）：可以是`django.db.backends.sqlite3`、`django.db.backends.postgresql`、`django.db.backends.mysql`、`django.db.backends.oracle`，当然其它的也行。
- NAME（名称）：类似Mysql数据库管理系统中用于保存项目内容的数据库的名字。如果你使用的是默认的SQLite，那么数据库将作为一个文件将存放在你的本地机器内，此时的NAME应该是这个文件的完整绝对路径包括文件名，默认值`os.path.join(BASE_DIR, ’db.sqlite3’)`，将把该文件储存在你的项目目录下。

**注意**：

- **在使用非SQLite的数据库时，请务必预先在数据库管理系统的提示符交互模式下创建数据库，你可以使用命令：`CREATE DATABASE database_name;`。Django不会自动帮你做这一步工作。**
- 确保你在settings文件中提供的数据库用户具有创建数据库表的权限，因为在接下来的教程中，我们需要自动创建一个test数据表。（在实际项目中也需要确认这一条要求。）
- 如果你使用的是SQLite，那么你无需做任何预先配置，直接使用就可以了。

在修改settings文件时，请顺便将`TIME_ZONE`设置为国内所在的时区`Asia/Shanghai`。

同时，请注意settings文件中顶部的`INSTALLED_APPS`设置项。它列出了所有的项目中被激活的Django应用（app）。你必须将你自定义的app注册在这里。每个应用可以被多个项目使用，并且可以打包和分发给其他人在他们的项目中使用。

默认情况，`INSTALLED_APPS`中会自动包含下列条目，它们都是Django自动生成的：

- django.contrib.admin：admin管理后台站点
- django.contrib.auth：身份认证系统
- django.contrib.contenttypes：内容类型框架
- django.contrib.sessions：会话框架
- django.contrib.messages：消息框架
- django.contrib.staticfiles：静态文件管理框架

上面的一些应用也需要建立一些数据库表，所以在使用它们之前我们要在数据库中创建这些表。使用下面的命令创建数据表：

`python manage.py migrate`

migrate命令将遍历`INSTALLED_APPS`设置中的所有项目，在数据库中创建对应的表，并打印出每一条动作信息。如果你感兴趣，可以在你的数据库命令行下输入：`\dt` (PostgreSQL)、 `SHOW TABLES;`(MySQL)或 `.schema`(SQLite) 来列出 Django 所创建的表。

提示：对于极简主义者，你完全可以在INSTALLED_APPS内注释掉任何或者全部的Django提供的通用应用。这样，migrate也不会再创建对应的数据表。

二、创建模型

现在，我们来定义模型model，模型本质上就是数据库表的布局，再附加一些元数据。

Django通过自定义Python类的形式来定义具体的模型，每个模型的物理存在方式就是一个Python的类Class，每个模型代表数据库中的一张表，每个类的实例代表数据表中的一行数据，类中的每个变量代表数据表中的一列字段。Django通过模型，将Python代码和数据库操作结合起来，实现对SQL查询语言的封装。也就是说，你可以不会管理数据库，可以不会SQL语言，你同样能通过Python的代码进行数据库的操作。Django通过ORM对数据库进行操作，奉行代码优先的理念，将Python程序员和数据库管理员进行分工解耦。

在这个简单的投票应用中，我们将创建两个模型：`Question`和`Choice`。Question包含一个问题和一个发布日期。Choice包含两个字段：该选项的文本描述和该选项的投票数。每一条Choice都关联到一个Question。这些都是由Python的类来体现，编写的全是Python的代码，不接触任何SQL语句。现在，编辑`polls/models.py`文件，具体代码如下：

```python
from django.db import models

class Question(models.Model):
	question_text=models.CharField(max_length=200)
	pub_date=models.DatetimeField('date published')

class Choice(models.Model):
	question=models.ForeignKey(Question,on_delete=models.CASCADE)
	choice_text=models.CharField(max_length=200)
	votes=models.IntegerField(default=0)
```

上面的代码非常简单明了。每一个类都是`django.db.models.Model`的子类。每一个字段都是`Field`类的一个实例，例如用于保存字符数据的CharField和用于保存时间类型的DateTimeField，它们告诉Django每一个字段保存的数据类型。

每一个Field实例的名字就是字段的名字（如： question_text 或者 pub_date ）。在你的Python代码中会使用这个值，你的数据库也会将这个值作为表的列名。

你也可以在每个Field中使用一个可选的第一位置参数用于提供一个人类可读的字段名，让你的模型更友好，更易读，并且将被作为文档的一部分来增强代码的可读性。

一些Field类必须提供某些特定的参数。例如CharField需要你指定max_length。这不仅是数据库结构的需要，同样也用于数据验证功能。

有必填参数，当然就会有可选参数，比如在votes里我们将其默认值设为0.

最后请注意，我们使用`ForeignKey`定义了一个外键关系。它告诉Django，每一个Choice关联到一个对应的Question（注意要将外键写在‘多’的一方）。Django支持通用的数据关系：一对一，多对一和多对多。

三、启用模型

上面的代码看着有点少，其实包含了大量的信息，据此，Django会做下面两件事：

- 创建该app对应的数据库表结构
- 为Question和Choice对象创建基于Python的数据库访问API

但是，首先我们得先告诉Django项目，我们要使用投票app。

要将应用添加到项目中，需要在`INSTALLED_APPS`设置中增加指向该应用的配置文件的链接。对于本例的投票应用，它的配置类文件PollsConfig是`polls/apps.py`，所以它的点式路径为`polls.apps.PollsConfig`。我们需要在`INSTALLED_APPS`中，将该路径添加进去：

```python
#mysite/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls,apps.PollsConfig',
]
```

实际上，在多数情况下，写成'polls'就可以

```python
#mysite/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls',
]
```

现在Django已经知道你的投票应用的存在了，并把它加入了项目大家庭。

我们需要再运行下一个命令：

`python manage.py makemigrations polls`

你会看到

```bash
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice

```

通过运行`makemigrations`命令，Django 会检测你对模型文件的修改，也就是告诉Django你对模型有改动，并且你想把这些改动保存为一个“迁移(migration)”。

`migrations`是Django保存模型修改记录的文件，这些文件保存在磁盘上。在例子中，它就是`polls/migrations/0001_initial.py`，你可以打开它看看，里面保存的都是人类可读并且可编辑的内容，方便你随时手动修改。

接下来有一个叫做`migrate`的命令将对数据库执行真正的迁移动作。但是在此之前，让我们先看看在migration的时候实际执行的SQL语句是什么。有一个叫做`sqlmigrate`的命令可以展示SQL语句，例如：

`python  manage.py sqlmigrate polls 0001`

显示文本如下

```bash
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "question_text" varchar(200) NOT NULL, "pub_date" datetime NOT NULL);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "choice_text" varchar(200) NOT NULL, "votes" integer NOT NULL, "question_id" integer NOT NULL REFERENCES "polls_question" ("id") DEFERRABLE INITIALLY DEFERRED);
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");
COMMIT;

```

**请注意：**

- 实际的输出内容将取决于您使用的数据库会有所不同。上面的是PostgreSQL的输出。
- 表名是自动生成的，通过组合应用名 (polls) 和小写的模型名`question`和`choice` 。 ( 你可以重写此行为。)
- 主键 (IDs) 是自动添加的。( 你也可以重写此行为。)
- 按照惯例，Django 会在外键字段名上附加 "_id" 。 (你仍然可以重写此行为。)
- 生成SQL语句时针对你所使用的数据库，会为你自动处理特定于数据库的字段，例如 auto_increment (MySQL),  serial (PostgreSQL), 或integer primary key (SQLite) 。 在引用字段名时也是如此 –  比如使用双引号或单引号。
- 这些SQL命令并没有在你的数据库中实际运行，它只是在屏幕上显示出来，以便让你了解Django真正执行的是什么。

如果你感兴趣，也可以运行`python manage.py check`命令，它将检查项目中的错误，并不实际进行迁移或者链接数据库的操作。

现在，我们可以运行migrate命令，在数据库中进行真正的表操作了。

```bash
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK

```

migrate命令对所有还未实施的迁移记录进行操作，本质上就是将你对模型的修改体现到数据库中具体的表上面。Django通过一张叫做django_migrations的表，记录并跟踪已经实施的migrate动作，通过对比获得哪些migrations尚未提交。

migrations的功能非常强大，允许你随时修改你的模型，而不需要删除或者新建你的数据库或数据表，在不丢失数据的同时，实时动态更新数据库。我们将在后面的章节对此进行深入的阐述，但是现在，只需要记住**修改模型时的操作分三步**：

- 在models.py中修改模型；
- 运行`python manage.py makemigrations`为改动创建迁移记录；
- 运行`python manage.py migrate`，将操作同步到数据库。

之所以要将创建和实施迁移的动作分成两个命令两步走是因为你也许要通过版本控制系统（例如github，svn）提交你的项目代码，如果没有一个中间过程的保存文件（migrations），那么github如何知道以及记录、同步、实施你所进行过的模型修改动作呢？毕竟，github不和数据库直接打交道，也没法和你本地的数据库通信。但是分开之后，你只需要将你的migration文件（例如上面的0001）上传到github，它就会知道一切。

四、使用模型的API

下面，让我们进入Python交互环境，尝试使用Django提供的数据库访问API。要进入Python的shell，请输入命令：

`python manage.py shell`

相比较直接输入“python”命令的方式进入Python环境，调用`manage.py`参数能将`DJANGO_SETTINGS_MODULE`环境变量导入，它将自动按照`mysite/settings.py`中的设置，配置好你的python shell环境，这样，你就可以导入和调用任何你项目内的模块了。

或者你也可以这样，先进入一个纯净的python shell环境，然后启动Django，具体如下：

```python
In [1]: import django                                                           

In [2]: django.setup()                                                          

In [3]: from polls.models import Question,Choice                                

In [4]: Question.objects.all()                                                  
Out[4]: <QuerySet []>
In[6]from django.utils import timezone                                       

In [7]: q=Question(question_text="What's new?",pub_date=timezone.now())         

In [8]: q.save()                                                                

In [9]: q.id                                                                    
Out[9]: 1

In [10]: q.question_text                                                        
Out[10]: "What's new?"

In [11]: q.pub_date                                                             
Out[11]: datetime.datetime(2020, 3, 29, 12, 40, 29, 650546, tzinfo=<UTC>)

In [12]: q.question_text="What's up?"                                           

In [13]: q.save()                                                               

In [14]: Question.objects.all()                                                 
Out[14]: <QuerySet [<Question: Question object (1)>]>


```

这里等一下：上面的``是一个不可读的内容展示，你无法从中获得任何直观的信息，为此我们需要一点小技巧，让Django在打印对象时显示一些我们指定的信息。

返回`polls/models.py`文件，修改一下question和Choice这两个类，代码如下：

```python
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

这个技巧不但对你打印对象时很有帮助，在你使用Django的admin站点时也同样有帮助。

另外，这里我们自定义一个模型的方法，用于判断问卷是否最近时间段内发布度的：

```python
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

请注意上面分别导入了两个关于时间的模块，一个是python内置的datetime一个是Django工具包提供的timezone。

保存修改后，我们重新启动一个新的python shell，再来看看其他的API:

```python
In [1]: from polls.models import Question,Choice                                

In [2]: Question.objects.all()                                                  
Out[2]: <QuerySet [<Question: What's up?>]>

In [3]: Question.objects.filter(id=1)                                           
Out[3]: <QuerySet [<Question: What's up?>]>

In [4]: Question.objects.filter(question_text__startswith='What')               
Out[4]: <QuerySet [<Question: What's up?>]>

In [5]: from django.utils import timezone                                       

In [6]: current_year=timezone.now().year                                        

In [7]: Question.objects.get(pub_date__year=current_year)                       
Out[7]: <Question: What's up?>

In [8]: Question.objects.get(id=2)                                              
---------------------------------------------------------------------------
DoesNotExist                              Traceback (most recent call last)
...
DoesNotExist: Question matching query does not exist.

In [16]: q.choice_set.create(choice_text='Not much',votes=0)               
Out[16]: <Choice: Not much>

In [17]: q.choice_set.create(choice_text='The sky',votes=0)                
Out[17]: <Choice: The sky>

In [18]: c=q.choice_set.create(choice_text='Just hacking again',votes=0)   

In [19]: c.question                                                        
Out[19]: <Question: What's up?>

In [20]: q.choice_set.all()                                                
Out[20]: <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

In [21]: q.choice_set.count()                                              
Out[21]: 3
In [23]: Choice.objects.filter(question__pub_date__year=current_year)      
Out[23]: <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

In [24]: c=q.choice_set.filter(choice_text__startswith='Just hacking')     

In [25]: c.delete()                                                        
Out[25]: (1, {'polls.Choice': 1})

```

关于模型的使用就暂时先介绍这么多。这部分内容是Django项目的核心，也是动态网站与数据库交互的核心，对于初学者，再难理解也要理解

五、admin后台管理站点

很多时候，我们不光要开发针对客户使用的前端页面，还要给后台管理人员提供相应的管理界面。但是大多数时候为你的团队或客户编写用于增加、修改和删除内容的后台管理站点是一件非常乏味的工作并且没有多少创造性，而且也需要花不少的时间和精力。Django最大的优点之一，就是体贴的为你提供了一个基于项目model创建的一个后台管理站点admin。这个界面只给站点管理员使用，并不对大众开放。虽然admin的界面可能不是那么美观，功能不是那么强大，内容不一定符合你的要求，但是它是免费的、现成的，并且还是可定制的，有完善的帮助文档，那么，你还要什么自行车？

`python manage.py createsuperuser`

2.启动开发服务器

服务器启动后，在浏览器访问`http://127.0.0.1:8000/admin/`。你就能看到admin的登陆界面了

在实际环境中，为了站点的安全性，我们一般不能将管理后台的url随便暴露给他人，不能用`/admin/`这么简单的路径。

打开根url路由文件`mysite/urls.py`，修改其中admin.site.urls对应的表达式，换成你想要的，比如：

```python
#mysite/urls.py
from django.contrib import admin
from django.urls import include,path

urlpatterns = [
	path('polls/',include('polls.urls')),
    path('control/', admin.site.urls),
]

```

3.进入admin站点

利用刚才建立的admin账户，登陆admin，你将看到如下的界面：

当前只有两个可编辑的内容：groups和users。它们是`django.contrib.auth`模块提供的身份认证框架。

4.在admin中注册投票应用

现在还无法看到投票应用，必须先在admin中进行注册，告诉admin站点，请将polls的模型加入站点内，接受站点的管理。

打开`polls/admin.py`文件，加入下面的内容：

```python
from django.contrib import admin
from . models import Question

admin.site.register(Question)
```

注册question模型后，刷新admin页面就能看到Question栏目了。

点击“Questions”，进入questions的修改列表页面。这个页面会显示所有的数据库内的questions对象，你可以在这里对它们进行修改。看到下面的`“What’s up?”`了么？它就是我们先前创建的一个question对象，并且通过`__str__`方法的帮助，显示了较为直观的信息，而不是一个冷冰冰的对象类型名称。

下面，点击`What’s up?`进入编辑界面：

这里需要注意的是：

- 页面中的表单是由Question模型自动生成的。
- 不同的模型字段类型(DateTimeField, CharField)会表现为不同的`HTML input`框类型。
- 每一个`DateTimeField`都会自动生成一个可点击链接。日期是Today，并有一个日历弹出框；时间是Now，并有一个通用的时间输入列表框。

在页面的底部，则是一些可选项按钮：

- `delete`：弹出一个删除确认页面
- `save and add another`：保存当前修改，并加载一个新的空白的当前类型对象的表单。
- `save and continue editing`：保存当前修改，并重新加载该对象的编辑页面。
- `save`：保存修改，返回当前对象类型的列表页面。

如果`Date published`字段的值和你在前面教程创建它的时候不一致，可能是你没有正确的配置`TIME_ZONE`，在国内，通常是8个小时的时间差别。修改TIME_ZONE配置并重新加载页面，就能显示正确的时间了。

在页面的右上角，点击`History`按钮，你会看到你对当前对象的所有修改操作都在这里有记录，包括修改时间和操作人员