---
layout: post
title: "Python开发之路系列：Flask RESTful的接口开发"
date: 2018-04-02
excerpt: "在Flask里如何进行RESTful接口的开发."
tags: [Python,Flask，RESTful]
slug: py-starter-flask-restful
category: Python
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522674068150&di=e4139ae45a707ca1502db47579081515&imgtype=0&src=http%3A%2F%2Fs11.sinaimg.cn%2Flarge%2F001l8XD7zy778aQrf4eea%26690
---

**这篇文章我们来看看在Flask里是如何进行RESTful接口开发的**

按照我个人的理解，RESTful的核心价值再与它的规范性。
RESTful接口是面向资源的， 而不是面向动作。

比如一个查书的接口，如果是面向动作的风格，可能是这样的:
/getBook?name=?...
这很好，一目了然。
但是在增加书，修改书，删除书就是这样了：
/addBook
/editBook
/deleteBook
和你对接的前端同事也许会记住这些名字，但是也有可能记错
比如 /deleteBook 记成 /delBook
如果记性好，也没关系，才四个接口。
但是如果对数学书写一些接口呢？ 烹饪书呢？你可能会看到这些接口：
```
/getCookBook
/get-cook-book
/getMathBook
/get-math-book
/getMathematicsBook
/addCookBook
/addRecipe
/edit-a-cook-book
/search-meth-book
/delete-a-math-book
/delmathbook
......  // 无尽的痛苦
```
或者是由于某些原因后台可能会改这些接口名，导致前后端不一致，增加沟通成本，浪费时间
随着资源增多（除了书你还有其他资源），接口数量成倍增长，如果没有一种统一风格，可以想象接口是多么混乱。
而使用了RESTful风格接口，接口应该是这样的：
```
/api/v1/books/math/       [GET,PUT,POST,DELETE]
/api/v1/books/cook/       [GET,PUT,POST,DELETE]
```
自解释并且优雅。

扯的太远了，还是那句话，不了解RESTful风格的通知请面壁然后去百度。
咱们看看怎么用Flask来实现一个最简单的RESTful接口

新建脚本 book.py

{% highlight python%}
from flask import Flask
from flask import request # 导入request对象

book_app = Flask(__name__)


@book_app.route('/books' , methods = ['GET','POST'])   # 这个函数允许GET和POST，其他动作不允许
def book_interface():
    if request.method == 'GET':  # 判断动作
        return 'get a book'
    if request.method == 'POST':
        return 'POST'

if __name__ == '__main__':
    book_app.run(host='0.0.0.0', port=27016, debug='true')
{% endhighlight %}

其实这种代码的缺点在于判断分支多，缺乏可读性。因为HTTP动词常用得上的大概有5种：GET POST PUT DELETE PATCH。


下面我们来看一下如果是使用flask提供的restful包是怎样写的

cat.py

{% highlight python%}
from flask import Flask
from flask_restful import Resource, Api

cat_app = Flask(__name__)  # 初始化Flask对象
cat_api = Api(cat_app)          # 初始化Api对象 


class Cat(Resource):
    def get(self, cat_id):    # GET方法
            return 'get a cat %s:' % cat_id
    def post(self):    # POST方法
        return 'post a cat'

cat_api.add_resource(Cat, '/cats/<string:cat_id>')   # 将Cat资源注册

if __name__ == '__main__':
    cat_app.run(host='0.0.0.0', port=27016, debug='true')
{% endhighlight %}

显然这样的写法更加优雅。
将RESTful资源添加到蓝图中：

{% highlight python%}
from flask import Blueprint
from flask_restful import Resource, Api

cat_blueprint = Blueprint('cats', __name__)  # 用Blueprint构造函数替代Flask构造函数
cat_api = Api(cat_blueprint)


class Cat(Resource):
    def get(self, cat_id):
            return 'get a cat %s:' % cat_id

    def post(self):
        return 'post a cat'

cat_api.add_resource(Cat, '/cats/<string:cat_id>')
{% endhighlight %}

然后在app包的init脚本里导入cat_blueprint并注册就OK了：

_ __init__ _.py:

{% highlight python%}
from cat import cat_blueprint
from flask import Flask
app = Flask(__name__)
app.register_blueprint(cat_blueprint, url_prefix='/api/v3')
{% endhighlight %}

总结： 到目前为止的Python系列文章，我们可以总结一些Python和Flask的特点：

1. 可以导入任何东西，变量、函数、类 都可以。
1. 到目前为止没有写过一行配置代码。而使用Spring的时候，至少得花时间去维护你的Pom.xml
1. Flask只提供最小模块。当你想用restful的时候，Flask只是提供了一种类的写法而已，至于SQL ORM这些你需要自己去找其他包来搞定。而SpringBoot提供了Spring Data JPA:从HTTP请求到ORM完成增删改查的整套体系。


下一篇文章我们来学习Python里比较流行的ORM: SQLAlchemy