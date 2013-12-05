title: Tornado Code Analytics
date: 2013-11-04 20:46:44
tags:
- python
- tornado

categories:
---

# tornado 源码分析

[Tornado](http://www.tornadoweb.org/en/stable/)是一个支持异步请求的python web框架, 最开始由FriendFeed开发, 后被facebook开源。由于使用了非阻塞的网络I/O, Tornado能够支持大量的并发连接, 对于使用[`long polling`](http://en.wikipedia.org/wiki/Push_technology#Long_polling), [`Websockets`](http://en.wikipedia.org/wiki/WebSocket)以及其他支持给用户持续连接(long-lived)的应用, 是一个非常好的选择。
并且tornado的源代码对于python程序员非常值得一读, 它的代码写的非常简洁优雅, 同时也能够了解python web服务器是怎么工作的。

读代码是一种享受, 本篇文章将会介绍tornado的代码架构, 更多细节的地方以后会再写。<!-- more -->

## web.py

`web.py` 模块主要包括：

* class ***Requesthandler***: 定义了一些基本的http方法供子类继承, 如get post方法
    `Requesthandler`以及tornado其他地方的方法都不是线程安全的, `~RequestHandler.write()`, `~RequestHandler.finish()`,
`~RequestHandler.flush()` 只应该在主线程中被调用, 如果使用了多线程, 非常重要的一点是要用 `.IOLoop.add_callback` 在结束请求之前回到主线程上

* decorator ***asynchronous***: 如果请求是异步的那么可以wrap此装饰器。
    如果用上了此装饰器的请求, 将只会在调用了 `self.finish() <RequestHandler.finish>` 方法之后才会结束, 例如：

```python
class MyRequestHandler(web.RequestHandler):
    @web.asynchronous
    def get(self):
       http = httpclient.AsyncHTTPClient()
       http.fetch("http://friendfeed.com/", self._on_download)

    def _on_download(self, response):
       self.write("Downloaded!")
       self.finish()
```

3.1之后推荐使用 `@gen.coroutine`

* decorator ***removeslash***: 去掉请求路径尾部的下划线的装饰器

* decorator ***addslash***: 加上请求路径尾部的下划线的装饰器

* class ***Application***: 一些handlers的集合就组成了一个web application
    这个类的实例可以直接传给HTTPServer:

```python
application = web.Application([
    (r"/", MainPageHandler),
])
http_server = httpserver.HTTPServer(application)
http_server.listen(8080)
ioloop.IOLoop.instance().start()
```

* class ***StaticFileHandler***: 设置小的静态资源的路径(浏览器可以缓存)，大的静态资源，还是用nginx或者apache处理比较妥当，支持HTTP ``Accept-Ranges``的机制返回文件的部分内容，支持版本化的url以便最大化的利用浏览器缓存的效率。这个handler的继承机制也比较复杂，具体的参看源码。

* class ***FallbackHandler(Requesthandler)***: 可以包裹另一个HTTP server的回调函数(牛逼)。
    fallback是一个接收 `~.httpserver.HTTPRequest`(`Application` | `tornado.wsgi.WSGIContainer`)的可执行对象。具体例子：

```python
wsgi_app = tornado.wsgi.WSGIContainer(
    django.core.handlers.wsgi.WSGIHandler())
application = tornado.web.Application([
    (r"/foo", FooHandler),
    (r".*", FallbackHandler, dict(fallback=wsgi_app),
])
```

* class ***OutputTransform***: 修改HTTP request的结果(e.g, GZip encoding)

* class ***GZipContentEncoding(OutputTransform)***: 为response打包一些[静态资源](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11)，比如text/html等。

* class ***ChunkedTransferEncoding(OutputTransform)***: response大文件传输的编码(not sure)

* decorator ***authenticated***: 需要用户登录的装饰器，如果用户没有登录就会跳转到配置过的 `login url <RequestHandler.get_login_url>`。

* class ***UIModule***: 一个可重用的，模块化的UI单元。

* class ***TemplateModule(UIModule)***: 渲染给定的模板。

* class ***URLSpec(object)***: Application里面mapping URLs 和 handlers。

* function ***create_signed_value(secret, name, value)***: 创建签名(用于加密)

* function ***decode_signed_value(secret, name, value, max_age_days=31)***: 解开签名(解密)

## httpserver.py

* class ***HTTPServer(TCPServer)***: 非阻塞单线程HTTPserver.初始化一个`HTTPServer`对象有下面3种方式：

    1. `~tornado.tcpserver.TCPServer.listen`:

    2. `~tornado.tcpserver.TCPServer.bind`/`~tornado.tcpserver.TCPServer.start`:

    3. `~tornado.tcpserver.TCPServer.add_sockets`:
