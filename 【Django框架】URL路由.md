# URL分发器

## 一、视图

视图一般都写在app的`views.py`中，并且视图的第一个参数永远都是`request`(一个`HttpRequest`对象)，这个对象存储了请求过来的所有信息，比如携带的参数以及一些头部信息等。视图中一般是完成逻辑业务相关的操作，比如这个请求添加一个用户，那么可以通过`request`来接受到这些数据，然后存储到数据库中，最后再将执行的结果返回给浏览器，视图函数的返回结果必须时`HttpResponse`对象或者子类的对象。比如：

~~~python
from django.http import HttpResponse

def add_user(request):  # 第一个参数request
    # 添加用户，过程省略

    return HttpResponse('添加用户成功')  # 返回HttpResponse对象
~~~

路由配置：

~~~python
from django.urls import path

from user.views import add_user

# 由一条条映射关系组成的urlpatterns这个列表称为路由表
urlpatterns = [
    path('admin/', admin.site.urls),
    
    path('user/', add_user),  # 应用名称，视图函数
]
~~~

启动服务，输入地址：

![1584524664077](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584524664077.png)

我们试下直接返回字符串，看下什么效果：

~~~python
def add_user(request):
    # 添加用户，过程省略

    # return HttpResponse('添加用户成功')
    return '添加用户成功'

~~~

启动服务，查看效果：
![1584524881566](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584524881566.png)

直接报错。



## 二、URL路由

### 1. 路由作用

将不同的`URL`地址请求分发给不同的视图处理，即将请求地址与视图映射。

当有请求到我们的网站的时候，`Django`会从项目的`urls.py`文件中寻找对应的视图。

![1584532082456](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584532082456.png)

为什么会去`urls.py`文件中寻找映射呢？

是因为在`settings.py`文件中配置了`ROOT_URLCONF = 'django_project.urls'`，所以`Django`会去`urls.py`中寻找。



### 2. URL匹配规则

#### 2.1 普通用法

~~~python
# 普通用法，无参数情况，配置URL及其视图如下：
from django.urls import path
from user.views import add_user

urlpatterns = [
    ....
    path('user/', add_user),
]


# 视图views.py
from django.http import HttpResponse

def add_user(request):
    # 添加用户，过程省略
    return HttpResponse('添加用户成功')

~~~



#### 2.2 传递参数

有时候，`url`中包含了一些参数需要动态调整，比如我博客中某篇文章的详情页的url，是<https://www.qmpython.com/articles/detail-45.html>，其中`45`就是这篇文章的`id`，如果请求的是其他文章，则这个`id`会动态变换，那博客文章的详情页面`url`就可以写成`https://www.qmpython.com/articles/detail-<id>.html`，其中`id`就是文章的`id`。那如何在`django`实现这种需求呢？

~~~python
# 传递参数情况
# 路由urls.py配置
path('user/<int:user_id>/', user_detail),

# 视图views.py
def user_detail(request, user_id):  # 视图也要写对应参数，入参user_id一定要与url中的参数命一致

    return HttpResponse('用户【{}】的详细信息'.format(user_id))

~~~



![1584545384634](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584545384634.png)



![1584545410903](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584545410903.png)

上面url中`<int:user_id>`表示为`user_id`变量添加了一个`int`转换器， Django在解析这个URL变量时会将其转换为整型。Django中提供了一些转换器可以在URL规则里使用，如下：

~~~python
# 具体可查看from django.urls import converters  类中逻辑

DEFAULT_CONVERTERS = {
    'int': IntConverter(),   # 整型
    'path': PathConverter(),  # 包含斜线的字符串
    'slug': SlugConverter(),
    'str': StringConverter(),  # 字符串，不包含斜线，遇到斜线就当作url相对路径，path则当作一个字符串接收。
    'uuid': UUIDConverter(),   # UUID字符串
}
~~~

URL中的变量部分默认类型为字符串。

这不仅仅是转换变量类型，还包括URL匹配，比如我们前面的例子查看某个用户的详细信息，url中定义的是int型转换器，但是在请求的时候传的是字母，导致转换int失败，这样url也不匹配。

![1584547247296](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584547247296.png)





还有种方式就是以查询字符串传递参数：

~~~python
# 配置路由，不需要传参
path('user/profile/', user_profile)

def user_profile(request):  # 也不需要定义额外的入参
    user_id = request.GET.get('id')   # 通过 request.GET.get获取查询字符串
    return HttpResponse('用户【{}】的个人首页'.format(user_id))

~~~

![1584546410112](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584546410112.png)

#### 2.3 正则url写法

有时候我们在写url匹配的时候，想要写使用正则表达式来实现一些复杂的需求，那么这时候我们可以使用`re_path`来实现，和`path`参数类似，只不过路径使用的是正则表达式。

##### 2.3.1 不传参情况

~~~python
# 应用urls.py
from django.urls import path, re_path

re_path(r'^birthday/[0-9]{4}/$', show_birthday)

# 视图views.py
def show_birthday(request):
    return HttpResponse('用户生日')

~~~

查看效果：

![1584623873191](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584623873191.png)

试下不满足正则表达式的情况：
![1584623940029](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584623940029.png)



##### 2.3.1 传参情况

上面是没有传递参数的情况下，那需要传递参数呢？

1、位置传参

~~~python
# 
# 应用urls.py
from django.urls import path, re_path

# 加上圆括号的时候，django就能从URL中捕获这一个值并传递给相对应的views函数，这里使用的是位置传参。
re_path(r'^birthday/([0-9]{4})/$', show_birthday)

# 视图views.py
def show_birthday(request, year):
    print(year, type(year))  # 2020 <class 'str'>
    return HttpResponse('用户生日，年份：%s'%year)
~~~



![1584625524261](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584625524261.png)

总结：

* 域名部分会被过滤掉；
* 不需要添加一个前导的反斜杠，因为每个`URL` 都有。例如，应该是`^test `而不是`^/test`；
* 每个正则表达式前面的`r'`是可选的但是建议加上，表示字符串中任何字符都不转义；
* 若要从URL 中捕获一个值，只需要在它周围放置一对圆括号。
* 任何组匹配的变量，都会以字符串的形式传递给`view`, 例如通过`([0-9]{4})`匹配出了`2019`，但`2019`会被当做字符串传递给`year`，所以我们打印出类型是`str`。



从这里可以看出，视图的参数是根据URL的正则式，按顺序匹配并自动赋值的。虽然这样可以实现任意多个参数的传递，但是却不够灵活，URL看起来很混乱，而且由于是正则匹配，有些情况下容易出错。那我们可以使用关键字传参。

2、关键字传参

语法是`(?P<name>pattern)`，其中`name`是指传递参数的名字，`pattern`是指匹配模式。

~~~python
# 应用urls.py
from django.urls import path, re_path

re_path(r'^birthday/(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/$', show_birthday)


# 视图views.py
def show_birthday(request, year, month, day):
    return HttpResponse('用户生日日期：%s-%s-%s'%(year, month, day))
~~~



![1584626335261](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584626335261.png)

一般情况下使用`path`就可以解决，除非一些特定的需求。



### 3. URL分层模块化

我们上面的`user`应用，所有有关`user`的请求都在项目根目录下的总`urls.py`进行映射。

~~~python
from django.contrib import admin
from django.urls import path

from user.views import add_user, user_detail, user_profile

urlpatterns = [
    path('admin/', admin.site.urls),

    path('user/', add_user),
    path('user/<int:user_id>/', user_detail),
    path('user/profile/', user_profile),

]
~~~

几个`url`地址还好，如果项目非常庞大，应用非常多，应用的 `URL` 都写在根` urls.py `配置文件中的话，会显的非常杂乱，还会出现名称冲突之类的问题，这样对开发整个项目是非常不利的。可以这样解决，把每个应用的 `URL`
写在它们各自的 `urls.py `配置文件里，然后在根 `urls.py `里用 `include() `函数引用

~~~python
# 在项目的主 urls.py 配置文件改为：
from django.urls import path, include

urlpatterns = [
    path('user/', include('user.urls')),  # 应用名字.urls
]

# 在应用app中新建一个urls.py文件放置具体的
from django.urls import path
from .views import add_user, user_detail, user_profile

urlpatterns = [
    path('', add_user),
    path('<int:user_id>/', user_detail),  # 开头不要用/，因为主url里面末尾写了/
    path('profile/', user_profile),
]
~~~

我们来查看下效果：

![1584593084533](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584593084533.png)

![1584593057447](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584593057447.png)

![1584593126798](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584593126798.png)

通过查看运行效果是一样的。



### 4. url重定向

我们经常在网站上可以看到，比如我们进入某个页面，如果没有登录，则直接会跳转到登录界面，让我们先登录。这个时候就需要用到重定向`redirect`。

~~~python
# 主urls.py
path('user/', include('user.urls')),

# 应用urls.py
from .views import index, login

path('index/', index),
path('login/', login),

# 视图views.py
def index(request):
    username = request.GET.get('username')
    if not username:
        return redirect('/user/login/')  # 从定向跳到另外一个地址，因为url地址开始肯定是一个/，  如果你不带斜杠的话，它就会当成一个相对地址，它自会自动在当前的url中往后添加。

    return HttpResponse('用户首页')


def login(request):
    return HttpResponse('用户登录页面')
~~~



没传查询字符串的时候：

![1584599066409](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584599066409.png)



传了参数之后：

![1584599190779](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584599190779.png)



### 5. url命名与反向解析

在实际需求中，可能url是经常变化的，如果在代码中写死，如果需要修改，则需要将所有涉及此url地址的地方都修改，这样比较繁琐，也有可能遗漏。我们可以给`url`取个名字，后续需要使用`url`地址的地方，直接使用它的名字，然后再进行解析即可，这样如果需要修改地址，只需要在`urls.py`中修改即可。

~~~python
# 应用urls.py

path('index/', index, name='index'),   # name给url命名
path('login/', login, name='login'),

# 视图views.py

from django.shortcuts import redirect, reverse

def index(request):
    username = request.GET.get('username')
    if not username:
        #return redirect('/user/login/')
        
        print(reverse('login'))   # /user/login/
        return redirect(reverse('login'))

    return HttpResponse('用户首页')
~~~

如果在反转`url`的时候，需要添加参数，那么可以传递`kwargs`。

~~~python
# 应用urls.py
path('index/', index, name='index'),
path('detail/<int:user_id>/', user_detail, name='detail'),


# 视图views.py
def index(request):
    detail_url = reverse('user:detail', kwargs={'user_id': 1})  # 带命名参数的跳转
    print(detail_url)   # /user/detail/1/
    
    # return redirect(reverse('login3',args=("2018", "03","21"))) 非命名参数的跳转
    return redirect(detail_url)


def user_detail(request, user_id):
    return HttpResponse('用户【{}】的详细信息'.format(user_id))

~~~

![1584631114552](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584631114552.png)







如果需要添加查询字符串的参数，则必须字符串的拼接。

~~~python

path('index/', index, name='index'),
path('login/', login, name='login')


def index(request):
    username = request.GET.get('username')
    if not username:
        login_url = reverse('user:login') + '?next=/'   # 拼接查询字符串参数
        print(login_url)    # /user/login/?next=/
        return redirect(login_url)

    return HttpResponse('用户首页')

def login(request):
    return HttpResponse('用户登录页面')

~~~

![1584631583842](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584631583842.png)







URL反向解析一般是通过`reverse`函数，以及模板中的url标记实现。

在需要URL 的地方，对于不同层级，Django 提供不同的工具用于URL 反查：
1）在模板中：使用url 模板标签。
2）在Python 代码中：使用`reverse` 函数。
3）在更高层的与处理`Django`模型实例相关的代码中：使用`get_absolute_url() `方法。



### 6. 应用命名空间

在多个应用`app`之间，有可能对不同app下的url都取了相同别名，`，比如我用户模块有个首页取名`index`，网站首页也取名为`index`，在使用的时候，都直接使用`index`，那这个时候应该反向解析为哪各呢？为了避免反向解析`url`的时候产生混淆，可以使用应用命名空间来区分。

~~~python
# 在应用urls.py

app_name = 'user'    # 应用命名空间

# 反向解析url的时候，使用：应用命名空间:url名称
# 视图views.py
def index(request):
    username = request.GET.get('username')
    if not username:
        #return redirect('/user/login/')

        print(reverse('user:login'))   # /user/login/
        return redirect(reverse('user:login'))   # 应用命名空间:url名称

    return HttpResponse('用户首页')

~~~



### 7. 实例命名空间

一个app，可以创建多个实例，可以使用多个url映射同一个app，所以这就会产生一个问题，以后在做反转的时候，如果使用应用命名空间，那么就会发生混淆，为了避免这个问题，我们可以使用实例命名空间，实例命名空间也是非常简单。

~~~python
# 主urls.py

# 同一个app下有2个实例
path('user/', include('user.urls')),
path('user2/', include('user.urls')),

# 应用urls.py
path('index/', index, name='index'),


# 视图views.py
def index(request):
    username = request.GET.get('username')
    if not username:
        #return redirect('/user/login/')

        print(reverse('user:login'))   # /user/login/
        return redirect(reverse('user:login'))

    return HttpResponse('用户首页')

~~~

我们请求`/user2/index/`，没有进行登录，我们期望跳转到`/user2/login/`界面中，但是结果不如所愿，跳到`/user/login/`去了。

![1584612629374](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584612629374.png)



为了避免这种情况，我们就需要用到实例命名空间：

~~~python
# 主urls.py
# 同一个app下有2个实例
path('user/', include('user.urls', namespace='user')),  # 在include中使用namespace
path('user2/', include('user.urls', namespace='user2')),

# 应用urls.py
path('index/', index, name='index'),
path('login/', login, name='login'),

# 视图views.py
def index(request):
    username = request.GET.get('username')
    if not username:
        current_namespace = request.resolver_match.namespace  # 获取当前实例
        print(current_namespace)  # user2
        return redirect(reverse('%s:login'%current_namespace))  # 实例名称:url名称

    return HttpResponse('用户首页')


def login(request):
    return HttpResponse('用户登录页面')
~~~

浏览效果，显示正确：	![1584613472783](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584613472783.png)





### 8. 自定义url转换器

此处先省略，后续补上





### 9. URL映射的时候指定默认参数

使用`path`或者是`re_path`，在路由中都可以利用分组为视图指定一个默认参数来匹配多条规则。

~~~python
path('detail/', user_detail),
path('detail/<int:user_id>/', user_detail)

def user_detail(request, user_id=1):
    return HttpResponse('用户【{}】的详细信息'.format(user_id))
~~~

当在访问/user/detail/时，没有传递user_id参数，所以会匹配第一个url，执行user_detail视图函数，而在函数中，有user_id默认参数，因此这时候就可以不用传递参数，而如果访问/user/detail/1/的时候，传递了参数，会匹配第二个url，执行函数时，使用传递的参数。