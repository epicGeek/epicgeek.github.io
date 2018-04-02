---
layout: post
title: "Python开发之路系列：多接口模块的组织和开发"
date: 2018-04-02
excerpt: "在Flask里如何进行多接口开发."
tags: [Python,Flask]
slug: py-starter-blueprint
category: Python
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522669019177&di=a1c0f14c7af53d74e76ac28f69ec0f97&imgtype=jpg&src=http%3A%2F%2Fimg2.imgtn.bdimg.com%2Fit%2Fu%3D3615493377%2C2733693374%26fm%3D214%26gp%3D0.jpg
---
## Flask多接口模块开发:Blueprint

**熟悉Spring Boot的朋友都知道，如果你想新建一个接口，那么这很简单，只要在controller注解标记的类里面新写方法，并在这个方法上以RequestMapping注解标记，这个方法就成为了一个接口方法。但是在Flask里面是行不通的。**

在上一篇文章里我们学习了Flask的最小应用。
* 首先第一步，实例化一个Flask对象（app = Flask(__name__)）
* 没个接口的入口也是一个方法，在这个方法上标记注解 ``` @app.route('/mapping') ``` 就完成了一个接口

但是随着应用体积的增大，势必要完成模块化。说白了，就是不可能所有接口都写到一个脚本里。

和Spring不同，不同脚本里的接口必须得经过一种“集中提取”的操作才能在这个Flask应用运行时，全部同时启用。

为了实现模块化，我们需要使用Blueprint(蓝图)

我们距离说明：
新建工程，在工程里我们有如下文件：

app
|-- __ _init_ __.py    # pycharm生成
|-- controller1.py     # 接口1
|-- controller2.py     # 接口2
|-- manager.py       # 启动脚本

按照SpringBoot习惯，应用总是有一个main入口的。按照习惯，flask应用的入口为manager.py

如果按照上一篇文章里的接口的写法，我们在controller1.py和controller2.py里的代码是这样的：

{% highlight python %}
from flask import Flask
con1= Flask(__name__)

@con1.route('/con1')                  # 同理，controller2就是con2，不再赘述
def index():
    return 'con1'


if __name__ == '__main__':
    con1.run()
{% endhighlight %}

上一篇文章也提到了，这个if块，只有你去把他当做入口来用，整个脚本才会生效。
所以我们需要一种手段来把这个接口集中到一个地方。
这就是blueprint(蓝图)

先看代码：

controller1.py:

{% highlight python %}

from flask import Blueprint  # 导入蓝图
con1_blueprint = Blueprint('con1',__name__)  # 初始化一个蓝图，而不是Flask对象 


@con1_blueprint.route('/c1')   # 接口函数。比上一篇文章的代码更少！
def con1():
    return 'c1 is here!'

{% endhighlight %}


controller2.py:

{% highlight python %}

from flask import Blueprint   # controller1.py的复制
con2_blueprint = Blueprint('con2',__name__)


@con2_blueprint.route('/c2')
def con2():
    return 'c2 is here!'


{% endhighlight %}

manager.py:

**这里的代码很重要了，注意看**

{% highlight python %}

from controller1 import con1_blueprint  # 导入全部蓝图变量
from controller2 import con2_blueprint
from flask import Flask
app = Flask(__name__)   # 初始化Flask对象
app.register_blueprint(con1_blueprint)   # 将所有蓝图对象注册到app这个flask对象内
app.register_blueprint(con2_blueprint)


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=27016, debug='true')  # 启动入口


{% endhighlight %}

然后访问 http://localhost:27016/c1 或者c2，发现成功访问。

## 更合理的组织代码

现在有个小知识点还没有触及，那就是 __ _init_ __.py 脚本。
这个脚本是IDE自动生成的，每一个python package都有一个。
按照我的理解，这里应该写一些初始化对象的代码。比如初始化一个数据库连接等动作。
按照这个逻辑，其实整个蓝图注册，manager.py的app对象初始化，实际上都可以写在init脚本里。

优化后的代码：

controller1.py 和 controller2.py代码不变

 __ _init_ __.py ：

{% highlight python %}
from controller1 import con1_blueprint   # 导入全部蓝图对象
from controller2 import con2_blueprint
from flask import Flask
app = Flask(__name__)    # 初始化Flask对象
app.register_blueprint(con1_blueprint , url_prefix = '/api/v1')
app.register_blueprint(con2_blueprint, url_prefix = '/api/v2')  # 将全部蓝图注册。url_prefix 参数意思为给这个蓝图内的接口地址添加前缀。
{% endhighlight %}

manager.py:

{% highlight python %}
from app import app   # 导入app包下的init脚本中初始化好的app对象

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=27016, debug='true')   #启动程序
{% endhighlight %}

现在我们开放了两个接口：

* http://localhost:27016/api/v1/c1
* http://localhost:27016/api/v2/c2

但是问题又出现了。似乎这两个接口距离RESTful风格好像还差一些东西。（不知道RESTful的同学请面壁一分钟然后百度一下）

下一篇文章我们来一起学习： 如何使用flask编写RESTful风格的接口