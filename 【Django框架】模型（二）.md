### 一、模型字段Field类型

在Django中，定义了一些`Field`来与数据库表中的字段进行映射，常用的字段类型。

| 类型                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| **AutoField**              | 一个自动增加的整数类型字段。通常你不需要自己编写它，Django会自动帮你添加字段：`id = models.AutoField(primary_key=True)`，这是一个自增字段，从1开始计数。如果你非要自己设置主键，那么请务必将字段设置为`primary_key=True`。Django在一个模型中只允许有一个自增字段，并且该字段必须为主键！ |
| `BigAutoField`             | (1.10新增)64位整数类型自增字段，数字范围更大，从1到9223372036854775807 |
| `BigIntegerField`          | 64位整数字段（看清楚，非自增），类似IntegerField ，-9223372036854775808 到9223372036854775807。在Django的模板表单里体现为一个textinput标签。 |
| **BooleanField**           | 布尔值类型。默认值是None。在HTML表单中体现为CheckboxInput标签。如果要接收null值，请使用NullBooleanField。 |
| **CharField**              | 字符串类型。必须指定字符串最大长度`max_length`，如果超过了254个字符，则不建议使用。默认的表单标签是input text。最常用的类型！ |
| CommaSeparatedIntegerField | 逗号分隔的整数类型。必须接收一个max_length参数。常用于表示较大的金额数目，例如1,000,000元。 |
| **DateField**              | `class DateField(auto_now=False, auto_now_add=False, **options)`日期类型。一个Python中的datetime.date的实例。在HTML中表现为TextInput标签。在admin后台中，Django会帮你自动添加一个JS的日历表和一个“Today”快捷方式，以及附加的日期合法性验证。两个重要参数：（参数互斥，不能共存） `auto_now`:每当对象被保存时将字段设为当前日期，常用于保存最后修改时间。`auto_now_add`：每当对象被创建时，设为当前日期，常用于保存创建日期(注意，它是不可修改的)。设置上面两个参数就相当于给field添加了`editable=False`和`blank=True`属性。如果想具有修改属性，请用default参数。例子：`pub_time = models.DateField(auto_now_add=True)`，自动添加发布时间。 |
| DateTimeField              | 日期时间类型。Python的datetime.datetime的实例。与DateField相比就是多了小时、分和秒的显示，其它功能、参数、用法、默认值等等都一样。 |
| TimeField                  | 时间字段，Python中datetime.time的实例。接收同DateField一样的参数，只作用于小时、分和秒。 |
| DecimalField               | 固定精度的十进制小数。相当于Python的Decimal实例，必须提供两个指定的参数！参数`max_digits`：最大的位数，必须大于或等于小数点位数 。`decimal_places`：小数点位数，精度。 当`localize=False`时，它在HTML表现为NumberInput标签，否则是text类型。例子：储存最大不超过999，带有2位小数位精度的数，定义如下：`models.DecimalField(..., max_digits=5, decimal_places=2)`。 |
| DurationField              | 持续时间类型。存储一定期间的时间长度。类似Python中的timedelta。在不同的数据库实现中有不同的表示方法。常用于进行时间之间的加减运算。但是小心了，这里有坑，PostgreSQL等数据库之间有兼容性问题！ |
| **EmailField**             | 邮箱类型，默认max_length最大长度254位。使用这个字段的好处是，可以使用DJango内置的EmailValidator进行邮箱地址合法性验证。 |
| **FileField**              | `class FileField(upload_to=None, max_length=100, **options)`上传文件类型，后面单独介绍。 |
| FilePathField              | 文件路径类型，后面单独介绍                                   |
| FloatField                 | 浮点数类型，参考整数类型                                     |
| **ImageField**             | 图像类型，后面单独介绍。                                     |
| **IntegerField**           | 整数类型，最常用的字段之一。取值范围-2147483648到2147483647。在HTML中表现为NumberInput标签。 |
| **GenericIPAddressField**  | `class GenericIPAddressField(protocol='both', unpack_ipv4=False, **options)[source]`,IPV4或者IPV6地址，字符串形式，例如`192.0.2.30`或者`2a02:42fe::4`在HTML中表现为TextInput标签。参数`protocol`默认值为‘both’，可选‘IPv4’或者‘IPv6’，表示你的IP地址类型。 |
| NullBooleanField           | 类似布尔字段，只不过额外允许`NULL`作为选项之一。             |
| PositiveIntegerField       | 正整数字段，包含0,最大2147483647。                           |
| PositiveSmallIntegerField  | 较小的正整数字段，从0到32767。                               |
| SlugField                  | slug是一个新闻行业的术语。一个slug就是一个某种东西的简短标签，包含字母、数字、下划线或者连接线，通常用于URLs中。可以设置max_length参数，默认为50。 |
| SmallIntegerField          | 小整数，包含-32768到32767。                                  |
| **TextField**              | 大量文本内容，在HTML中表现为Textarea标签，最常用的字段类型之一！如果你为它设置一个max_length参数，那么在前端页面中会受到输入字符数量限制，然而在模型和数据库层面却不受影响。只有CharField才能同时作用于两者。 |
| **URLField**               | 一个用于保存URL地址的字符串类型，默认最大长度200。           |
| **UUIDField**              | 用于保存通用唯一识别码（Universally Unique Identifier）的字段。使用Python的UUID类。在PostgreSQL数据库中保存为uuid类型，其它数据库中为char(32)。这个字段是自增主键的最佳替代品，后面有例子展示。 |





关于时间问题：

~~~python
TIME_ZONE = 'Asia/Shanghai'  # 用于存放本地时区信息，默认值为UTC，意思为采用国际标准时间“格林尼治时间”。中国处于东八区，官方文档上有两个取值“Asia/Shanghai”和“Asia/Chongqing”(没有北京).

USE_TZ = True  # 修改时区确认,默认是Ture，时间是utc时间，由于我们要用本地时间，所用手动修改为false!

# 如果修改设置为USE_TZ=True与TIME_ZONE = 'Asia/Shanghai'，用datetime.datetime.now()获取的时间由于不带时区，
# django会把这个时间当成Asia/Shanghai时间，即东八区时间，然后django会把这个时间转成带时区UTC时间存储到数据库中去，
# 而读的时候直接按UTC时间读出来，这就是网上很多人遇到的存储到数据库中的时间比本地时间会小8个小时的原因。
# 如果设置了USE_TZ=True之后，model里面认为DateTimeField使用UTC时间（带时区的时间），
# 这时用datetime.datetime.now()获取的时间是不带时区的就会报这个问题。
~~~



### 二、模型字段Field常用参数

![img](https://upload-images.jianshu.io/upload_images/9286065-0fba7c689a1c0544.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)



* `null`与`blank`：
  **blank：**针对表单的，如果`blank=True`，表示表单填写该字段的时候可以不填，默认不允许为空。只是在form表单验证时可以为空，而在数据库上存储的是一个空字符串。

  **null：**是针对数据库级别的，如果`null`=`True`，表示数据库的该字段可以为空，默认`false`不允许为空。日期型、时间型和数字型字段不接受空字符串，所以设置IntegerField，DateTimeField型字段可以为空时，需要将blank，null均设为True。

* `auto_now` 与`auto_now_add`区别：

  `auto_now_add`： 只有第一次才会生效，比如可以用于文章创建时间。

  `auto_now`： 每一次修改保存对象时都会将当前时间更新进去，只有调用Model.save()时更新，在以其他方式（例如 `QuerySet.update()`）更新其他字段时，不会更新该字段，但您可以在此类更新中为字段指定自定义值。可用于文章修改时间。





### 三、模型元选项

每个`model`都可以定义一个`Meta`类，使用内部的`class Meta `定义模型的元数据，这个类中可以定义一些关于你的配置，`Meta`是一个`model`内部的类。

模型元数据是`任何不是字段的数据`，比如排序选项（ordering），数据库表名（db_table）或者人类可读的单复数名称（verbose_name 和verbose_name_plural）。在模型中添加class Meta是完全可选的，所有选项都不是必须的。

~~~python

class Article(models.Model):
    id = models.AutoField(primary_key=True)  
    title = models.CharField(verbose_name='标题', max_length=20, null=False)
    content = models.CharField(verbose_name='内容', max_length=150, null=False)
    comment = models.CharField(verbose_name='评论', max_length=50)
    reply = models.CharField(verbose_name='回复', max_length=50)
    status = models.IntegerField(verbose_name='状态', default=1)

    class Meta:
        db_table = 'tb_article'
        order = ['-id']
        verbose_name = '文章'
        verbose_name_plural = verbose_name
~~~

`db_table`：`string`，在数据库中的表名，否则`Django`自动生成为`app名字_类名`。
`managed`： `bool`， 默认值为`True`，这意味着`Django`可以使用`syncdb`和`reset`命令来创建或移除对应的数据库。
`ordering`: 数组，默认排序规则，每个字符串是一个字段名，前面带有可选的`-`前缀表示倒序。前面没有`-`的字段表示正序。使用`?`来表示随机排序。
`verbose_name`: `string model`对象的描述，用于`admin`后台管理。
`verbose_name_plural`: `string` 复数时的描述。



### 四、外键

#### 1. ORM外键初识

接下来我们学习如何在`Django`中使用外键，比如博客，一个分类`Category`下有多篇文章`Article`，而一篇文章只属于一个分类。

~~~python
from django.db import models

class Category(models.Model):
    name = models.CharField(verbose_name='分类名称', max_length=100)

    class Meta:
        db_table = 'tb_category'

        
class Article(models.Model):
    title = models.CharField(verbose_name='标题', max_length=20)
    content = models.CharField(verbose_name='内容', max_length=150)
    # 外键使用ForeignKey，在多的一边定义。第一个参数参数，引用的哪个模型，第二个参数，使用外键引用的模型数据被删除，该字段如何处理。
    category = models.ForeignKey("Category", on_delete=models.CASCADE)

    class Meta:
        db_table = 'tb_article'
~~~

我们进行数据迁移映射之后：

![1585837651423](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585837651423.png)

在`tb_article`表中生成了外键字段`category_id`。

我们来看下如何操作：

~~~python
from django.shortcuts import render, HttpResponse
from .models import Category, Article

def index(request):
    category = Category(name='Python基础')
    category.save()  # 需要先保存，才能给后面的赋值。

    article = Article(title='python数据类型')
    article.category = category    # 直接将模型对象赋值给外键的模型
    article.save()

    return HttpResponse('success')
~~~

![1585902195317](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585902195317.png)

添加数据成功。

如果模型的外键引用是本身自己，那么参数可为`self`或者这个模型的名字，例如在评论中就用到。



#### 2. ORM外键删除操作

如果一个模型使用了外键，那么在对方模型(**主表**)被删除后，（**子表**）该进行什么样的操作，可以通过`on_delete`来指定，可以指定的类型如下：

1）`CASCADE`：级联操作，如果外键对应的那条(**主表**)数据被删除，那么这条(**子表**)数据也会被删除。

~~~python
def delete_views(request):
    category = Category.objects.get(pk=1)
    category.delete()

    return HttpResponse('delete success')
~~~

![1585903290179](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1585903290179.png)



我们在视图中只删除`category`对象，没有手动删除`article`，因为前面模型定义使用的是`CASCADE`，所以最后级联删除了。

2）`PROTECT`：受保护，如果删除，将会抛出一个`ProtectedError`错误。主表，子表都删除不了。

3）`SET_NULL`：设置为空，如果外键对应的那条(**主表**)数据被删除，那么本条(**子表**)数据上就将这个字段设置为空。前提是要指定这个字段可以为空（**只有在外键null为True的情况下才可以使用**）。

4）`SET_DEFAULT`：设置默认值，如果外键对应的那条(**主表**)数据被删除，那么本条(**子表**)数据上就将这个字段设置为默认值，前提是要指定这个字段一个默认值（**这个只有在外键那个字段设置了default参数才可以使用**）。

5）`SET()`：如果外键对应的那条(**主表**)数据被删除，那么将会获取`SET`函数中的值来作为这个外键的值。`SET`函数可以接收一个可以调用的对象（比如函数或者方法），如果是可以调用的对象，那么会将这个对象调用后的结果返回去。

6）`DO_NOTHING`：不采取任何行为，一切全看数据库级别的约束。

**以上这些选项只是Django级别的，数据库级别依旧是RESTRICT!**



### 五、表关系

表之间的关系都是通过外键来进行关联的，而表之间的关系，无非就是三种关系：一对一、一对多(多对一)、多对多等，以下将讨论以下三种关系的应用场景及实现方式。

#### 1. 多对一

**应用场景：**比如文章和分类之间的关系，一篇文章只能在一个分类下，但是一个分类下可以有多篇文章，文章和分类之间就是典型的多对一关系。

**实现方式：**一对多或多对一，都是通过`ForeignKey`来实现的，还是以分类和文章的案例进行讲解。

~~~python
class Category(models.Model):
    name = models.CharField(verbose_name='分类名称', max_length=100)

    class Meta:
        db_table = 'tb_category'

class Article(models.Model):
    title = models.CharField(verbose_name='标题', max_length=20)
    content = models.CharField(verbose_name='内容', max_length=150)
    category = models.ForeignKey("Category", on_delete=models.CASCADE)  # ForeignKey定义在多的一方。

    class Meta:
        db_table = 'tb_article'          
~~~

我们可以看到在`tb_article`表中多出了一个`category_id`字段，这个是引用了`tb_category`的主键，因为我们在文章模型中定义了外键，所以`Django`生成数据库文件的时候就会新增这个字段。

![1590150992420](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1590150992420.png)



**增加：**那么以后，增加文章的时候，可以指定分类了：

~~~python
def one_to_many_view(request):
    article = Article(title='Python快速入门', content='本篇将针对Python做一个简单入门，帮助小白迅速了解Python。')
    # 1.先获取主表中的对象
    category = Category.objects.get(pk=2)
    # 2.再赋给子表中的外键
    article.category = category
    # 3.最后保存
    article.save()

    return HttpResponse('增加成功')
~~~



![1586325127256](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1586325127256.png)



**访问：**查询某个文章所属分类（多查一，从子表查主表）：

~~~python
    # 查询某篇文章属于哪个分类，可以理解为通过子表查询
    # 1. 先获取子表对象
    article = Article.objects.get(pk=5)
    # 2. 通过子表对象.外键属性来获取主表对象
    category = article.category

    print(category.name)
~~~

查询某个分类下所有文章（一查多，主查从）：

~~~python
    # 获取某一个分类下所有文章，可以理解为通过主表查询子表，一查多，这就需要用到反查
    category = Category.objects.get(pk=2)
    print(category)

    articles = category.article_set.all() 
    for article in articles:
        print(article)
~~~

外键反向访问：如果`category`对象想要访问所有引用了它的`article`对象，因为没有外键属性，不能采用`.`方式访问，默认可以通过**模型名小写_set**进行访问(上面默认以`article_set`访问)，可以在从表定义时设置`related_name `参数来覆盖默认 的名称。通过主表来查询子表信息用到反向查询（返回的是管理器，需要`all`之类返回`QuerySet`）。

那我们改成：

~~~python
class Category(models.Model):
    name = models.CharField(verbose_name='分类名称', max_length=100)

    class Meta:
        db_table = 'tb_category'

    def __str__(self):
        return self.name
    
class Article(models.Model):
    title = models.CharField(verbose_name='标题', max_length=20)
    content = models.CharField(verbose_name='内容', max_length=150)
    category = models.ForeignKey("Category", on_delete=models.CASCADE, related_name='articles')  # 反查别名

    class Meta:
        db_table = 'tb_article'

    def __str__(self):
        return self.title      

    # 获取某一个分类下所有文章，可以理解为通过主表查询子表，一查多，这就需要用到反查
    category = Category.objects.get(pk=2)
    print(category)

    # articles = category.article_set.all()
    articles = category.articles.all()   # 通过articles参数设置的来
    for article in articles:
        print(article)

~~~

![1586400154806](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1586400154806.png)



#### 2. 一对一

**应用场景：**比如一个用户表和一个用户信息表，在实际项目中，可能需要保存用户的许多信息，但是有些信息是不经常用的，如果把所有信息都存放到一张表中可能会影响查询效率，因此可以把用户的一些不常用的信息存放到另外一张表中，用户表和用户信息表就是典型的一对一了。

**实现方式：**一对一通过`OneToOneField`来实现一对一操作的，还是以用户和用户其他信息进行详解。

~~~python

class Account(models.Model):
    username = models.CharField(max_length=16)
    password = models.CharField(max_length=20)

    class Meta:
        db_table = 'tb_account'

    def __str__(self):
        return self.username


class AccountExtension(models.Model):
    birthday = models.DateTimeField(null=True)
    school = models.CharField(max_length=40, null=True)
    user = models.OneToOneField('Account', on_delete=models.CASCADE, related_name='accountExtension')


    class Meta:
        db_table = 'tb_account_ext'

    def __str__(self):
        return self.user.username
~~~

**增加：**那么以后，增加文章的时候，可以指定分类了：

~~~python
def one_to_one_view(request):
    account = Account(username='张三', password='123456')
    account.save()

    account_ext = AccountExtension(school="清华")
    account_ext.user = account
    account_ext.save()

    return HttpResponse('增加成功')
~~~

![1587391305793](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1587391305793.png)



**访问：**查询某个用户的学校（从主表查子表）：

~~~python
def show_birthday_view(request, user_id):
    account = Account.objects.get(pk=user_id)
    school = account.accountExtension.school  # 通过accountExtension参数设置的来
    return HttpResponse('用户【{}】在【{}】读书'.format(user_id, school))
~~~



#### 3. 多对多

**应用场景：**比如文章和标签的关系，一篇文章可能有多个标签，一个标签可以被多个文章所引用，文章和标签的关系就是多对多的关系。

**实现方式：**多对多，通过`ManyToManyField`来实现的，以文章和标签案例进行讲解。

~~~python
class Article(models.Model):
    title = models.CharField(verbose_name='标题', max_length=20)
    content = models.CharField(verbose_name='内容', max_length=150)
    category = models.ForeignKey("Category", on_delete=models.CASCADE, related_name='articles')  # 多对一关系
    tags = models.ManyToManyField('Tag', related_name='articles')   # 多对多关系

    class Meta:
        db_table = 'tb_article'

    def __str__(self):
        return self.title


class Tag(models.Model):
    name = models.CharField(verbose_name='标签名称', max_length=20, unique=True)

    class Meta:
        db_table = 'tb_tag'

    def __str__(self):
        return self.name
~~~

在数据库层面，实际上`Django`是为这种多对多建立了一个中间表：![1590151231092](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1590151231092.png)

这个中间表分别定义了两个外键，引用到`tb_article`和`tb_tag`两张表的主键：
![1590151277626](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1590151277626.png)



**增加：**多对多数据添加需要先保存子表对象，然后再`add`进行添加主类对象。

如发表一篇文章的时候，可以指定标签了：

~~~python
def many_to_many_view(request):
    # 增加文章的时候，可以指定分类了
    # 1.创建一篇文章
    article = Article(title='最近，火遍朋友圈的地摊经济', content='相信，最近朋友圈都在讨论各种有关摆地摊的话题，你是否有种冲动也去试下呢？', category_id=4)
    # 保存文章
    article.save()

    # 2.给文章指定标签
    # 可以获取已存在的标签，或者同时新建标签(需要先保存)
    tag1 = Tag(name='摆地摊')
    tag1.save()
    tag2 = Tag(name='朋友圈')
    tag2.save()

    # 3.再将标签添加到文章中
    article.tags.add(tag1, tag2)  # 添加最终是添加到中间表中去，一次性可添加一个或多个

    return HttpResponse('增加成功')
~~~

`![1591623485160](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1591623485160.png)



**访问：**查询某个文章所属标签（从子表查主表）：

~~~python
    # 获取某篇文章
    article = Article.objects.get(pk=9)  # <Article: 最近，火遍朋友圈的地摊经济>
    # 文章属于的标签
    tags = article.tags.all()
    for tag in tags:
        print(tag.name) 
        
        
# ===》    
    # 摆地摊
    # 朋友圈
~~~

查询某个标签下所有文章（主表查子表），需要用到反查：

~~~python
    tag = Tag.objects.get(pk=4)  # <Tag: 快速入门>

    # 使用反查，前面设置了related_name，我通过她来进行反查
    articles = tag.articles.all()  # <QuerySet [<Article: MySQL快速入门>]>
    for article in articles:        
        print(article.title)  # MySQL快速入门
~~~



**删除：**将某篇文章的标签去掉：

```python
    # 获取某篇文章
    article = Article.objects.get(pk=9)  # <Article: 最近，火遍朋友圈的地摊经济>
    # 获取文章的标签
    tags = article.tags.all()  # <QuerySet [<Tag: 摆地摊>, <Tag: 朋友圈>]>
    # 删除对应标签，可以一个标签对象，也可以多个对象
    article.tags.remove(tags[0], tags[1])
    print(article.tags.all())    # <QuerySet []>
```

![1592055504168](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1592055504168.png)





备注：有时候测试，我们不一定都要写个对应视图，绑定相关URL，我们可以使用

![1592054876205](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1592054876205.png)



**ForeignKey Manager ** 还有如下一些方法：

**add(obj1, obj2, ...)**，将某个特定的model对象添加到被关联对象集合中。

~~~python
# 前面我们 增加文章的时候，指定分类了

# 我们现在要将某篇文章添加到某个分类，则可以使用add方法
category = Category.objects.get(pk=2)
article_1 =  Article.objects.get(pk=6)
article_2 =  Article.objects.get(pk=8)

category.articles.add(article_1, article_2)  # articles是在Article模型类中定义外键时通过related_name设置的。
~~~











![1592142036338](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1592142036338.png)



![1592223887502](C:\Users\44801\AppData\Roaming\Typora\typora-user-images\1592223887502.png)





**create(\*\*kwargs)**

创建并保存一个新对象，然后将这个对象放在关联的对象集返回新创建的对象。

**remove(obj, obj2, ...)**

从关联的对象集中删除指定的模型对象。

**clear()**

从关联的对象集中删除所有的对象





<https://cloud.tencent.com/developer/article/1465936>





















