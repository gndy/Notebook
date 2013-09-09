###Django站点管理###

这一章是关于 Django 的自动管理界面。 这个特性是这样起作用的： 它读取你模式中的元数据，然后提供给你一个强大而且可以使用的界面，网站管理者可以用它立即工作。

###django.contrib 包###

Django自动管理工具是django.contrib的一部分。django.contrib是一套庞大的功能集，它是Django基本代码的组成部分，Django框架就是由众多包含附加组件(add-on)的基本代码构成的。 你可以把django.contrib看作是可选的Python标准库或普遍模式的实际实现。 它们与Django捆绑在一起，这样你在开发中就不用“重复发明轮子”了。

管理工具是本书讲述django.contrib的第一个部分。从技术层面上讲，它被称作django.contrib.admin。django.contrib中其它可用的特性，如用户鉴别系统(django.contrib.auth)、支持匿名会话(django.contrib.sessioins)以及用户评注系统(django.contrib.comments)。这些，我们将在第十六章详细讨论。在成为一个Django专家以前，你将会知道更多django.contrib的特性。 目前，你只需要知道Django自带很多优秀的附加组件，它们都存在于django.contrib包里。

###激活管理界面###


第一步，对你的settings文件做如下这些改变：


* 将'django.contrib.admin'加入setting的INSTALLED_APPS配置中 （INSTALLED_APPS中的配置顺序是没有关系的, 但是我们喜欢保持一定顺序以方便人来阅读）


* 保证INSTALLED_APPS中包含'django.contrib.auth'，'django.contrib.contenttypes'和'django.contrib.sessions'，Django的管理工具需要这3个包。 (如果你跟随本文制作mysite项目的话，那么请注意我们在第五章的时候把这三项INSTALLED_APPS条目注释了。现在，请把注释取消。)


* 确保MIDDLEWARE_CLASSES 包含'django.middleware.common.CommonMiddleware' 、'django.contrib.sessions.middleware.SessionMiddleware' 和'django.contrib.auth.middleware.AuthenticationMiddleware' 。(再次提醒，如果有跟着做mysite的话，请把在第五章做的注释取消。


运行 ` python manage.py syncdb ` 。这一步将生成管理界面使用的额外数据库表。 当你把'django.contrib.auth'加进INSTALLED_APPS后，第一次运行syncdb命令时, 系统会请你创建一个超级用户。 如果你不这么作，你需要运行python manage.py createsuperuser来另外创建一个admin的用户帐号，否则你将不能登入admin (提醒一句: 只有当INSTALLED_APPS包含'django.contrib.auth'时，python manage.py createsuperuser这个命令才可用.)* 


第三，将admin访问配置在URLconf(记住，在urls.py中). 默认情况下，命令django-admin.py startproject生成的文件urls.py是将Django admin的路径注释掉的，你所要做的就是取消注释。 请注意，以下内容是必须确保存在的：

    # Include these import statements...
    from django.contrib import admin
    admin.autodiscover()//
    # And include this URLpattern...
    urlpatterns = patterns('',
        (r'^admin/', include(admin.site.urls)),
    )


###将你的Models加入到Admin管理中###

有一个关键步骤我们还没做。 让我们将自己的模块加入管理工具中，这样我们就能够通过这个漂亮的界面添加、修改和删除数据库中的对象了。 我们将继续第五章中的`` book`` 例子。在其中，我们定义了三个模块： Publisher 、 Author 和 Book 。1

在`` books`` 目录下(`` mysite/books`` )，创建一个文件：`` admin.py`` ，然后输入以下代码：

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    admin.site.register(Publisher)
    admin.site.register(Author)
    admin.site.register(Book)


完成后，打开页面 `` http://127.0.0.1:8000/admin/`` ，你会看到一个Books区域，其中包含Authors、Books和Publishers。  （你可能需要先停止，然后再启动服务(`` runserver`` )，才能使其生效。）


###Admin是如何工作的###


当服务启动时，Django从`` url.py`` 引导URLconf，然后执行`` admin.autodiscover()`` 语句。 这个函数遍历INSTALLED_APPS配置，并且寻找相关的 admin.py文件。 如果在指定的app目录下找到admin.py，它就执行其中的代码。4

在`` books`` 应用程序目录下的`` admin.py`` 文件中，每次调用`` admin.site.register()`` 都将那个模块注册到管理工具中。 管理工具只为那些明确注册了的模块显示一个编辑/修改的界面。1

应用程序`` django.contrib.auth`` 包含自身的`` admin.py`` ，所以Users和Groups能在管理工具中自动显示。 其它的django.contrib应用程序，如django.contrib.redirects，其它从网上下在的第三方Django应用程序一样，都会自行添加到管理工具。


###设置字段可选###

指定email字段为可选，你只要编辑Book模块（回想第五章，它在mysite/books/models.py文件里），在email字段上加上blank=True。代码如下：

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(blank=True)



