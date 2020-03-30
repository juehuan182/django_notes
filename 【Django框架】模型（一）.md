### 一、MYSQL安装

我们这里使用`MYSQL`数据库，安装使用详情见  [Centos7中安装Mysql](https://www.qmpython.com/articles/detail-37.html)。PS：我是在`Centos7`下安装的。



### 二、MYSQL驱动程序

我们使用`Django`来操作`MYSQL`，实际上底层还是通过`Python`来操作，因此我们想要用`Django`来操作`MYSQL`，首先还是需要安装一个驱动程序，在`Python3`中，驱动程序有多种选择，常见驱动介绍：

1、`MySQL-python`：也就是`MySQLdb`，是对C语言操作`MySQLdb`数据库的一个简单封装。遵循了`Python DB API v2`，但是只支持`Python2`，目前还不支持`Python3`。

2、`mysqlclient`：是`MySQL-python`的另外一个分支，支持`Python3`并且修复了一些`bug`。

3、`pymysql`：纯`Python`实现的一个驱动，因为是纯`Python`编写的，因此执行效率不如上面2个，并且也因为是纯`Python`编写的，因此可以和`Python`无缝连接。

4、`MySQL Connector/Python`：`MySQL`官方退出的使用纯`Python`连接`MySQL`的驱动，因为是纯`Python`开发的，效率不高。

这里我们就使用`mysqlclient`来操作。安装非常简单，只需要：

~~~bash
# 进入虚拟环境
workon django_project
# 安装mysqlclient
pip install mysqlclient
~~~

安装报错：

![1585194761381](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585194761381.png)

如果报错，则需要安装依赖，即可

~~~bash
# 安装依赖
yum install mysql-devel
# 然后再执行 
pip install mysqlclient
~~~



### 三、操作数据库

#### 1. 创建数据库

~~~mysql
create database django_db charset=utf8;   -- 创建数据库

mysql> show databases;		-- 查看所有数据库
+--------------------+
| Database           |
+--------------------+
| information_schema |
| django_db          |
| mysql              |
| mytest             |
| performance_schema |
| qmpython           |
| sys                |
+--------------------+
7 rows in set (0.00 sec)

~~~



#### 2. 配置连接数据库

在操作数据库之前，首先要连接数据库，配置在`settings.py`文件中，默认使用的是自带的`sqlite`轻量级数据库：

~~~python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

~~~

一般我们自己学习可以使用这个，手机上也使用这个数据库，但是实际大型开发中，用的最多的还是`mysql`数据库，我们需要将配置修改为连接`mysql`数据库的。

~~~python
DATABASES = {
    'default': {
        # 数据库引擎（sqlite, mysql, oracle等）
        'ENGINE': 'django.db.backends.mysql',
        # 数据库库的名字
        'NAME': 'django_db',
        # 数据库用户名
        'USER': 'root',
        # 数据库密码
        'PASSWORD': '1234@root', 
        # 数据库的主机地址
        'HOST': '127.0.0.1',  # 你的数据地址，localhost代表本地
        # 端口，数据库的默认端口一般是3306，如果就算采用虚拟机NAT方式，此端口是指虚拟机上mysql端口，
        # 因为连接数据库前用ssh连接了，后面操作就相当于在虚拟机环境本地连接
        # 数据库的端口号，默认3306
        'PORT': '3306'
    }
}
~~~



#### 3. 原生SQL语句操作

在Django中操作数据库有2种方式，第一种方式就是使用原生`sql`语句操作，第二种就是使用`ORM`模型来操作，我们先学习使用原生`sql`来操作。

使用原生sql语句操作其实就是使用`python db api`的接口来操作。

~~~python
from django.db import connection

# 获取游标对象
cursor = connection.cursor()

# 拿到游标对象后执行sql语句
cursor.execute('select * from test')

# 获取所有的数据
rows = cursor.fetchall()

# 遍历查询到的数据
for row in rows:
    print(row)
~~~



`cursor`对象常用接口：
1、`description`：如果`cursor`执行了查询的sql代码，那么读取`cursor.description`属性的时候，将返回一个列表，这个列表中装的是元组。

2、`rowcount`：执行`sql`语句后受影响的行数。

3、`close`：关闭游标，关闭游标以后就再也不能使用了，否则抛出异常。

4、`execute(sql, parameters)`：执行某个sql语句，如果在执行sql语句的时候还需要传递参数，那么可以传给`parameters`参数。

5、`fetchone`：在执行了查询操作以后，获取第一条数据。

6、`fetchmany(size)`：在执行查询操作以后，获取多条数据，具体是多少条要看传的`size`参数，如果不传`size`参数，那么默认是获取第一条数据。

7、`fetchall`：获取所有满足`sql`语句的数据。



详细的请自行查询相关信息，这里我们重点学习`ORM`模型。



### 四、ORM介绍

随着项目越来越大，采用原生`SQL`的方式在代码中会出现大量的`SQL`语句，随之问题也出现了：
1、`sql`语句重复利用率不高，越复杂的`sql`语句条件越多，代码越长，会出现很多类似的`sql`语句。

2、很多`sql`语句是在业务逻辑中拼出来的，如果数据库要更改，就要去一个个修改这些逻辑，这很容易漏掉。

3、原生`sql`容易造成安全隐患，比如`sql`注入。

我们就可以通过`ORM`来。

`ORM`，全称`Object Relation Mapping`，即`对象关系映射`，通过操作对象的方式去操作数据库，而不用再写原生`SQL`语句。通过把表映射成类，把行作实例，把字段作为属性，`ORM`在执行对象操作的时候最终还是会把对应的操作转换为数据库原生`SQL`语句。



使用`ORM`有许多优点：

不管后端业务逻辑交互怎么样，最终的数据是要储存在数据库的，因为目前市面上的数据库比较多比如`SqlServer`、`oracle`、`mysql`等等，但是如果项目可能根据需要，使用不同的数据库，因为每个数据库各自语言有差异：

![1585210082233](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585210082233.png)

那是不是针对每个数据库再重写一次`SQL`，还是该怎么办呢？

![1585210684293](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585210684293.png)



对于复杂的查询，如果`ORM`无法表达，可以使用原生`SQL`。



### 五、Django中使用ORM

#### 1. 编写模型model

在每个`app`中都有对应的`models.py`文件，我们可以在里面定义需要的`models`。

```python
from django.db import models

# Create your models here.
class Article(models.Model): #  继承了models.Model或者它的子类
    # 属性=models.字段类型(选项)
    id = models.AutoField(primary_key=True)  # 如果没有添加主键，django会默认添加一个自增长的id主键
    title = models.CharField(verbose_name='标题', max_length=20, null=False) # 第一参数为备注
    content = models.CharField(verbose_name='内容', max_length=150, null=False)
    comment = models.CharField(verbose_name='评论', max_length=50)
    reply = models.CharField(verbose_name='回复', max_length=50)
    status = models.IntegerField(verbose_name='状态', default=1)
```

上面定义的模型就对应到我们后面数据库表对应关系：

```python
类（class）  --> 数据库的表（table）
属性（attribute） --> 字段（field）
记录（record，行数据）--> 对象（object）
```

#### 2. 生成迁移文件

##### 2.1 命令方式

```bash
# 1. 进入虚拟环境
workon django_project
# 2. 切换项目根目录下，即与manage.py文件同目录
cd django_project/
# 3. 执行命令
python manage.py makemigrations  # 将模型生成迁移文件

# 4.可以看到生成了文件：
Migrations for 'article':
  article/migrations/0001_initial.py
    - Create model Article   
```

应用`app`的`migrations`目录下多了个迁移文件：

![1585278933892](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585278933892.png)

可能`pycharm`没有这个`py`文件，需要从远程服务器`download`下来。

##### 2.2 pycharm中执行

类似之前创建应用`app`一样：

![1585279050387](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585279050387.png)

![1585279093243](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585279093243.png)

后面可以指定某个app名字，这样之迁移特定的应用，不用全部迁移。



**注意：**

使用`makemigrations` 命令需要事项：
1、一但将生成的migrations文件通过migrate命令同步至数据库以后，被操作的migrations文件不能再操作
2、如何避免上面的情况：
（1）不要对已经生成好的migrations文件做任何操作
（2）所有对数据库的操作同一个人做，并保留好一套migrations文件
（3）不要直接在数据库中修改任何操作，比如修改字段，添加字段，删除
ps: 一套对应生成的migrations文件与数据库需保存一致，否则系统会报错！！！
ps: 需要对数据库表的操作都需要models.py,然后走对应的migrate流程，不能直接在数据库中修改数据字段等等；
如果我之前通过`model`创建过表之类的，也有数据，后期因业务需要对现存表进行字段扩充或者条件约束，如果对原有`model`修改，那么原有的`migrations`文件不能变动，只需要重新 `python manage.py makemigrations`，它会与之前文件比较生成另外的文件的，然后migrate命令同步到数据库，不影响历史数据的。

#### 3. 迁移文件映射到数据库

```bash
 python manage.py migrate   # 将迁移文件真正映射到数据库
```

![1585279566440](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585279566440.png)

我们再来看数据库中是否生成对应数据表：

![1585279863184](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585279863184.png)



成功生成！

#### 4. ORM基本增删改查操作

##### 4.1 新增数据

```python
def index(request):
    # 新增数据
    # 方式1：
    article1 = Article()
    article1.title = '方式1-使用django新增一条数据'
    article1.content = '方式1-可以使用django的save方法持久化一条数据'
    article1.comment = '方式1-文章总结的到位，点赞'
    article1.reply = '方式1-谢谢，互相学习'
    article1.save()  # 使用save方法保存

    # 方式2：简写方式
    article2 = Article(title="方式2-使用django新增一条数据", content='方式2-可以使用django的save方法持久化一条数据',
                      comment='方式2-文章总结的到位，点赞', reply='方式2-谢谢，互相学习')
    article2.save()  # 使用save方法保存

    # 方式3：  create方式新增数据
    article3 = Article.objects.create(title="方式3-使用django新增一条数据", content='方式3-可以使用django的save方法持久化一条数据')

    return render(request, 'index.html')
```

查看表数据：

![1585281479830](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585281479830.png)

上面我们用了三种方式新增数据。

方式1：一个个字段来复制，最后使用`save()`方法保存。

方式2：使用关键字参数将方式1简化了，创建了一个对象，但还没保存到数据库，最后使用`save()`方法才保存到数据库。

方式3：使用`Article.objects.create`直接创建对象并保存了到数据库了。

##### 4.2 查询数据

```python
def index(request):
    # 查询数据
    # 查询单个数据
    # 通过get方式返回的数据，它只会返回一个对象
    # 如果通过条件返回的数据有多条或者找不到，都会报错
    # 1. 根据主键进行查找
    article1 = Article.objects.get(pk=3) # 在django 的ORM查询中，数据库的主键可以用PK代替， 官方推荐使用pk     
    print(article1) # <Article:(3,方式1-使用django新增一条数据,1)>

    # 2.根据条件查询
    article2 = Article.objects.filter(title='方式1-使用django新增一条数据')
    print(article2)  # <QuerySet [<Article: <Article:(3,方式1-使用django新增一条数据,1)>>]>，返回QuerySet列表
    print(article2.first())   # 可以通过first获取第一个，last获取最后一个，列表索引方式获取某一个，<Article:(3,方式1-使用django新增一条数据,1)>


    # 3.使用all()方法获取所有
    article3 = Article.objects.all()
    print(article3)  # <QuerySet [<Article: <Article:(3,方式1-使用django新增一条数据,1)>>, <Article: <Article:(4,方式2-使用django新增一条数据,1)>>, <Article: <Article:(5,方式3-使用django新增一条数据,1)>>]>


    return render(request, 'index.html')


# 输出：Article object (3)， 如果方便查看信息，我们去模型类中重写__str__方法
class Article(models.Model): 
	....
    
    def __str__(self):

        return '<Article:({id},{title},{status})>'.format(id=self.id, title=self.title, status=self.status)
```

##### 4.3 修改数据

```python
def index(request):
    # 单个修改
    article = Article.objects.get(pk=3)  # 取出pk=3的对象，然后修改其状态，再保存
    article.status = 2
    article.save()  # 实际上save方法如果当前实例已经存在于数据库中，它就会当作一个update操作

    # 批量修改
    # update article_article set status=6 where status=1
    Article.objects.filter(status=1).update(status=6) 

    return render(request, 'index.html')
```

![1585284658005](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585284658005.png)

##### 4.4 删除数据

```python
def index(request):
    # 删除数据
    # 删除单个， 如果数据不存在则报错
    Article.objects.get(id=3).delete()  # 等同于 delete from article_article where id=3

    # 批量删除
    Article.objects.filter(status=6).delete()  # 等同于 delete from article_article where status=6

    return render(request, 'index.html')
```

![1585284985351](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585284985351.png)





以上基本的增删改查操作，后续学习更多的操作。











