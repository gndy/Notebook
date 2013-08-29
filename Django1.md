
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


      
