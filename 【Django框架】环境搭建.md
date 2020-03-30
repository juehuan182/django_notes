# 项目环境搭建

## 一、创建Django项目

### 1. 创建python虚拟环境

实际工作中是在Linux上进行开发，所以最好是在虚拟机上或者连接阿里云等各种云进行开发。我这里直接在阿里云上，使用`Centos 7`中进行开发。

我们前面已经学习过怎么样搭建python虚拟环境，详情参考[Centos7中部署Python环境](https://www.qmpython.com/articles/detail-36.html)。

这里我们新创建个虚拟环境用于学习。

~~~bash
mkvirtualenv django_project   # 创建虚拟环境

workon django_project   # 激活虚拟环境
~~~

![1584442432386](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584442432386.png)



### 2. 安装django

`Django`框架目前最新版本`3.x`了，目前版本之前没什么太大差别，具体差异请自行查阅资料，我们这里选择2.x安装。

~~~bash
pip install django==2.1.8   # 安装django框架
# pip install django ~=2.x    ~=表示安装指定版本的最新版本，x表示2版本的最新版本

# 查看安装的模块
(django_project) [root@qmpython ~]# pip list
Package    Version
---------- -------
Django     2.1.8  
pip        20.0.2 
pytz       2019.3 
setuptools 46.0.0 
wheel      0.34.2 
~~~



### 3.创建项目

我们先创建一个存放项目的目录，比如我们这里存放到`/root/src/www`下。

~~~bash
cd /root/src
mkdir www   # 没有就创建
~~~

#### 3.1 用命令行方式

##### 3.1.1 创建项目

~~~bash
# 1、cd进入保存项目的目录下
cd /root/src/www

# 2、创建Django项目，django-admin startproject 项目名称
django-admin startproject django_project


(django_project) [root@qmpython www]# ls
demo_test  django_project  QmpythonBlog
(django_project) [root@qmpython www]# cd django_project/
(django_project) [root@qmpython django_project]# tree
.
├── django_project
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 5 files
~~~

此时，成功创建一个名为`django_project`的`Django`项目，我们可以通过上面查看目录结构。



##### 3.1.2 创建应用

Django框架通过app应用的方式来管理整个网站项目，一个网站中包含多个子业务模块，比如用户模块、商品模块，新闻模块等，我们可以将这些子模块称作一个应用(app)。



~~~bash
# 进入项目根目录下
cd www/django_project

# 创建应用，python manage.py startapp 应用名称
(django_project) [root@qmpython django_project]# python manage.py startapp user
(django_project) [root@qmpython django_project]# tree
.
├── django_project
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-37.pyc
│   │   └── settings.cpython-37.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
└── user
    ├── admin.py
    ├── apps.py
    ├── __init__.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    └── views.py

4 directories, 14 files
~~~

我们可以看到成功创建了user应用，其中包含文件含义如下：

* `__init__.py`：空文件，指定当前目录可作为包使用。
* `tests.py`：用于开发测试用例，在实际开发中，如果需要对模块进行测试，可在此文件中编写测试代码。
* `views.py`：视图文件，编写视图相关代码。
* `models.py`：模型文件，编写模型相关代码。
* `migrations.py`：与模型移植相关。



注意，创建的应用需要在settings.py模块中进行配置才能够被项目识别。

~~~python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'user',   # 配置应用
]
~~~



##### 3.1.3 启动项目

在开发阶段，为了能够快速预览到网站的显示和运行效果，Django提供了一个纯Python编写的轻量级Web服务器，仅在开发阶段使用。

~~~bash
# 进入项目根目录下，即manage.py文件所在路径下
cd /root/src/www/django_project

# 启动服务器，python manage.py runserver ip:port
python manage.py runserver 47.107.x.x:5003
~~~

报错：`Error: That IP address can't be assigned to.`

但是使用localhost、或者0.0.0.0 就可以成功启动，但只是进行无法外网访问。

字面理解这句话的意思就是：该IP地址不能被分配。

我们有几种解决方案：

1、查看`settings.py`文件的`ALLOWED_HOSTS`是否配置了该`ip`地址，如果没有的话就加上，开发时，建议把`0.0.0.0 、127.0.0.1 、localhost`这些都添加上去。

~~~python
ALLOWED_HOSTS = ['0.0.0.0','127.0.0.1','localhost','47.107.x.x']
~~~



2、如果使用的是服务器的话，请在服务器控制台的防火墙设置里，查看是否添加了这个端口。

3、如果上述方法均不可以，那么就：

~~~bash
python manage.py runserver 0:8004
~~~

直接把启动的ip设置为0，然后问题就解决了，配置好防火墙之后，就可以直接用域名或者是服务器ip访问了 。

 ![服务器启动后效果](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584455965180.png)

此时，在浏览器中输入网址，看效果如下，表示启动服务器正常能够访问。

![1584456422078](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584456422078.png)

在以后的开发中，就可以使用该网址和端口查看当前项目的开发效果了。

如果在项目中执行了增删改查文件操作，服务器会自动重启，不需要手动启动（当然后期如果使用了nginx服务器，修改静态文件则需要重启），如果要停止，直接ctrl+c。



#### 3.2 使用pycharm创建

##### 3.2.1 创建项目 

![1584503437489](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584503437489.png)

![1584503575826](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584503575826.png)



配置之前创建的虚拟环境Python解释器：

![1584459380389](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584459380389.png)

![1584459449081](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584459449081.png)

![1584459730774](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584459730774.png)

配置远程Django项目根目录：

![1584503875072](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584503875072.png)

`Rmote project location`，是python脚本上传到远程服务器Django项目的根目录，这个路径会自动同步到`Tools中`的`mapping`里面；注意：这个路径需要在虚拟环境中存在，没有则新建！



##### 3.2.2 创建应用

设置`Application name`，即app应用程序，否则需要用命令创建。后续其他app还是需要使用命令行方式创建。

![1584504031558](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584504031558.png)



![1584504190787](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584504190787.png)

注意：有的时候，可能某些原因导致pycharm看不到生成相关文件，这时不要以为没创建成功，可以先去远程服务器相关目录下有没有生成。如果有，就使用后面同步设置方法进行同步。



![1584504240309](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584504240309.png)



注意：使用pycharm相对于命令行方式生成了模板文件存放路径`templates`，如果使用命令行的方式创建，则只需要手工创建即可。



##### 3.2.3 设置同步配置

设置同步配置，用于将本地代码同步到服务器，或者将服务器代码下载到本地中。

![1584504503414](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584504503414.png)



![1584504920656](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584504920656.png)





![1584504897334](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584504897334.png)



![1584504872318](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584504872318.png)





![1584505053477](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505053477.png)





![1584505141193](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505141193.png)





![1584505250484](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505250484.png)







![1584505457270](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505457270.png)



也可以打开远程项目目录窗口

![1584505539798](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505539798.png)



![1584505591181](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505591181.png)





![1584505618533](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505618533.png)



也可以这里直接下载上传或者对比本地和远程服务器项目差异

![1584505724728](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505724728.png)





##### 3.2.4 配置项目远程拟环境

查看当前项目是否连接所需环境，如需修改配置当前工程的远程环境，则：

![1584511555069](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584511555069.png)



![1584511627209](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584511627209.png)



##### 3.2.5 配置运行环境

![1584505749627](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505749627.png)





![1584505846188](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584505846188.png)

`0.0.0.0` 或者 `0 `， 代表任何`IP`都允许访问，`8004：` 代表我们对外的端口，默认端口为80。



`settings.py`修改配置，允许ip地址访问。

~~~python
ALLOWED_HOSTS = ['*']    # 测试环境配置*，允许任何ip地址访问
~~~

应用配置，使用pycharm创建的第一个app，系统会默认配置好。

~~~python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'user.apps.UserConfig',   # 默认配置好，若是用命令方式，这里则需要配置应用名称即可。
]
~~~



##### 3.2.6 启动项目

![1584511699117](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584511699117.png)

![1584511743146](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584511743146.png)

浏览器输入网址进行访问：

![1584511902886](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584511902886.png)



至此使用Pycharm创建Django项目成功。



## 二、项目结构介绍

我们来学习下，Django的项目结构，我们先看下目录结构：

~~~
[root@qmpython django_project]# tree
.
├── db.sqlite3
├── django_project
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-37.pyc
│   │   ├── settings.cpython-37.pyc
│   │   ├── urls.cpython-37.pyc
│   │   └── wsgi.cpython-37.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
├── templates
└── user
    ├── admin.py
    ├── apps.py
    ├── __init__.py
    ├── migrations
    │   ├── __init__.py
    │   └── __pycache__
    │       └── __init__.cpython-37.pyc
    ├── models.py
    ├── __pycache__
    │   ├── admin.cpython-37.pyc
    │   ├── apps.cpython-37.pyc
    │   ├── __init__.cpython-37.pyc
    │   └── models.cpython-37.pyc
    ├── tests.py
    └── views.py

7 directories, 22 files
~~~



其中：

- `manage.py`：项目运行的入口文件，执行项目配置文件路径。以后和项目交互基本都是基于这个文件，一般在终端输入`python manage.py 子命令`，可以输入`python manage.py help`查看能执行哪些命令。
- `__init__.py`：空文件，指定当前目录可作为包使用。
- `settings.py`：整个项目的配置文件，例如配置应用、模板目录、静态文件目录、邮件发送等。
- ` urls.py`：是项目的总URL配置文件，在该文件中将用户请求的URL对应到某个视图函数。
- `wsgi.py`：项目与支持WSGI协议的Web服务器对接的入口文件。

