##第5章 模型##

###在视图中进行数据库查询的笨方法###

我们使用了 MySQLdb 类库（可以从 http://www.djangoproject.com/r/python-mysql/ 获得）来连接 MySQL 数据库，取回一些记录，将它们提供给模板以显示一个网页：
    
    from django.shortcuts import render_to_response
    import MySQLdb

    def book_list(request):
        db = MySQLdb.connect(user='me', db='mydb', passwd='secret', host='localhost')
        cursor = db.cursor()
        cursor.execute('SELECT name FROM books ORDER BY name')
        names = [row[0] for row in cursor.fetchall()]
        db.close()
        return render_to_response('book_list.html', {'names': names})

* 我们将数据库连接参数硬行编码于代码之中。 理想情况下，这些参数应当保存在 Django 配置中。1

* 我们不得不重复同样的代码： 创建数据库连接、创建数据库游标、执行某个语句、然后关闭数据库。 理想情况下，我们所需要应该只是指定所需的结果。1

* 它把我们栓死在 MySQL 之上。 如果过段时间，我们要从 MySQL 换到 PostgreSQL，就不得不使用不同的数据库适配器（例如 psycopg 而不是 MySQLdb ），改变连接参数，根据 SQL 语句的类型可能还要修改SQL 。 理想情况下，应对所使用的数据库服务器进行抽象，这样一来只在一处修改即可变换数据库服务器。 

###MTV 开发模式###

让我们先花点时间考虑下 Django 数据驱动 Web 应用的总体设计。

我们在前面章节提到过，Django 的设计鼓励松耦合及对应用程序中不同部分的严格分割。 遵循这个理念的话，要想修改应用的某部分而不影响其它部分就比较容易了。 在视图函数中，我们已经讨论了通过模板系统把业务逻辑和表现逻辑分隔开的重要性。 在数据库层中，我们对数据访问逻辑也应用了同样的理念。

把数据存取逻辑、业务逻辑和表现逻辑组合在一起的概念有时被称为软件架构的 Model-View-Controller (MVC)模式。 在这个模式中， Model 代表数据存取层，View 代表的是系统中选择显示什么和怎么显示的部分，Controller 指的是系统中根据用户输入并视需要访问模型，以决定使用哪个视图的那部分。

以下是 Django 中 M、V 和 C 各自的含义：

* M ，数据存取部分，由django数据库层处理，本章要讲述的内容。

* V ，选择显示哪些数据要显示以及怎样显示的部分，由视图和模板处理。2

* C ，根据用户输入委派视图的部分，由 Django 框架根据 URLconf 设置，对给定 URL 调用适当的 Python 函数。

由于 C 由框架自行处理，而 Django 里更关注的是模型（Model）、模板(Template)和视图（Views），Django 也被称为 MTV 框架 。在 MTV 开发模式中：

* M 代表模型（Model），即数据存取层。 该层处理与数据相关的所有事务： 如何存取、如何验证有效性、包含哪些行为以及数据之间的关系等。

* T 代表模板(Template)，即表现层。 该层处理与表现相关的决定： 如何在页面或其他类型文档中进行显示。

* V 代表视图（View），即业务逻辑层。 该层包含存取模型及调取恰当模板的相关逻辑。 你可以把它看作模型与模板之间的桥梁。

--------------------------------------------------------------------

##数据库配置##

数据库配置也是在Django的配置文件里，缺省 是 settings.py 。

    DATABASE_ENGINE = ''
    DATABASE_NAME = ''
    DATABASE_USER = ''
    DATABASE_PASSWORD = ''
    DATABASE_HOST = ''
    DATABASE_PORT = ''

第二章我们已经创建了 project , 那么 project 和 app 之间到底有什么不同呢？它们的区别就是一个是配置另一个是 代码：

* 一个project包含很多个Django app以及对它们的配置。

* 技术上，project的作用是提供配置文件，比方说哪里定义数据库连接信息, 安装的app列表， TEMPLATE_DIRS ，等等。

* 一个app是一套Django功能的集合，通常包括模型和视图，按Python的包结构的方式存在。

* 例如，Django本身内建有一些app，例如注释系统和自动管理界面。 app的一个关键点是它们是很容易移植到其他project和被多个project复用。


不错，你可以不用创建app，这一点应经被我们之前编写的视图函数的例子证明了 。 在那些例子中，我们只是简单的创建了一个称为views.py的文件，编写了一些函数并在URLconf中设置了各个函数的映射。 这些情况都不需要使用apps。7

但是，系统对app有一个约定： 如果你使用了Django的数据库层（模型），你 必须创建一个Django app。 模型必须存放在apps中。 因此，为了开始建造 我们的模型，我们必须创建一个新的app。

在 `` mysite`` 项目文件下输入下面的命令来创建 `` books`` app：

    python manage.py startapp books

这个命令并没有输出什么，它只在 mysite 的目录里创建了一个 books 目录。 让我们来看看这个目录的内容：

    books/
        __init__.py
        models.py
        tests.py
        views.py

--------------------------------------------------------------------------------
第一步是用Python代码来描述它们。 打开由`` startapp`` 命令创建的models.py 并输入下面的内容：

    from django.db import models

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField()

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

首先要注意的事是每个数据模型都是 django.db.models.Model 的子类。它的父类 Model 包含了所有必要的和数据库交互的方法，并提供了一个简洁漂亮的定义数据库字段的语法。

每个模型相当于单个数据库表，每个属性也是这个表中的一个字段。 属性名就是字段名，它的类型（例如 CharField ）相当于数据库的字段类型 （例如 varchar ）。例如， Publisher 模块等同于下面这张表（用PostgreSQL的 CREATE TABLE 语法描述）：

    CREATE TABLE "books_publisher" (
        "id" serial NOT NULL PRIMARY KEY,
        "name" varchar(30) NOT NULL,
        "address" varchar(50) NOT NULL,
        "city" varchar(60) NOT NULL,
        "state_province" varchar(30) NOT NULL,
        "country" varchar(50) NOT NULL,
        "website" varchar(200) NOT NULL
    );

**“每个数据库表对应一个类”这条规则的例外情况是多对多关系。 在我们的范例模型中， Book 有一个 多对多字段 叫做 authors 。 该字段表明一本书籍有一个或多个作者，但 Book 数据库表却并没有 authors 字段。 相反，Django创建了一个额外的表（多对多连接表）来处理书籍和作者之间的映射关系。**

最后需要注意的是，我们并没有显式地为这些模型定义任何主键。 除非你单独指明，否则Django会自动为每个模型生成一个自增长的整数主键字段每个Django模型都要求有单独的主键-id

###模型安装###

完成这些代码之后，现在让我们来在数据库中创建这些表。 要完成该项工作，第一步是在 Django 项目中 激活 这些模型。 将 books app 添加到配置文件的已安装应用列表中即可完成此步骤。

再次编辑 settings.py 文件， 找到 INSTALLED_APPS 设置。 INSTALLED_APPS 告诉 Django 项目哪些 app 处于激活状态。 缺省情况下如下所示：

    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
    )
**再次注意app和project的区别，这里installed apps 默认的是django自带的apps，而我们自己创建的apps要手动的添加进来。**

'mysite.books'指示我们正在编写的books app。 INSTALLED_APPS 中的每个app都使用 Python的路径描述，包的路径，用小数点“.”间隔。

现在我们可以创建数据库表了。 首先，用下面的命令验证模型的有效性：

    python manage.py validate

一旦你觉得你的模型可能有问题，运行 python manage.py validate 。 它可以帮助你捕获一些常见的模型定义错误。

模型确认没问题了，运行下面的命令来生成 CREATE TABLE 语句

    python manage.py sqlall books

sqlall 命令并没有在数据库中真正创建数据表，只是把SQL语句段打印出来，这样你可以看到Django究竟会做些什么.可以把那些SQL语句复制到你的数据库客户端执行，或者通过Unix管道直接进行操作（例如，`` python manager.py sqlall books | psql mydb`` ）。不过，Django提供了一种更为简易的提交SQL语句至数据库的方法： `` syncdb`` 命令。

    python manage.py syncdb

syncdb 命令是同步你的模型到数据库的一个简单方法。 它会根据 INSTALLED_APPS 里设置的app来检查数据库， 如果表不存在，它就会创建它。 需要注意的是， syncdb 并 不能将模型的修改或删除同步到数据库；如果你修改或删除了一个模型，并想把它提交到数据库，syncdb并不会做出任何处理。

如果你再次运行 python manage.py syncdb ，什么也没发生，因为你没有添加新的模型或者 添加新的app。因此，运行python manage.py syncdb总是安全的，因为它不会重复执行SQL语句。


----------------------------------------------------------
###插入和更新数据###

    >>> p = Publisher(name='Apress',
    ...         address='2855 Telegraph Ave.',
    ...         city='Berkeley',
    ...         state_province='CA',
    ...         country='U.S.A.',
    ...         website='http://www.apress.com/')

    >>> p.save()//save()是必须的

前面执行的 save() 相当于下面的SQL语句：

    UPDATE books_publisher SET
        name = 'Apress Publishing',
        address = '2855 Telegraph Ave.',
        city = 'Berkeley',
        state_province = 'CA',
        country = 'U.S.A.',
        website = 'http://www.apress.com'
    WHERE id = 52;

**注意，并不是只更新修改过的那个字段，所有的字段都会被更新。 这个操作有可能引起竞态条件，这取决于你的应用程序**

###选择对象###

* 让我们来仔细看看 Publisher.objects.all() 这行的每个部分：
* 首先，我们有一个已定义的模型 Publisher 。没什么好奇怪的： 你想要查找数据， 你就用模型来获得数据。
* 然后，是objects属性。 它被称为管理器，我们将在第10章中详细讨论它。 目前，我们只需了解管理器管理着所有针对数据包含、还有最重要的数据查询的表格级操作。
* 所有的模型都自动拥有一个 objects 管理器；你可以在想要查找数据时使用它。
* 最后，还有 all() 方法。这个方法返回返回数据库中所有的记录。 尽管这个对象 看起来 象一个列表（list），它实际是一个 QuerySet 对象， 这个对象是数据库中一些记录的集合。

###数据过滤###

我们很少会一次性从数据库中取出所有的数据；通常都只针对一部分数据进行操作。 在Django API中，我们可以使用`` filter()`` 方法对数据进行过滤：

    >>> Publisher.objects.filter(name='Apress')
    [<Publisher: Apress>]

你可以传递多个参数到 filter() 来缩小选取范围：

    >>> Publisher.objects.filter(country="U.S.A.", state_province="CA")
    [<Publisher: Apress>]

注意，SQL缺省的 = 操作符是精确匹配的， 其他类型的查找也可以使用：

    >>> Publisher.objects.filter(name__contains="press")
    [<Publisher: Apress>]
在 name 和 contains 之间有双下划线。和Python一样，Django也使用双下划线来表明会进行一些魔术般的操作。这里，contains部分会被Django翻译成LIKE语句：

    SELECT id, name, address, city, state_province, country, website
    FROM books_publisher
    WHERE name LIKE '%press%';


其他的一些查找类型有：icontains(大小写无关的LIKE),startswith和endswith, 还有range(SQLBETWEEN查询）。

###获取单个对象###

上面的例子中 `` filter()`` 函数返回一个记录集，这个记录集是一个列表。 相对列表来说，有些时候我们更需要获取单个的对象， `` get()`` 方法就是在此时使用的：

    >>> Publisher.objects.get(name="Apress")
    <Publisher: Apress>
这样，就返回了单个对象，而不是列表（更准确的说，QuerySet)。 所以，如果结果是多个对象，会导致抛出异常：2

    >>> Publisher.objects.get(country="U.S.A.")
    Traceback (most recent call last):
    ...
    MultipleObjectsReturned: get() returned more than one Publisher --


        it returned 2! Lookup parameters were {'country': 'U.S.A.'}

如果查询没有返回结果也会抛出异常：

    >>> Publisher.objects.get(name="Penguin")
    Traceback (most recent call last):
        ...
DoesNotExist: Publisher matching query does not exist.
这个 DoesNotExist 异常 是 Publisher 这个 model 类的一个属性，即 Publisher.DoesNotExist。在你的应用中，你可以捕获并处理这个异常，像这样：

    try:
        p = Publisher.objects.get(name='Apress')
    except Publisher.DoesNotExist:
        print "Apress isn't in the database yet."
    else:
        print "Apress is in the database."

###数据排序###

使用 order_by() 这个方法就可以搞定了。

    >>> Publisher.objects.order_by("name")
    [<Publisher: Apress>, <Publisher: O'Reilly>]

如果需要以多个字段为标准进行排序（第二个字段会在第一个字段的值相同的情况下被使用到），使用多个参数就可以了，如下：

    >>> Publisher.objects.order_by("state_province", "address")
     [<Publisher: Apress>, <Publisher: O'Reilly>]
我们还可以指定逆向排序，在前面加一个减号 - 前缀：

    >>> Publisher.objects.order_by("-name")
    [<Publisher: O'Reilly>, <Publisher: Apress>]
  

Django让你可以指定模型的缺省排序方式：

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

    def __unicode__(self):
        return self.name

    **class Meta:**
        **ordering = ['name']**


现在，让我们来接触一个新的概念。 class Meta，内嵌于 Publisher 这个类的定义中（如果 class Publisher 是顶格的，那么 class Meta 在它之下要缩进4个空格－－按 Python 的传统 ）。你可以在任意一个 模型 类中使用 Meta 类，来设置一些与特定模型相关的选项。 在 附录B 中有 Meta 中所有可选项的完整参考，现在，我们关注 ordering 这个选项就够了。 如果你设置了这个选项，那么除非你检索时特意额外地使用了 order_by()，否则，当你使用 Django 的数据库 API 去检索时，Publisher对象的相关返回值默认地都会按 name 字段排序。


###限制返回的数据数量###

    >>> Publisher.objects.order_by('name')[0]//可以使用切片
    <Publisher: Apress>

注意，不支持Python的负索引(negative slicing)：

    >>> Publisher.objects.order_by('name')[-1]
    Traceback (most recent call last):
      ...
AssertionError: Negative indexing is not supported.
虽然不支持负索引，但是我们可以使用其他的方法。 比如，稍微修改 order_by() 语句来实现：

    >>> Publisher.objects.order_by('-name')[0]

###更新多个对象###

在“插入和更新数据”小节中，我们有提到模型的save()方法，这个方法会更新一行里的所有列。 而某些情况下，我们只需要更新行里的某几列。

更改某一指定的列，我们可以调用结果集（QuerySet）对象的update()方法： 示例如下：

    >>> Publisher.objects.filter(id=52).update(name='Apress Publishing')
与之等同的SQL语句变得更高效，并且不会引起竞态条件。

    UPDATE books_publisher
    SET name = 'Apress Publishing'
    WHERE id = 52;


update()方法对于任何结果集（QuerySet）均有效，这意味着你可以同时更新多条记录。 以下示例演示如何将所有Publisher的country字段值由’U.S.A’更改为’USA’：

    >>> Publisher.objects.all().update(country='USA')
    
update()方法会返回一个整型数值，表示受影响的记录条数。 在上面的例子中，这个值是2。



###删除对象###

删除数据库中的对象只需调用该对象的delete()方法即可：

    >>> p = Publisher.objects.get(name="O'Reilly")
    >>> p.delete()
    >>> Publisher.objects.all()
    [<Publisher: Apress Publishing>]
同样我们可以在结果集上调用delete()方法同时删除多条记录。这一点与我们上一小节提到的update()方法相似：

    >>> Publisher.objects.filter(country='USA').delete()
    >>> Publisher.objects.all().delete()
    >>> Publisher.objects.all()

删除数据时要谨慎！ 为了预防误删除掉某一个表内的所有数据，Django要求在删除表内所有数据时显示使用all()。 比如，下面的操作将会出错：1

    >>> Publisher.objects.delete()
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
    AttributeError: 'Manager' object has no attribute 'delete'
而一旦使用all()方法，所有数据将会被删除：

    >>> Publisher.objects.all().delete()
如果只需要删除部分的数据，就不需要调用all()方法。再看一下之前的例子：

    >>> Publisher.objects.filter(country='USA').delete()
