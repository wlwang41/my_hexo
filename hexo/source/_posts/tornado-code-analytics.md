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

* class Requesthandler: 定义了一些基本的http方法供子类继承, 如get post方法
    `Requesthandler`以及tornado其他地方的方法都不是线程安全的, `~RequestHandler.write()`, `~RequestHandler.finish()`,
`~RequestHandler.flush()` 只应该在主线程中被调用, 如果使用了多线程, 非常重要的一点是要用 `.IOLoop.add_callback` 在结束请求之前回到主线程上
* decorator asynchronous: 如果请求是异步的那么可以warp此装饰器。
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

    3.1之后推荐使用 ``@gen.coroutine``
* decorator removeslash: 去掉请求路径尾部的下划线的装饰器
* decorator addslash: 加上请求路径尾部的下划线的装饰器
* class Application: 一些handlers的集合就组成了一个web application
    这个类的实例可以直接传给HTTPServer:

    ```python
    application = web.Application([
        (r"/", MainPageHandler),
    ])
    http_server = httpserver.HTTPServer(application)
    http_server.listen(8080)
    ioloop.IOLoop.instance().start()
    ```

    StaticFileHandler 艹没看明白
