
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
