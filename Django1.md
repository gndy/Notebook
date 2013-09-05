
    # models.py (the database tables)

    from django.db import models

    class Book(models.Model):
        name = models.CharField(max_length=50)
        pub_date = models.DateField()


    # views.py (the business logic)

    from django.shortcuts import render_to_response
    from models import Book

    def latest_books(request):
        book_list = Book.objects.order_by('-pub_date')[:10]
        return render_to_response('latest_books.html', {'book_list': book_list})


    #urls.py (the URL configuration)

    from django.conf.urls.defaults import *
    import views

    urlpatterns = patterns('',
        (r'^latest/$', views.latest_books),
    )


    #latest_books.html (the template)

    <html><head><title>Books</title></head>
    <body>
    <h1>Books</h1>
    <ul>
    {% for book in book_list %}
    <li>{{ book.name }}</li>
    {% endfor %}
    </ul>
    </body></html>
 

+ models.py 文件主要用一个 Python 类来描述数据表。 称为 模型(model) 。 运用这个类，你可以通过简单的 Python 的代码来创建、检索、更新、删除 数据库中的记录而无需写一条又一条的SQL语句。
+ views.py文件包含了页面的业务逻辑。 latest_books()函数叫做视图
+ urls.py 指出了什么样的 URL 调用什么的视图。 在这个例子中 /latest/ URL 将会调用 latest_books() 这个函数。 换句话说，如果你的域名是example.com，任何人浏览网址http://example.com/latest/将会调用latest_books()这个函数。
+ latest_books.html 是 html 模板，它描述了这个页面的设计是如何的。 使用带基本逻辑声明的模板语言，如{% for book in book_list %}

----------------------------------------------------------------------
 `startprojet` 命令创建一个目录，包含4个文件：
    mysite/
        __init__.py
        manage.py
        settings.py
        urls.py

+ __init__.py ：让 Python 把该目录当成一个开发包 (即一组模块)所需的文件。 这是一个空文件，一般你不需要修改它。
+ manage.py ：一种命令行工具，允许你以多种方式与该 Django 项目进行交互。 键入python manage.py help，看一下它能做什么。 你应当不需要编辑这个文件；在这个目录下生成它纯是为了方便。
+ settings.py ：该 Django 项目的设置或配置。 查看并理解这个文件中可用的设置类型及其默认值。
+ urls.py：Django项目的URL设置。 可视其为你的django网站的目录。 目前，它是空的。
`URLcon(urls.py)` 就像是 Django 所支撑网站的目录。 它的本质是 URL 模式以及要为该 URL 模式调用的视图函数之间的映射表。 你就是以这种方式告诉 Django，对于这个 URL 调用这段代码，对于那个 URL 调用那段代码。 例如，当用户访问/foo/时，调用视图函数foo_view()，这个视图函数存在于Python模块文件view.py中。


=======================================================================
### setting文件

所有均开始于setting文件。当你运行python manage.py runserver，脚本将在于manage.py同一个目录下查找名为setting.py的文件。这个文件包含了所有有关这个Django项目的配置信息，均大写： TEMPLATE_DIRS , DATABASE_NAME , 等. 最重要的设置时*ROOT_URLCONF*，它将作为URLconf告诉Django在这个站点中那些Python的模块将被用到

总结一下：

1.进来的请求转入/hello/.

2.Django通过在ROOT_URLCONF配置来决定根URLconf.

3.Django在URLconf中的所有URL模式中，查找第一个匹配/hello/的条目。

4.如果找到匹配，将调用相应的视图函数

5.视图函数返回一个HttpResponse

6.Django转换HttpResponse为一个适合的HTTP response， 以Web page显示出来


------------------------------------------------------------------------
###在你视图的任何位置，临时插入一个 assert False 来触发出错页。 然后，你就可以看到局部变量和程序语句了。###

##直接将HTML硬编码到你的视图里却并不是一个好主意##

+ 对页面设计进行的任何改变都必须对 Python 代码进行相应的修改。 站点设计的修改往往比底层 Python 代码的修改要频繁得多，因此如果可以在不进行 Python 代码修改的情况下变更设计，那将会方便得多。

+ Python 代码编写和 HTML 设计是两项不同的工作，大多数专业的网站开发环境都将他们分配给不同的人员（甚至不同部门）来完成。 设计者和HTML/CSS的编码人员不应该被要求去编辑Python的代码来完成他们的工作。

+ 程序员编写 Python代码和设计人员制作模板两项工作同时进行的效率是最高的，远胜于让一个人等待另一个人完成对某个既包含 Python又包含 HTML 的文件的编辑工作。

----------------------------------------------------------------------------------------

    <html>

    <head><title>Ordering notice</title></head>

    <body>

    <h1>Ordering notice</h1>

    <p>Dear {{ person_name }},</p>

    <p>Thanks for placing an order from {{ company }}. It's scheduled to
    ship on {{ ship_date|date:"F j, Y" }}.</p>

    <p>Here are the items you've ordered:</p>

    <ul>
    {% for item in item_list %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>

    {% if ordered_warranty %}
        <p>Your warranty information will be included in the packaging.</p>
    {% else %}
        <p>You didn't order a warranty, so you're on your own when
        the products inevitably stop working.</p>
    {% endif %}

    <p>Sincerely,<br />{{ company }}</p>

    </body>
    </html>


      
    用两个大括号括起来的文字（例如 {{ person_name }} ）称为 变量(variable) 。这意味着在此处插入指定变量的值。 如何指定变量的值呢？ 稍后就会说明。4

    被大括号和百分号包围的文本(例如 {% if ordered_warranty %} )是 模板标签(template tag) 。标签(tag)定义比较明确，即： 仅通知模板系统完成某些工作的标签。

    这个例子中的模板包含一个for标签（ {% for item in item_list %} ）和一个if 标签（{% if ordered_warranty %} ）

    for标签类似Python的for语句，可让你循环访问序列里的每一个项目。 if 标签，正如你所料，是用来执行逻辑判断的。 在这里，tag标签检查ordered_warranty值是否为True。如果是，模板系统将显示{% if ordered_warranty %}和{% else %}之间的内容；否则将显示{% else %}和{% endif %}之间的内容。{% else %}是可选的。

    最后，这个模板的第二段中有一个关于filter过滤器的例子，它是一种最便捷的转换变量输出格式的方式。 如这个例子中的{{ship_date|date:”F j, Y” }}，我们将变量ship_date传递给date过滤器，同时指定参数”F j,Y”。date过滤器根据参数进行格式输出。 过滤器是用管道符(|)来调用的，具体可以参见Unix管道符。


该模板是一段添加了些许变量和模板标签的基础 HTML 。
    用两个大括号括起来的文字（例如 {{ person_name }} ）称为 变量(variable) 。这意味着在此处插入指定变量的值。 如何指定变量的值呢？ 稍后就会说明。4

    被大括号和百分号包围的文本(例如 {% if ordered_warranty %} )是 模板标签(template tag) 。标签(tag)定义比较明确，即： 仅通知模板系统完成某些工作的标签。

    这个例子中的模板包含一个for标签（ {% for item in item_list %} ）和一个if 标签（{% if ordered_warranty %} ）

    for标签类似Python的for语句，可让你循环访问序列里的每一个项目。 if 标签，正如你所料，是用来执行逻辑判断的。 在这里，tag标签检查ordered_warranty值是否为True。如果是，模板系统将显示{% if ordered_warranty %}和{% else %}之间的内容；否则将显示{% else %}和{% endif %}之间的内容。{% else %}是可选的。

    最后，这个模板的第二段中有一个关于filter过滤器的例子，它是一种最便捷的转换变量输出格式的方式。 如这个例子中的{{ship_date|date:”F j, Y” }}，我们将变量ship_date传递给date过滤器，同时指定参数”F j,Y”。date过滤器根据参数进行格式输出。 过滤器是用管道符(|)来调用的，具体可以参见Unix管道符。


Django 模板含有很多内置的tags和filters,我们将陆续进行学习. 附录F列出了很多的tags和filters的列表.


=====================================================
###在Python代码中使用Django模板的最基本方式如下：###

* 可以用原始的模板代码字符串创建一个 Template 对象， Django同样支持用指定模板文件路径的方式来创建 Template 对象;
* 调用模板对象的render方法，并且传入一套变量context。它将返回一个基于模板的展现字符串，模板中的变量和标签会被context值替换。

    from django import template
    t = template.Template('My name is {{ name }}.')
     c = template.Context({'name': 'Adrian'})
     print t.render(c)
    My name is Adrian.
     c = template.Context({'name': 'Fred'})
     print t.render(c)
    My name is Fred.



###模板渲染###

一旦你创建一个 Template 对象，你可以用 context 来传递数据给它。 一个context是一系列变量和它们值的集合。
    
ontext在Django里表现为 Context 类，在 django.template 模块里。 她的构造函数带有一个可选的参数： 一个字典映射变量和它们的值。 调用 Template 对象 的 render() 方法并传递context来填充模板

*我们必须指出的一点是，t.render(c)返回的值是一个Unicode对象，不是普通的Python字符串。*


###无论何时我们都可以像这样使用同一模板源渲染多个context，只进行 一次模板创建然后多次调用render()方法渲染会更为高效：###

    # Bad
    for name in ('John', 'Julie', 'Pat'):
        t = Template('Hello, {{ name }}')
        print t.render(Context({'name': name}))

    # Good
    t = Template('Hello, {{ name }}')
    for name in ('John', 'Julie', 'Pat'):
        print t.render(Context({'name': name}))


###深度变量的查找###

可以在context中调用对象的属性和方法，但是只能是没有参数的方法。
       
       
------------------------------------------------------------------
##在视图中使用模板##


    from django.template import Template, Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = Template("<html><body>It is now {{ current_date }}.</body></html>")
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

 模板仍然嵌入在Python代码里，并未真正的实现数据与表现的分离。 让我们将模板置于一个 单独的文件 中，并且让视图加载该文件来解决此问题。

 假设文件保存在 /home/djangouser/templates/mytemplate.html 中的话，代码就会像下面这样:
 from django.template import Template, Context
from django.http import HttpResponse
import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        # Simple way of using templates from the filesystem.
        # This is BAD because it doesn't account for missing files!
        fp = open('/home/djangouser/templates/mytemplate.html')
        t = Template(fp.read())
        fp.close()
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

该方法还算不上简洁：

*它没有对文件丢失的情况做出处理。 如果文件 mytemplate.html 不存在或者不可读， open() 函数调用将会引发 IOError 异常。
*这里对模板文件的位置进行了硬编码。 如果你在每个视图函数都用该技术，就要不断复制这些模板的位置。 更不用说还要带来大量的输入工作！
*它包含了大量令人生厌的重复代码。 与其在每次加载模板时都调用 open() 、 fp.read() 和 fp.close() ，还不如做出更佳选择。

###模板加载###
为了减少模板加载调用过程及模板本身的冗余代码，Django 提供了一种使用方便且功能强大的 API ，用于从磁盘中加载模板，

如果你是一步步跟随我们学习过来的，马上打开你的settings.py配置文件，找到TEMPLATE_DIRS这项设置吧。 它的默认设置是一个空元组（tuple），加上一些自动生成的注释。

    TEMPLATE_DIRS = (
        # Put strings here, like "/home/html/django_templates" or "C:/www/django/templates".
        # Always use forward slashes, even on Windows.
        # Don't forget to use absolute paths, not relative paths.
    )

选择一个目录用于存放模板并将其添加到 TEMPLATE_DIRS 中：

    TEMPLATE_DIRS = (
        '/home/django/mysite/templates',
    )

最省事的方式是使用绝对路径（即从文件系统根目录开始的目录路径）。 如果想要更灵活一点并减少一些负面干扰，可利用 Django 配置文件就是 Python 代码这一点来动态构建 TEMPLATE_DIRS 的内容，如： 例如：

    import os.path

    TEMPLATE_DIRS = (
        os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/'),
    )

这个例子使用了神奇的 Python 内部变量 __file__ ，该变量被自动设置为代码所在的 Python 模块文件名。 `` os.path.dirname(__file__)`` 将会获取自身所在的文件，即settings.py 所在的目录，然后由os.path.join 这个方法将这目录与 templates 进行连接。如果在windows下，它会智能地选择正确的后向斜杠”“进行连接，而不是前向斜杠”/”。

    from django.template.loader import get_template
    from django.template import Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = get_template('current_datetime.html')
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

此范例中，我们使用了函数 django.template.loader.get_template() ，而不是手动从文件系统加载模板。 该 get_template() 函数以模板名称为参数，在文件系统中找出模块的位置，打开文件并返回一个编译好的 Template 对象。

###render_to_response()###

Django为此提供了一个捷径，让你一次性地载入某个模板文件，渲染它，然后将此作为 HttpResponse返回。

下面就是使用 render_to_response() 重新编写过的 current_datetime 范例。

    from django.shortcuts import render_to_response
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
          return render_to_response('current_datetime.html', {'current_date': now})

###get_template()中使用子目录###

只需在调用 get_template() 时，把子目录名和一条斜杠添加到模板名称之前，如：

    t = get_template('dateapp/current_datetime.html')

---------------------------------------------------------
###include 模板标签###

在讲解了模板加载机制之后，我们再介绍一个利用该机制的内建模板标签： {% include %} 。该标签允许在（模板中）包含其它的模板的内容。 标签的参数是所要包含的模板名称，可以是一个变量，也可以是用单/双引号硬编码的字符串。 每当在多个模板中出现相同的代码时，就应该考虑是否要使用 {% include %} 来减少重复。

##模板继承##

但在实际应用中，你将用 Django 模板系统来创建整个 HTML 页面。 这就带来一个常见的 Web 开发问题： 在整个网站中，如何减少共用页面区域（比如站点导航）所引起的重复和冗余代码？
解决该问题的传统做法是使用 服务器端的 includes ，你可以在 HTML 页面中使用该指令将一个网页嵌入到另一个中。 事实上， Django 通过刚才讲述的 {% include %} 支持了这种方法。 但是用 Django 解决此类问题的首选方法是使用更加优雅的策略—— 模板继承 。

模板继承就是先构造一个基础框架模板，而后在其子模板中对它所包含站点公用部分和定义块进行重载。

第一步是定义 基础模板 ， 该框架之后将由 子模板 所继承。

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
    <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
    <h1>My helpful timestamp site</h1>
    {% block content %}{% endblock %}
    {% block footer %}
    <hr>
    <p>Thanks for visiting my site.</p>
    {% endblock %}
    </body>
    </html>

这个叫做 base.html 的模板定义了一个简单的 HTML 框架文档，我们将在本站点的所有页面中使用。 子模板的作用就是重载、添加或保留那些块的内容。

我们使用一个以前已经见过的模板标签： {% block %} 。 所有的 {% block %} 标签告诉模板引擎，子模板可以重载这些部分。 每个{% block %}标签所要做的是告诉模板引擎，该模板下的这一块内容将有可能被子模板覆盖。

现在我们已经有了一个基本模板，我们可以修改 current_datetime.html 模板来 使用它：

    {% extends "base.html" %}

    {% block title %}The current time{% endblock %}

    {% block content %}
    <p>It is now {{ current_date }}.</p>
    {% endblock %}

每个模板只包含对自己而言 独一无二 的代码。 无需多余的部分。 如果想进行站点级的设计修改，仅需修改 base.html ，所有其它模板会立即反映出所作修改。

以下是其工作方式。 在加载 current_datetime.html 模板时，模板引擎发现了 {% extends %} 标签， 注意到该模板是一个子模板。 模板引擎立即装载其父模板，即本例中的 base.html 。

此时，模板引擎注意到 base.html 中的三个 {% block %} 标签，并用子模板的内容替换这些 block 。因此，引擎将会使用我们在 { block title %} 中定义的标题，对 {% block content %} 也是如此。 所以，网页标题一块将由 {% block title %}替换，同样地，网页的内容一块将由 {% block content %}替换。9

注意由于子模板并没有定义 footer 块，模板系统将使用在父模板中定义的值。 父模板 {% block %} 标签中的内容总是被当作一条退路。

继承并不会影响到模板的上下文。 换句话说，任何处在继承树上的模板都可以访问到你传到模板中的每一个模板变量。

你可以根据需要使用任意多的继承次数。 使用继承的一种常见方式是下面的三层法：

创建 base.html 模板，在其中定义站点的主要外观感受。 这些都是不常修改甚至从不修改的部分。

为网站的每个区域创建 base_SECTION.html 模板(例如, base_photos.html 和 base_forum.html )。这些模板对 base.html 进行拓展，并包含区域特定的风格与设计。1

为每种类型的页面创建独立的模板，例如论坛页面或者图片库。 这些模板拓展相应的区域模板。

这个方法可最大限度地重用代码，并使得向公共区域（如区域级的导航）添加内容成为一件轻松的工作。
