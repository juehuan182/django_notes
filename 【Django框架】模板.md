## 一、模板介绍

我们之前学习的，都是在视图函数直接返回文本，在实际中我们更多的是带有样式的`HTML`代码，这样可以让浏览器渲染出非常漂亮的页面，目前市面上有非常多的模板系统，其中最常用的是`DTL`和`Jinja2`，`DTL(Django Template Language)`，也就是`Django`自带的模板语言，当然也可以配置`Django`支持`Jinja2`，但是作为`Django`内置的模板语言，不会产生一些不兼容的情况，最好还是使用内置的。

### 1. DTL与普通的HTML文件区别

DTL模板是一种带有特殊语法的HTML文件，这个HTML文件可以被Django编译，可以传递参数进去，实现数据动态化，在编译完成后，生成一个普通的HTML文件，然后发送给客户端。



### 2. 模板渲染

可以使用`render`函数进行渲染。

我们在之前的基础上，再新建一个`article`的`app`，除了刚开始第一个应用，我们在`pycharm`创建项目的时候，填了`app`名字，`pycharm`会自动帮我们创建应用，后续如果还要再新建应用，我们都要使用命令方式才能新建`app`。

![1584670885528](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584670885528.png)

![1584671026345](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584671026345.png)

![1584671223615](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584671223615.png)

![1584671267402](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584671267402.png)

![1584671308976](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584671308976.png)



应用创建完成了，接下来我们需要在配置文件`settings.py`里面注册应用app。

~~~python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'user.apps.UserConfig',   # 使用pycharm创建的应用，pycharm会自动注册
    'article'  # 手工命令方式创建的应用app，需要手工在这注册应用！
]
~~~

路由和视图代码：

~~~python
# 主urls.py
path('article/', include('article.urls', namespace='article'))
    
# 应用urls.py，创建的应用这个文件不会自动生成，需要我们新建

from django.urls import path
from . import views

app_name = 'article'
urlpatterns = [
    path('index/', views.index, name='index')

]

# 视图views.py
from django.shortcuts import render

def index(request):
    return render(request, 'index.html')  # 使用render渲染模板
~~~

模板文件，项目根目录下`templates`目录（若没有自动生成，则手工新建此文件夹）下新建`index.html`模板文件。

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
</head>
<body>
    <h2>文章首页</h2>
</body>
</html>
~~~

查看效果：
![1584672280886](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584672280886.png)

使用`render`函数将模板成功渲染出来了。



### 3. 模板文件查找路径

在项目的`settings.py`配置文件中，有一个`TEMPLATES`配置。

~~~python
TEMPLATES = [
    {
        # 模板引擎，就是Django内置的模板引擎，
        # 若要配置为Jinja2：django.template.backends.jinja2.Jinja2
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # BASE_DIR，项目根目录路径下的templates查找
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        # 是否从app应用下的templates下查找模板文件
        'APP_DIRS': True,
        'OPTIONS': {
            # 模板中间件
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
~~~

这个配置包含了模板引擎的配置，这个配置包含了模板引擎的配置，模板查找路径的配置，模板上下文的配置等，模板路径可以2个地方配置。

1. `DIRS`：这是一个列表，在这个列表中可以存放所有的模板路径，以后在视图中使用`render`渲染模板时，就会从这个列表的路径中查找模板。
2. `APP_DIRS`：默认为`True`，这个表示已经在`INSTALLED_APPS`文件中注册过的应用app下的`templates`目录下查找模板文件。
3. 查找顺序：比如代码`render(request,'index.html')，`先会在`DIRS`这个列表中依次查找路径下有没有这个模板，如果有，就返回，如果没有，则会先检查当前视图所处的`app`是否已经安装，那么就先在当前app下的`templates`目录下查找模板，如果没有找到，就会去其他注册的`app`中继续查找，如果所有路径下都没有找到，则会抛出`TemplateDoesNotExist`。



## 二、DTL模板语法

### 1. 模板变量

我们想要将后台数据渲染到页面上，这就需要用到模板变量。

~~~python
# 主urls.py
path('article/', include('article.urls', namespace='article'))

# 应用urls.py
path('index/', views.index, name='index')



# 视图views.py
from django.shortcuts import render

class Article:
    def __init__(self, title):
        self.title = title


def index(request):
    context = {
        'msg': '模板变量',    # 字符串,
        'article': Article('Django框架之模板文件'),    # 实例一个对象,
        'author': {'name': 'admin'},    # 一个字典
        'tag': ['python入门', 'python进阶', '数据库']  # 列表或元组
    }

    return render(request, 'index.html', context=context)  # 传入一个字典
~~~

模板文件：

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
</head>
<body>
    <h2>文章首页</h2>
    <p>{{ msg }}</p> <!-- 在模板中使用变量，需要将变量放到 {{ 变量 }} 中 -->
    <h3>{{ article.title }}</h3>  <!-- 要访问对象的属性，通过"对象.属性名" 访问 -->
    <p>作者：{{ author.name }}</p>  <!-- 要访问字典key对应值value，通过"字典.key" 访问，不能[]形式 -->
    <ul>
        <li>{{ tag.0 }}</li>  <!-- 要访问列表或者元组的元素，通过"列表.索引" 访问 -->
        <li>{{ tag.1 }}</li>
        <li>{{ tag.2 }}</li>
    </ul>
</body>
</html>
~~~

效果：

![1584697151373](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584697151373.png)



在模版中使用变量：语法：{{变量名}}
1）命名由字母和数字以及下划线组成，不能有空格和标点符号。
2）不要和python或django关键字重名。原因：如果data是一个字典，那么访问data.items将会访问data这个字典的key名为items的值，而不会访问字典的items方法。



### 2. 模板标签

标签语法：`{% 标签名称 %}  {% 结束标签名称 %}`。

例：` {%tag%} {%endtag%}`

#### 2.1 if标签

`if`标签相当于python中的if语句，有`elif`和`else`相对应，可以使用`and、or、in、not、==、!=、<=、>=、is、is not`，来进行判断。

视图`views.py`：

~~~python
def index(request):
    age = random.randint(6, 20)  # 随机生成6-20的整数

    context = {
        'age': age
    }

    return render(request, 'index.html', context=context)
~~~

模板文件：

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
</head>
<body>
    <p>
        <span>年龄：<strong>{{ age }}</strong>，</span>
        <span>适合读</span>
        <span>
            {% if age >= 18 %}
                【大学】书籍
            {% elif 12 < age and age < 18 %}
                【高中】书籍
            {% else %}
                【小学】书籍
            {% endif %}
        </span>
    </p>
</body>
</html>
~~~

![1584705976891](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584705976891.png)

再刷新一次：

![1584706002969](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584706002969.png)

#### 2.2 for...in...标签

`for`标签相当于python中的`for`循环，可以遍历列表、元组、字符串、字典等一切可以遍历的对象。

~~~python
# 视图views.py
def index(request):
    context = {
        'articles': [              # 列表
            '21天学会Python',
            'C++从入门到入土',
            'Mysql从入门到跑路'
        ],
        'author': {             # 字典
            'name': '老P',
            'age': 18,
            'sex': 'boy'
        },
        'artile_details': [
            {'title': '21天学会Python', 'author': '小P', 'words': 1000},
            {'title': 'C++从入门到入土', 'author': '中P', 'words': 2000},
            {'title': 'Mysql从入门到跑路', 'author': '老P', 'words': 3000}
        ]
    }

    return render(request, 'index.html', context=context)
~~~

模板文件：

~~~django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
</head>
<body>
    <ul>
        {% for article in articles %}
            <li>{{ article }}</li>
        {% endfor %}
    </ul>
    <p>获取字典键：</p>
    <ul>
        {% for key in author.keys %}
            <li>{{ key }}</li>
        {% endfor %}
    </ul>
    <p>获取字典值：</p>
    <ul>
        {% for key in author.values %}
            <li>{{ key }}</li>
        {% endfor %}
    </ul>
    <p>获取字典键值对：</p>
    <ul>
        {% for key, value in author.items %}
            <li>{{ key }}：{{ value }}</li>
        {% endfor %}
    </ul>

    <p>文章信息列表：</p>
    <table border="1">
        <thead>
            <tr>
                <th>序号</th>
                <th>标题</th>
                <th>作者</th>
                <th>字数</th>
            </tr>
        </thead>
        <tbody>
            {% for article_detail in artile_details %}
                {% if forloop.first %}   <!-- 是否第一次遍历 -->
                    <tr style="background: red;">
                {% elif forloop.last %}   <!-- 是否最后一次遍历 -->
                    <tr style="background: pink;">
                {% else %}
                    <tr>
                {% endif %}
                    <td>{{ forloop.counter }}</td>  <!-- 当前迭代的次数，下标从1开始。-->
                    <td>{{ article_detail.title }}</td>
                    <td>{{ article_detail.author }}</td>
                    <td>{{ article_detail.words }}</td>
                </tr>
            {% endfor %}
        </tbody>
    </table>

</body>
</html>

<!-- 
    forloop.counter：当前迭代的次数，下标从1开始。
    forloop.counter0：当前迭代的次数，下标从0开始。
    forloop.first：返回bool类型，如果是第一次迭代，返回true,否则返回false。
    forloop.last：返回bool类型，如果是最后一次迭代，返回True，否则返回False。
-->
~~~

![1584717272397](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584717272397.png)

还有个`for...in...empty`，在遍历的时候如果需要遍历的对象没有为空，则执行`empty`下的。

~~~jinja2
    {% for comment in comments %}  
        {{ comment }}
    {% empty %}   <!-- comments没有或为空，则执行empty下的。-->
        还没有评论~~~
    {% endfor %}
~~~



#### 2.3 with标签

使用一个简单地名字缓存一个复杂的变量，当你需要使用一个代价较大的方法（比如访问数据库）很多次的时候这是非常有用的。

~~~jinja2
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
</head>
<body>
    {{ title }}

    {% with t1=title %}  <!-- 使用= -->
        {{ t1 }}
    {% endwith %}

    {% with title as t2%} <!-- 使用as 取别名 -->
        {{ t2 }}
    {% endwith %}
</body> 
</html>
~~~

![1584759867131](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584759867131.png)

注意：定义的变量只能在`with`语句块中使用，在`with`语句块外面使用取不到这个变量。



#### 2.4 url标签

在模板中，经常要写一些超链接，比如`a`标签中需要定义`href`属性，当然如果通过硬编码的方式直接将这个`url`写死也是可以，但是不便于以后维护，比如需要修改`url`地址，那涉及到的地方都要修改，因此建议使用这种反转的方式来实现，类似于`django`中的`reverse`一样。

~~~python
# 应用urls.py
from django.urls import path
from . import views

app_name = 'article'
urlpatterns = [
    path('index/', views.index, name='index'),
    path('book/', views.book, name='book'),
    path('movie/', views.movie, name='movie'),
    path('city/', views.city, name='city'),
    path('book/hot/<int:book_id>', views.hot_book, name='hot_book')
]

# 视图views.py
from django.shortcuts import render, HttpResponse

def index(request):
    return render(request, 'index.html')

def book(request):
    return HttpResponse('读书页面')

def movie(request):
    return HttpResponse('电影页面')

def city(request):
    return HttpResponse('同城页面')

def hot_book(request, book_id):
    return HttpResponse('最热门文章：%d'%book_id)
~~~

模板文件：

~~~django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
    <style>
        li{
            list-style: none;
            display: inline-block;
            margin-right: 20px ;
        }
    </style>
</head>
<body>
    <ul>
        <li><a href="">首页</a></li>
        <li><a href="/article/book/">读书</a></li>  <!-- url硬编码的方式 -->
        <li><a href="{% url 'article:movie' %}">电影</a></li> <!-- url标签 -->
        <li><a href="{% url 'article:city' %}">同城</a></li>
        <li><a href="{% url 'article:hot_book' book_id=1 %}">最火文章</a></li> <!-- url标签 传递参数，这个参数需要于url映射的参数名一样，如果传递多个参数，则空格隔开 -->
    </ul>
</body>
</html>
~~~

![1584882199689](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584882199689.png)

![1584882209146](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584882209146.png)



#### 2.5 autoescape标签

`DTL`中默认已经开启了自动转义，会将哪些特殊字符进行转义，比如会将`<`转义成`&lt`等。

~~~python
def index(request):
    comment_info = '个人博客网站：<a href="http://www.qmpython.com"> 全民python</a>'
    context = {
        'info': comment_info
    }
    return render(request, 'index.html', context=context)
~~~

模板文件：

~~~html
 <p>个人博客网站：<a href="http://www.qmpython.com"> 全民python</a></p>
 <p>
      {{ info }}   <!-- DTL默认开启了自动转义 -->
 </p>
 <p>
     {% autoescape off %}   <!-- 关闭自动转义 -->
        {{ info }}
     {% endautoescape %}
 </p>
~~~

![1584883288269](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584883288269.png)



如果你不知道自己在干什么，最好是使用自动转义，这样网站才不容易出现XSS漏洞(比如有些网站进行评论的时候，发一些含恶意的js代码，如果不进行自动转义会进行渲染，而不会做成普通的字符串)，比如我个人博客的评论就处理了。如果是可信用的，可以关闭自动转义，比如我的文章内容是只允许我个人发表，是可信任的，所以我直接关闭。后面可以通过过滤器`safe`类似处理。



#### 2.6 verbatim标签

默认在DTL模板中是会去解析那些特殊字符的，比如`{% %}`和`{{ }}`等，如果我们在某个代码片段不想使用DTL的解析，那么我们可以将这段代码放在`verbatim`标签中。

~~~python
# 视图
def index(request):
    return render(request, 'index.html')

# 模板
<body>
    {% if msg %}
        {{ msg }}
    {% else %}
        没消息
    {% endif %}
    <br>
    {% verbatim %}
        {% if msg %}
            {{ msg }}
        {% endif %}
    {% endverbatim %}
</body>
~~~

![1584884085899](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584884085899.png)



这个主要是用在，有些前端模板语法layui和DTL的类似，这样就容易引起混淆，我们可以使用这个标签，来屏蔽掉DTL的解析，而让前端模板去解析。



#### 2.7 注释标签

我们在HTML中经常使用`<!-- 注释内容 -->`来进行注释，在Django模板中，我们还可以使用其他注释。

~~~django
{# 被注释的内容 #}： 将中间的内容注释掉。只能单行注释。
{% comment %}被注释的内容{% endcomment %}：可以多行注释。
~~~

这种注释，当我们查看网页元素的时候是看不到的，即没有被渲染，而`<!-- -->`还是可以看到原样字符串的。



### 3. 模板过滤器

在模板中，有时候需要对一些数据进行处理以后才能使用，一般在`Python`中我们是通过函数的形式来完成的，因为在`DTL`中，不支持函数的调用形式`()`，因此在模板中，则是通过过滤器来实现的，过滤器使用的是`|`来使用。语法：

~~~python
{{ 变量名|过滤器名:传给过滤器的参数}}
~~~



#### 3.1 常用过滤器

比如使用`date`过滤器。

~~~python
# 视图
from datetime import datetime

def index(request):
    now_time = datetime.now()
    context = {
        'now_time': now_time
    }
    return render(request, 'index.html', context=context)


# 模板文件
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
</head>
<body>
    {{ now_time }} <!-- 原始渲染 -->
    <br>
    {{ now_time | date:'Y-m-d H:i:s' }}  <!-- 通过过滤器，先将日期格式化，再渲染出-->
</body>
</html>
~~~

![1584971014392](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584971014392.png)

`date`时间过滤器格式：

| 格式字符 | 描述                   | 示例       |
| -------- | ---------------------- | ---------- |
| Y        | 4位数的年              | 1999       |
| y        | 2位数的年              | 99         |
| m        | 2位数的月              | 01，09     |
| n        | 1位数的月              | 1，9，12   |
| d        | 2位数的日              | 01，09，31 |
| j        | 1位数的日              | 1，9，31   |
| g        | 12小时制的一位数的小时 | 1，9，12   |
| G        | 24小时制的一位数小时   | 0，8，23   |
| h        | 12小时制的两位数的小时 | 01，09，12 |
| H        | 24小时制的两位数的小时 | 01，13，24 |
| i        | 分钟                   | 00-59      |
| s        | 秒                     | 00-59      |

`django`常用过滤器：

`add`：字符串相加，数字相加，列表相加，如果失败，将会返回一个空字符串。
`default`：提供一个默认值，在这个值被`django`认为是`False`的时候使用。比如：空字符串、`None、[]、{}`等。区别于`default_if_none`，这个只有在变量为`None`的时候才使用默认值。
`first`：返回列表中的第一个值。
`last`：返回列表中的最后一个值。
`date`：格式化日期和时间。
`time`：格式化时间。
`join`：跟python中的join一样的用法。
`length`：返回字符串或者是数组的长度。
`length_is`：字符串或者是数组的长度是否是指定的值。
`lower`：把所有字符串都编程小写。
`truncatechars`：根据后面给的参数，截断字符，如果超过了用`…`表示。`{{value|truncatechars:5}}`，如果`value`是等于`北京欢迎您`，那么输出的结果是`北京...`，为啥不是`北京欢迎您...`呢？因为三个点也占了三个字符，所以`北京` + 三个点的字符长度就是5。
`truncatewords`：类似`truncatechars`，不过不会切割`html`标签，`{{value|truncatewords:5}}`，如果`value`是等于`<p>北京欢迎您</p>`，那么输出的结果是`<p>北京...</p>`，
`capfirst`：首字母大写。
`slice`：切割列表。用法跟python中的切片操作是一样的，区间是前闭合后开放。
`striptags`：去掉所有的html标签。
`safe`：关闭变量的自动转义
`floatformat`：浮点数格式化。
更多可以查询官方文档： [https://yiyibooks.cn/xx/Django_1.11.6/ref/templates/builtins.html](https://links.jianshu.com/go?to=https%3A%2F%2Fyiyibooks.cn%2Fxx%2FDjango_1.11.6%2Fref%2Ftemplates%2Fbuiltins.html)
英文：[https://docs.djangoproject.com/en/1.11/ref/templates/builtins/](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.djangoproject.com%2Fen%2F1.11%2Fref%2Ftemplates%2Fbuiltins%2F)

过滤器总结：
1、作用：对变量进行过滤。在真正渲染出来之前，过滤器会根据功能处理好变量，然后得出结果后再替换掉原来的变量展示出来。
2、语法：`{{greeting|lower}}`，变量和过滤器中间使用管道符号”|”进行使用。
3、可以通过管道符号进行链式调用，比如实现一个功能，先把所有字符变成小写，把第一个字符转换成大写，如`{{message|lower|capfirst}}`

4、过滤器可以使用参数，在过滤器名称后面使用冒号”:”再加上参数，比如要把一个字符串中所有的空格去掉，则可以使用`cut`过滤器，代码如下`{{message|cut:" "}}`，冒号和参数之间不能有任何空格，一定要紧挨着。

#### 3.2 自定义过滤器

虽然`DTL`给我们内置了许多好用的过滤器，但是有些时候还是不能满足我们的需求，因此`Django`给我们提供了一个接口，可以让我们自定义过滤器，实现自己的需求。

步骤：

1、首先在某个应用`app`中，创建一个`python`包，叫做`templatetags`，注意名字不能改动，不然找不到。

2、在这个`templatetags`包下创建一个`python`文件用来存储过滤器。

3、在新建的`python`文件中，定义过滤器（也就是函数），这个函数的第一个参数永远是被过滤器的那个值，并且如果在使用过滤器的时候传递参数，那么还可以定义另外一个参数，但是过滤器最多只能有2个参数。

4、自定义完过滤器，要使用`django.template.Library.filter`进行注册。

5、过滤器所在`app`所在app需要在`settings.py`中进行注册。

6、在模板中使用`load`标签加载过滤器所在的`python`包。

7、在模板中使用自定义的过滤器。



我们来实现一个朋友圈，或者文章发表时间显示的过滤器：

~~~python
# 自定义过滤器 date_filter.py
from django import template

from datetime import datetime, timedelta

# 将注册类实例化为register对象
# register = template.Library() 创建一个全局register变量，它是用来注册你自定义标签和过滤器的，只有向系统注册过的tags，系统才认得你。
# register 不能做任何修改，一旦修改，该包就无法引用
register = template.Library()

# 使用装饰器注册自定义过滤器
@register.filter
def date_filter(value):
    # 判断一下，是不是时间日期类型
    if not isinstance(value, datetime):
        return value

    now = datetime.now() + timedelta(hours=8)  # 将UTC时间转为本地时间

    # total_seconds()是获取两个时间之间的总差
    timestamp = (now - value).total_seconds()

    if timestamp < 60:
        return '刚刚'
    elif 60 <= timestamp < 60*60:   # sec >= 60 and sec < 60*60
        mint = int(timestamp / 60)
        return '{}分钟前'.format(mint)
    elif 60*60 <= timestamp < 60*60*24:
        hour = int(timestamp / 60 / 60)
        return '{}小时前'.format(hour)
    elif 60*60*24 <= timestamp < 60*60*24*2:
        return '昨天'
    else:
        return timestamp.strftime('%Y-%m-%d %H:%M:%S')


# 视图views.py
from django.shortcuts import render, HttpResponse

from datetime import datetime

def index(request):
    context = {
        'public_time': datetime(year=2020, month=3, day=23, hour=23, minute=0, second=0)
    }
    print(context)
    return render(request, 'index.html', context=context)



# 模板文件
{% load date_filter %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
</head>
<body>
    {{ public_time|date_filter }}

</body>
</html>
~~~

![1584976654680](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1584976654680.png)



### 4. 模板结构化优化

#### 4.1 引入模板

有时候一些代码是在许多模板中都用到的，如果每次都重复的去拷贝代码那肯定不符合项目的规范，一般我们可以把这些重复性的代码抽取出来，就类似于`Python`的函数一样，以后想要使用这些代码的时候，就通过`include`包含进来，这个标签就是`include`。比如大多数网页都有顶部，中间，底部，可能进入任何页面顶部和底部都是一样的，只有中间部分内容不同，这个时候顶部和底部，我们就可以通过`include`包含起来。

![1585048505164](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585048505164.png)

~~~html
# 顶部，header.html
<header>
    <ul>
        <li>首页</li>
        <li>Python教程</li>
        <li>Python Web开发</li>
        <li>Python爬虫</li>
    </ul>
    <p>用户名:{{ username }}</p> <!-- 用于测试是否传参 -->
</header>

# 底部，footer.html
<footer>
    <div>©2019<a href="/">全民python</a>&nbsp;|&nbsp;
        <a href="http://beian.miit.gov.cn" rel="nofollow">粤ICP备18143893号</a>&nbsp;|&nbsp;
    </div>
</footer>

# 首页，index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文章首页</title>
    <style>
        div{
            margin: 10px;
        }
    </style>
</head>
<body>
  {% include 'header.html' with username='全民python' %}  <!-- 引入模板，并用with传递参数 -->
  <div>
    这是中间内容
  </div>
  {% include 'footer.html' %}
</body>
</html>
~~~

视图：

~~~python
def index(request):
    return render(request, 'index.html')
~~~

![1585060089888](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585060089888.png)



#### 4.2 模板继承

对于模板的重复利用，除了使用`include`标签引入，还有另外种方式来实现，那就是模板继承。模板继承类似于`Python`中的类，在父类中可以先定义好一些变量和方法，然后在子类中实现，模板继承也可以在父模板中先定义好一些子模板需要用到的代码，然后子模板直接继承就可以了，并且因为子模板肯定有自己的不同代码，因此可以在父模板中定义一个`block`接口，然后子模板再去实现。

我们将上面那个例子，使用模板继承改写下，将固定不变的顶部和底部，放在父模板中，中间变动的部分，子模板去实现。

~~~django
<!-- 父模板-->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}全民python{% endblock %}</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        header,footer{
            background: black;
            height: 60px;
            width: 100%;
            color: white;
            text-align: center;
            line-height: 40px;
        }
        footer{
            height: 90px;
            line-height: 90px;
            position: absolute;
            bottom: 0;
        }
        ul li{
            list-style: none;
            float: left;
            margin: 20px;
        }
        #loginbox{
            width: 400px;
            height: 200px;
            border: 1px solid red;
            margin: 90px auto;
            text-align: center;
        }
    </style>
</head>
<body>
    <header>
       <ul>
           <li>首页</li>
           <li>编程语言</li>
           <li>项目实战</li>
           <li>每日一学</li>
        </ul>
    </header>
    <div class="content">
        {% block content %}
            这是子模板自己实现的部分
        {% endblock %}
    </div>
    <footer >
        这是footer部分
    </footer>
</body>
</html>


<!-- 子模板-->
{% extends 'base.html' %}  <!-- extends必须是模板中的第一个出现的标签 -->
{% block title %}{{ block.super }}-登录界面{% endblock %} <!-- block.super继承父类的内容，否则会覆盖父类内容 -->
{% block content %}
    <div id="loginbox">
        <br/>
        <h2>登录界面</h2>
        <br/>
        <form>
            帐&nbsp;号：<input type="text">
            <br/>
            <br/>
            密&nbsp;码：<input type="password">
            <br/>
            <br/>
            <input type="submit" value="登录">
            &nbsp;&nbsp;&nbsp;&nbsp;
            <input type="button" value="注册">
        </form>
    </div>
{% endblock %}
~~~

视图函数：

~~~python
def index(request):
    return render(request, 'index.html')
~~~

效果展示：

![1585100431284](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585100431284.png)



总结：

1. 模板继承使用`extends`标签实现，通过使用`block`来给子模板开放接口；
2. `extends`必须是模板中第一个出现的标签；
3. 子模板中的内容必须放在父模板定义好的`block`中，否则将不会被渲染；
4. 如果在某个`block`中需要使用父模板的内容，那么可以使用`{{  block.super }}`来继承；
5. 子模板决定替换的`block`块，无须关注其它部分，没有定义的块即不替换，直接使用父模板的block块；
6. 除了在开始的地方定义名字，我们还可以在结束的时候定义名字，如`{% block title %}{% endblock %}`，这样在大型模板中显得尤其有用，能让我们快速看到`block`所包含的开始和结束。



### 5. 加载静态文件

在一个网页中，不仅仅只有一个`html`，还需要`css`样式，`js`脚本以及一些图片，因此在`DTL`中加载静态文件是一个必须要解决的问题，在`DTL`中，使用`static`标签来加载静态文件，要使用`static`标签，首先需要`{% load static %}`。

1、首先，`settings.py`文件中，确保`django.contrib.staticfiles`已经添加到`INSTALLED_APPS`中。

~~~python
INSTALLED_APPS = [
    ...
    'django.contrib.staticfiles',
    ...
]
~~~

2、`settings.py`文件中确保配置了`STATIC_URL`。

~~~python
# 静态文件存储，一般是我们的JS、css、系统的图片文件等。
# 这个“static”指访问静态文件，引入时用的别名
# 在浏览器中请求静态文件的url，	127.0.0.1/static/xxx.jpg
STATIC_URL = '/static/'  	
~~~

3、在已注册的应用`app`下创建一个静态文件夹名称`static`，然后再再这个`static`目录下创建于`app`同名的文件夹用于存放静态文件(避免不同应用`app`，静态文件名字一样，造成冲突)。

在应用`user`下新建`static`目录，再在`static`下新建user目录：

![1585106587510](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585106587510.png)

视图：

~~~python
# user应用下的视图views.py

def index(request):
    return render(request, 'user/index.html')
~~~

模板文件：

~~~django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户登录首页</title>
</head>
<body>
    <p>用户界面</p>
    <img src="/static/user/img/logo.png">  <!-- 如果配置文件是 STATIC_URL = '/abc/'，则这里url：/abc/user/img/logo.png-->
</body>
</html>
~~~

但是上面要写绝对路径不方便，那我们可以通过static标签来加载静态文件：

~~~django
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户登录首页</title>
</head>
<body>
    <p>用户界面</p>
{#    <img src="/static/user/img/logo.png">#}

    <img src="{% static '/user/img/logo.png' %}"> <!-- 通过static标签，django会去static目录下寻找 -->
</body>
</html>
~~~

![1585108344127](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585108344127.png)

4、如果有些静态文件和应用没什么太大的关系，我们可以在项目根目录下再创建`static`目录，这个时候我们还需要在`settings.py`文件配置添加`STATICFILES_DIRS`，以后`DTL`就会在这个列表的路径中查找静态文件。

~~~python
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]  # 这里指静态文件保存哪个目录下,这个“static”指目录名
~~~

然后再像上面一样加载静态文件使用，一般直接在根项目下新建`static`，然后在下面新建不同以应用名称命名的目录。

额外知识点：

1、如果不想在模板文件中每次都通过`{% load static %}`加载静态文件。

~~~python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
            'builtins': ['django.templatetags.static']  # 添加这一行，就会将static当作内置的标签，无需再手动load
        },
    },
]
~~~

