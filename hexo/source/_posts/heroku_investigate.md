title: Heroku Investigate
date: 2013-12-05 16:01:44
tags:
- Heroku

categories: Investigate
---

# heroku 技术调研

### Add-ons

[heroku的插件系统](https://addons.heroku.com/)还是非常给力的。基本上我们要用的一些东西这些插件商店上面都有。管理的话看它给的简单的[文档](https://devcenter.heroku.com/articles/managing-add-ons) <!-- more -->

* **Mongo**: 我们这个应该是会用mongodb作为我们的主数据库，在nosql里面还是mongo写起来比较顺手(cassandra的count太坑)。在heroku的插件商店里面提供mongo服务的不只一个。主要是[MongoLab](https://addons.heroku.com/mongolab#dedicated-cluster) 和 [MongoHQ](https://addons.heroku.com/mongohq#ssd_150g)。
> NOTE: 这两个里面我比较倾向的是MongoHQ(不要钱的情况下)，理由：
>
> 1. MongoHQ的容量是512m，MongoLab只有496m
> 2. MongoHQ提供健康检查，MongoLab不提供
> 3. 遗憾的是都没有提供newrelic的dashboard，比较担心的是到时候的监控问题

* **Redis**: 这个没得说，就用[Reids Cloud](https://addons.heroku.com/rediscloud#25)，不要钱的有2个，这个大小是另一个的5倍。simple as that.

* **Search**: 这个没有找到好的免费的add-ons，所以到时候可能看是用elasticsearch搞一个还是用sphinx搞一个

* **MQ**: 这个有一个rabitMQ的插件。叫[RabbitMQ Bigwig](https://addons.heroku.com/rabbitmq-bigwig#pipkin)。估计我们不会这个的。

* **Analytic**: 前段时间看到一个叫[New Relic](https://addons.heroku.com/newrelic#wayne)的，也注册了，Heroku正好的它的add-ons就用呗。

* **Monitoring**: 有[sentry](https://addons.heroku.com/sentry)，就用它，公司的项目也用的sentry做的异常的监控，debug特别方便
