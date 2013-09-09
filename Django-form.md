##第7章 表单##

###从Request对象中获取数据###

####URL相关信息####

HttpRequest对象包含当前请求URL的一些信息：

| 属性/方法 | 说明                       |举例        |
|-----------|----------------------------|------------|
|request.path|除域名以外的请求路径    |'/hello/'   |
|request.get_host()|主机名            |127.0.0.1   |
|request.get_full_path()|请求路径可能包括查询参数|"/hello/?print=true"|
|request.is_secure()|是否通过https访问，返回True or False|True or False|


在view函数里，要始终用这个属性或方法来得到URL，而不要手动输入。 这会使得代码更加灵活，以便在其它地方重用。

    BAD!
    def current_url_view_bad(request):
        return HttpResponse("Welcome to the page at /current/")

    GOOD
    def current_url_view_good(request):
        return HttpResponse("Welcome to the page at %s" % request.path)


###有关request的其它信息###

request.META 是一个Python字典，包含了所有本次HTTP请求的Header信息，比如用户IP地址和用户Agent（通常是浏览器的名称和版本号）。 注意，Header信息的完整列表取决于用户所发送的Header信息和服务器端设置的Header信息。 这个字典中几个常见的键值有：


* HTTP_REFERER，进站前链接网页，如果有的话。 （请注意，它是REFERRER的笔误。）3

* HTTP_USER_AGENT，用户浏览器的user-agent字符串，如果有的话。 例如： "Mozilla/5.0 (X11; U; Linux i686; fr-FR; rv:1.8.1.17) Gecko/20080829 Firefox/2.0.0.17" .

* REMOTE_ADDR 客户端IP，如："12.345.67.89" 。(如果申请是经过代理服务器的话，那么它可能是以逗号分割的多个IP地址，如："12.345.67.89,23.456.78.90" 。)


注意，因为 request.META 是一个普通的Python字典，因此当你试图访问一个不存在的键时，会触发一个KeyError异常。 （HTTP header信息是由用户的浏览器所提交的、不应该给予信任的“额外”数据，因此你总是应该好好设计你的应用以便当一个特定的Header数据不存在时，给出一个优雅的回应。）你应该用 try/except 语句，或者用Python字典的 get() 方法来处理这些“可能不存在的键”：

    BAD!
    def ua_display_bad(request):
        ua = request.META['HTTP_USER_AGENT']  # Might raise KeyError!
        return HttpResponse("Your browser is %s" % ua)

    GOOD (VERSION 1)
    def ua_display_good1(request):
        try:
            ua = request.META['HTTP_USER_AGENT']
        except KeyError:
            ua = 'unknown'
        return HttpResponse("Your browser is %s" % ua)

    GOOD (VERSION 2)
    def ua_display_good2(request):
        ua = request.META.get('HTTP_USER_AGENT', 'unknown')
        return HttpResponse("Your browser is %s" % ua)


###提交的数据信息###

除了基本的元数据，HttpRequest对象还有两个属性包含了用户所提交的信息： request.GET 和 request.POST。二者都是类字典对象，你可以通过它们来访问GET和POST数据。

POST数据是来自HTML中的〈form〉标签提交的，而GET数据可能来自〈form〉提交也可能是URL中的查询字符串(the query string)。

-----------------------------------------------------------------------

###一个简单的表单处理示例###

表单开发分为两个部分： 前端HTML页面用户接口和后台view函数对所提交数据的处理过程。

现在我们来建立个view来显示一个搜索表单：

    from django.shortcuts import render_to_response

    def search_form(request):
        return render_to_response('search_form.html')

search_form.html 模板

    <html>
    <head>
        <title>Search</title>
    </head>
    <body>
        <form action="/search/" method="get">
            <input type="text" name="q">
            <input type="submit" value="Search">
        </form>
    </body>
    </html>

而 urls.py 中的 URLpattern 可能是这样的：

    from mysite.books import views

    urlpatterns = patterns('',
        # ...
        (r'^search-form/$', views.search_form),
        # ...
    )

当你通过这个form提交数据时，你会得到一个Django 404错误。 这个Form指向的URL /search/ 还没有被实现。 让我们添加第二个视图函数并设置URL：

# urls.py

    urlpatterns = patterns('',
        # ...
        (r'^search-form/$', views.search_form),
        (r'^search/$', views.search),
        # ...
    )

# views.py

    def search(request):
        if 'q' in request.GET:
            message = 'You searched for: %r' % request.GET['q']
        else:
            message = 'You submitted an empty form.'
        return HttpResponse(message)

----------------------------------------------------------------
