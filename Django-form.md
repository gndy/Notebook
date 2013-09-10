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

urls.py

    urlpatterns = patterns('',
        # ...
        (r'^search-form/$', views.search_form),
        (r'^search/$', views.search),
        # ...
    )

views.py

    def search(request):
        if 'q' in request.GET:
            message = 'You searched for: %r' % request.GET['q']
        else:
            message = 'You submitted an empty form.'
        return HttpResponse(message)
--------------------------------------------------------------------------

###查询字符串参数###

    from django.http import HttpResponse
    from django.shortcuts import render_to_response
    from mysite.books.models import Book

    def search(request):
        if 'q' in request.GET and request.GET['q']:
            q = request.GET['q']
            books = Book.objects.filter(title__icontains=q)
            return render_to_response('search_results.html',
                {'books': books, 'query': q})
        else:
            return HttpResponse('Please submit a search term.')

让我们来分析一下上面的代码：

>>除了检查q是否存在于request.GET之外，我们还检查来reuqest.GET[‘q’]的值是否为空。

>>我们使用Book.objects.filter(title__icontains=q)获取数据库中标题包含q的书籍。 icontains是一个查询关键字（参看第五章和附录B）。这个语句可以理解为获取标题里包含q的书籍，不区分大小写。

>>这是实现书籍查询的一个很简单的方法。 我们不推荐在一个包含大量产品的数据库中使用icontains查询，因为那会很慢。

最后，我们给模板传递来books，一个包含Book对象的列表。 查询结果的显示模板search_results.html如下所示：

    <p>You searched for: <strong>{{ query }}</strong></p>

    {% if books %}
        <p>Found {{ books|length }} book{{ books|pluralize }}.</p>
        <ul>
            {% for book in books %}
            <li>{{ book.title }}</li>
            {% endfor %}
        </ul>
    {% else %}
        <p>No books matched your search criteria.</p>
    {% endif %}

注意这里pluralize的使用，这个过滤器在适当的时候会输出s（书籍为复数时).


###改进表单###

首先，search()视图对于空字符串的处理相当薄弱——仅显示一条”Please submit a search term.”的提示信息。 若用户要重新填写表单必须自行点击“后退”按钮， 这种做法既糟糕又不专业。如果在现实的案例中，我们这样子编写，那么Django的优势将荡然无存。

在检测到空字符串时更好的解决方法是重新显示表单，并在表单上面给出错误提示以便用户立刻重新填写。 最简单的实现方法既是添加else分句重新显示表单，代码如下：

    from django.http import HttpResponse
    from django.shortcuts import render_to_response
    from mysite.books.models import Book

    def search_form(request):
        return render_to_response('search_form.html')

    def search(request):
        if 'q' in request.GET and request.GET['q']:
            q = request.GET['q']
            books = Book.objects.filter(title__icontains=q)
            return render_to_response('search_results.html',
                {'books': books, 'query': q})
        else:
            return render_to_response('search_form.html', {'error': True})


这段代码里，我们改进来search()视图：在字符串为空时重新显示search_form.html。 并且给这个模板传递了一个变量error，记录着错误提示信息。 现在我们编辑一下search_form.html，检测变量error：

    <html>
    <head>
        <title>Search</title>
    </head>
    <body>
        **{% if error %}**
            **<p style="color: red;">Please submit a search term.</p>**
        **{% endif %}**
        <form action="/search/" method="get">
            <input type="text" name="q">
            <input type="submit" value="Search">
        </form>
    </body>
    </html>
我们修改了search_form()视图所使用的模板，因为search_form()视图没有传递error变量，所以在条用search_form视图时不会显示错误信息。

-----------------------------------------------------

###关于JavaScript验证###

可以使用Javascript在客户端浏览器里对数据进行验证，这些知识已超出本书范围。 要注意： 即使在客户端已经做了验证，但是服务器端仍必须再验证一次。 因为有些用户会将JavaScript关闭掉，并且还有一些怀有恶意的用户会尝试提交非法的数据来探测是否有可以攻击的机会。5

除了在服务器端对用户提交的数据进行验证（例如在视图里验证），我们没有其他办法。 JavaScript验证可以看作是额外的功能，但不能作为唯一的验证功能。

###编写Contact表单###

虽然我们一直使用书籍搜索的示例表单，并将起改进的很完美，但是这还是相当的简陋： 只包含一个字段，q。这简单的例子，我们不需要使用Django表单库来处理。 但是复杂一点的表单就需要多方面的处理，我们现在来一下一个较为复杂的例子： 站点联系表单。

这个表单包括用户提交的反馈信息，一个可选的e-mail回信地址。 当这个表单提交并且数据通过验证后，系统将自动发送一封包含题用户提交的信息的e-mail给站点工作人员。

我们从contact_form.html模板入手：

    <html>
    <head>
        <title>Contact us</title>
    </head>
    <body>
        <h1>Contact us</h1>

        {% if errors %}
            <ul>
                {% for error in errors %}
                <li>{{ error }}</li>
                {% endfor %}
            </ul>
        {% endif %}

        <form action="/contact/" method="post">
            <p>Subject: <input type="text" name="subject"></p>
            <p>Your e-mail (optional): <input type="text" name="email"></p>
            <p>Message: <textarea name="message" rows="10" cols="50"></textarea></p>
            <input type="submit" value="Submit">
        </form>
    </body>
    </html>

如果我们顺着上一节编写search()视图的思路，那么一个contact()视图代码应该像这样：

    from django.core.mail import send_mail
    from django.http import HttpResponseRedirect
    from django.shortcuts import render_to_response

    def contact(request):
        errors = []
        if request.method == 'POST':
            if not request.POST.get('subject', ''):
                errors.append('Enter a subject.')
            if not request.POST.get('message', ''):
                errors.append('Enter a message.')
            if request.POST.get('email') and '@' not in request.POST['email']:
                errors.append('Enter a valid e-mail address.')
            if not errors:
                send_mail(
                    request.POST['subject'],
                    request.POST['message'],
                    request.POST.get('email', 'noreply@example.com'),
                    ['siteowner@example.com'],
                )
                return HttpResponseRedirect('/contact/thanks/')
        return render_to_response('contact_form.html',
            {'errors': errors})

当邮件发送成功之后，我们使用HttpResponseRedirect对象将网页重定向至一个包含成功信息的页面。 包含成功信息的页面这里留给读者去编写（很简单 一个视图/URL映射/一份模板即可），但是我们要解释一下为何重定向至新的页面，而不是在模板中直接调用render_to_response()来输出。

原因就是： 若用户刷新一个包含POST表单的页面，那么请求将会重新发送造成重复。 这通常会造成非期望的结果，比如说重复的数据库记录；在我们的例子中，将导致发送两封同样的邮件。 如果用户在POST表单之后被重定向至另外的页面，就不会造成重复的请求了。

我们应每次都给成功的POST请求做重定向。 这就是web开发的最佳实践。

这看起来杂乱，且写的时候容易出错。 希望你开始明白使用高级库的用意——负责处理表单及相关校验任务。

###第一个Form类###

Django带有一个form库，称为django.forms，这个库可以处理我们本章所提到的包括HTML表单显示以及验证。 接下来我们来深入了解一下form库，并使用她来重写contact表单应用。

在Django社区上会经常看到 **django.newforms** 这个词语。当人们讨论django.newforms，其实就是我们本章里面介绍的django.forms。

改名其实有历史原因的。 当Django一次向公众发行时，它有一个复杂难懂的表单系统：django.forms。后来它被完全重写了，新的版本改叫作：django.newforms，这样人们还可以通过名称，使用旧版本。 当Django 1.0发布时，旧版本django.forms就不再使用了，而django.newforms也终于可以名正言顺的叫做：django.forms。


表单框架最主要的用法是，为每一个将要处理的HTML的`` <Form>`` 定义一个Form类。 在这个例子中，我们只有一个`` <Form>`` ，因此我们只需定义一个Form类。 这个类可以存在于任何地方，甚至直接写在`` views.py`` 文件里也行，但是社区的惯例是把Form类都放到一个文件中：forms.py。在存放`` views.py`` 的目录中，创建这个文件，然后输入：4

    from django import forms

    class ContactForm(forms.Form):
        subject = forms.CharField()
        email = forms.EmailField(required=False)
        message = forms.CharField()


