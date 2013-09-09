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


