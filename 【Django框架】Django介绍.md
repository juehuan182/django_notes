### Django介绍

Django是一个开放源代码的Web应用框架，由Python写成，它源自一个在线新闻Web站点。它采用MTV模式，即模型M，模板T和视图V，Django在近年来发展迅速，应用也越来越广发。Django是一个全功能的Web开发框架，Web开发所需要的一切它都包含了，你不需要去选择，只需要去熟悉，然后使用。

列举常用的内置功能：

* HTTP的封装：request和response
* ORM
* admin
* Form
* template
* session和cookie
* 权限
* 安全
* cache
* Logging
* sitemap
* RSS

上面只是列举常用的部分，Django提供了的功能远超过我们想要的。

Django相关参考资料：

* 官网：<https://www.djangoproject.com/>
* Django第三方插件：https://djangopackages.org/
* 基于Django的网站：https://www.djangosites.org/



Django作为一个从新闻系统生成环境中诞生的框架，是直接面向企业级开发的。对于Web开发来说，尤其是内容驱动的项目，推荐使用Django来开发，因为即使选择了Flask或者其他微型框架，然后把插件拼装起来，最终也是基于松散的配置做了一个类Django的框架，还不如直接使用Django整合性强，基于Django内置的功能，我们可以省去自己造轮子的时间。



### Web服务器和应用服务器以及Web应用框架

* web服务器：负责处理http请求，响应静态文件，常见有`Apache、Nginx`以及微软的`IIS`。
* 应用服务器：负责处理逻辑的服务器，比如`java,python,php`的代码，是不能直接通过`nginx`这种web服务器来处理的，只能通过应用服务器来处理，常见的应用服务器有`uwsgi、tomcat`等。
* web应用框架：一般使用某种语言封装了常用的web功能的框架就是web应用框架，`flask、Django、tornado`以及Java中的SSH框架都是web应用框架。



关系图：

![1584441208106](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584441208106.png)





### Django框架分层与请求生命周期

Django是基于`MVC(Model-View-Controller)`模式的框架，但是在Django中，控制器接受用户输入的部分由框架自行处理，所以Django里更关注的是模型`Model`、视图`View`和模板`Template`，简称`MTV`模式。

* 模型（`Model`）
  直接同数据库打交道的一层，数据处理的部分都在这一层。通常每个模型对应数据库中唯一的一张表。详见`models.py`。
*  视图（`View`）
  负责接收用户请求，进行业务处理，并返回响应逻辑。详见`views.py`
* 模板（`Template`）
  负责决定显示内容，模板包含所需的HTML输出的静态部分，以及一些特殊的语法，描述如何将动态内容插入模板中。详见`templates`

除了上面三层之外，还需要一个`URL`配置文件（`URLconf`），它的作用是将不同的URL地址请求分发给不同的视图处理，视图再调用响应的模型和模板。URLconf机制使用正则表达式匹配URL，然后调用合适的函数。详见`urls.py`



我们先看一下Django处理请求示意图：

![1584440658629](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584440658629.png)





`MTV`架构流程如下：







这种设计模式，最终目的都是解耦，把一个软件系统划分为一层层的结构，让每一层的逻辑更加纯粹，便于开发人员维护。



后面我们陆续详细学习每一层。











