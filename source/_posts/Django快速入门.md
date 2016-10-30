title: Django快速入门
author: Adam
date: 2016-10-29 12:53:26
# 类别
categories:
  - blog
# 标签
tags:
  - Django
  - Python
---
作者：Adam at 2016-10-29 12:53:26
(注：本教程来源于[`官方最新Django教程`](http://https://docs.djangoproject.com/en/1.10/))

作为一名前端开发，了解后端技术很有必要。作为一名`web`程序员，掌握`web`开发全栈技能，成为未来发展的必然趋势。既然聘宝的研发以`Python`开发为主，我们有必要先学习一下`Django`这个开发框架。

<!-- more -->

首先，为了快速进入学习，我们假设你已经安装好聘宝研发的`Python+Docker`开发环境。接着我们建立虚拟开发环境。

```python
➜  ~ mkdir myFirstDjango
➜  ~ cd myFirstDjango
➜  myFirstDjango mkvirtualenv myFirstDjango
New python executable in myFirstDjango/bin/python
Installing setuptools, pip, wheel...done.
(myFirstDjango)➜  myFirstDjango
```

安装Django

```python
(myFirstDjango)➜  myFirstDjango pip install django
You are using pip version 7.1.0, however version 8.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Collecting django
  Downloading http://pypi.doubanio.com/packages/8a/09/46f790104abca7eb93786139d3adde9366b1afd59a77b583a1f310dc8cbd/Django-1.10.2-py2.py3-none-any.whl (6.8MB)
    100% |████████████████████████████████| 6.8MB 1.5MB/s
Installing collected packages: django
Successfully installed django-1.10.2
```

创建一个项目mysite
```python
(myFirstDjango)➜ django-admin startproject mysite

项目文件列表
------------------
mysite/					项目名称
    manage.py			命令行工具
    mysite/				代码目录
        __init__.py		模块化申明文件，一般为空
        settings.py		配置文件
        urls.py			路由配置文件
        wsgi.py			服务器文件

启动本地开发环境：
------------------
(myFirstDjango)➜  myFirstDjango cd mysite && python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).

You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

October 29, 2016 - 05:47:37
Django version 1.10.2, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.

启动前需要用 migrate 做些数据库迁移修复工作：
------------------
(myFirstDjango)➜  mysite python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK

再次启动开发环境：
------------------
(myFirstDjango)➜ python manage.py runserver 0.0.0.0:8000
```

浏览器访问： http://127.0.0.1:8000/

![](http://img.pinbot.me:8080/uploads/2016/10/29/blob_1477720230271.png)

第一步完成了，我们还需要澄清一个细节，注意看这两个命令：

`django-admin startproject` 和 `python manage.py startapp polls`

这里面的`project`和`app`是有区别的：

`app`可以看作是一个完成功能模块，而`project`可以看作成一个网站，由多个功能模块`app`组成。关键是模块`app`可以被多个`project`直接使用，这点非常重要，`DRY`万岁。

我们运行`python manage.py startapp polls`，生成一个新的投票模块`polls`。可以看到`mysite`目录下多了一个`polls`目录。
```python
mysite/
    manage.py
    mysite/
    polls/
        __init__.py
        admin.py
        apps.py
        migrations/
            __init__.py
        models.py
        tests.py
        views.py
```

接着，我们需要补充路由文件`urls.py`
```python
#urls.py
from django.conf.urls import url
from . import views
urlpatterns = [
	url(r'^$', views.index, name='index'),
]
```

同样在`site`目录下也需要补充一个：

```python
#urls.py
from django.conf.urls import include, url
from django.contrib import admin
urlpatterns = [
	url(r'^polls/', include('polls.urls')),  #包含polls模块的路由
	url(r'^admin/', admin.site.urls),
]
```

同时在mysite/settings.py修改：
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',	#引入安装的模块
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

最后增加模版文件`mysite/polls/templates/polls/index.html`
```python
{% if latest_question_list %} <ul>
{% for question in latest_question_list %}
<li><a href="/polls/{{ question.id }}/">
{{ question.question_text }}</a>
</li>
{% endfor %}
</ul> {% else %}
<p>暂无投票。</p> {% endif %}
```

在`mysite`目录下，再次启动开发环境：
```python
(myFirstDjango)➜ python manage.py runserver 0.0.0.0:8000
```

访问`http://0.0.0.0:8000/polls/`可以看到：
```python
暂无投票。
```
对，还没有投票内容。我们还需要建立`Model`制定数据结构，添加投票数据，然后从数据库获取投票数据。

为了讲解方便我们先使用`sqlite`作为默认的数据库存储数据：
```python
#mysite/settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
		#数据库文件保存的位置
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3')
    }
}
```

首先，编辑`polls/models.py`，制定`Model`数据：

```python
from __future__ import unicode_literals
from django.db import models
from django.utils.encoding import python_2_unicode_compatible
@python_2_unicode_compatible
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    def __str__(self):
        return self.question_text

@python_2_unicode_compatible
class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
    def __str__(self):
        return self.choice_text
```

然后利用`makemigrations`工具生成数据库迁移文件`polls/migrations/0001_initial.py`
```python
(myFirstDjango)➜  mysite python manage.py makemigrations polls
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field question to choice
```
接着执行数据库迁移操作，这里应该包含建表的操作，这样我们就可以通过管理工具添加投票数据了。
```python
(myFirstDjango)➜  mysite python manage.py sqlmigrate polls 0001
BEGIN;
--
-- Create model Choice
--
CREATE TABLE "polls_choice" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "choice_text" varchar(200) NOT NULL, "votes" integer NOT NULL);
--
-- Create model Question
--
CREATE TABLE "polls_question" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "question_text" varchar(200) NOT NULL, "pub_date" datetime NOT NULL);
--
-- Add field question to choice
--
ALTER TABLE "polls_choice" RENAME TO "polls_choice__old";
CREATE TABLE "polls_choice" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "choice_text" varchar(200) NOT NULL, "votes" integer NOT NULL, "question_id" integer NOT NULL REFERENCES "polls_question" ("id"));
INSERT INTO "polls_choice" ("choice_text", "votes", "id", "question_id") SELECT "choice_text", "votes", "id", NULL FROM "polls_choice__old";
DROP TABLE "polls_choice__old";
CREATE INDEX "polls_choice_7aa0f6ee" ON "polls_choice" ("question_id");
COMMIT;
(myFirstDjango)➜  mysite
```

完成后可以检查下是否迁移有错误发生：
```python
(myFirstDjango)➜  mysite python manage.py check
System check identified no issues (0 silenced).
```

或者直接使用`migrate`命令执行所有未执行的迁移操作。
```python
(myFirstDjango)➜  mysite python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```

然后我们生成后台管理界面：
```python
(myFirstDjango)➜  mysite python manage.py createsuperuser
Username (leave blank to use 'super'): admin
Email address: yourname@gmail.com
Password:
Password (again):
Superuser created successfully.
```

在`polls/admin.py`里面注册可以管理的`Model`:
```python
from .models import Question, Choice
admin.site.register(Question)
admin.site.register(Choice)
```

然后访问`http://0.0.0.0:8000/admin/`
![](http://img.pinbot.me:8080/uploads/2016/10/31/blob_1477848222580.png)

添加完投票内容后，

![](http://img.pinbot.me:8080/uploads/2016/10/31/blob_1477848419636.png)

![](http://img.pinbot.me:8080/uploads/2016/10/31/blob_1477848476411.png)

修改一下`polls`的路由：
```python
app_name = 'polls'
urlpatterns = [
    url(r'^$', views.index, name='index'),
    # the 'name' value as called by the {% url %} template tag
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
]
```

修改投票主页模版：
```python
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

并且增加投票详细页的模版`detail.html`：
```python
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
<li>{{ choice.choice_text }}</li> {% endfor %}
</ul>
```

访问`http://0.0.0.0:8000/polls/1/`，最简单的`Django`投票样例就完成了。
```python
你会学习python吗？
* 会
* 不会
* 不知道
```

这是个非常简单的`MVC`架构，熟悉`Angular`的同学应该很快就能理解`Django`的做法，怎么样，`Python`也不难吧～

















