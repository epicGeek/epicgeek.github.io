---
layout: post
title: "Python开发之路系列：Flask入门使用"
date: 2018-04-02
excerpt: "Flask应用的最简单实现."
tags: [Python]
slug: py-starter-flask-intro
category: Python
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522649764826&di=fe825e1d3dc597742ee7593eccf50379&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20150422%2Fflask.jpg
---
## Python开发之路系列：Flask入门使用

正如Java一样，Python Web应用的开发也离不开框架。那么在python的世界里面，主流框架有这几种:
* Flask
* Django

他们的区别和选型具体请参考百度和知乎，在这里就不赘述了。
这个系列是基于Flask的开发心得。

根据我的使用,Flask的特点：
* 小而强大，无需配置
* 不是一站式的框架
* 简单好学

Flask官网：
<http://flask.pocoo.org/>

这里给大家伙提个醒： **Flask某些组件是有中文文档的，不推荐使用。因为中文文档的更新落后太多，很多代码和文档对于新版本是不兼容的。**

本文使用python2.7.14和flask 0.12.2

让我们开始吧！

新建文件： flask_test.py

{% highlight python%}

from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return 'Index Page'


if __name__ == '__main__':
    app.run()
{% endhighlight %}

右键,Run 'flask_test'

控制台显示如下：

![1.png](https://upload-images.jianshu.io/upload_images/9774769-dd904148291c9af0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这就说明我们的flask应用已经在本地跑起来了，端口为5000

在浏览器里简单访问：
![2.png](https://upload-images.jianshu.io/upload_images/9774769-b46c1ee164a5392d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个最简单的Flask应用就完成了。

简单解释一下代码：

```
app = Flask(__name__) # 实例化Flask对象
@app.route('/') #app注册一个链接给app
if块： 如果是以这个脚本文件为入口，启动app
```
问题来了：你在别的python脚本里写出这些链接是不能使用的，因为flask和spring的原理不一样。所有你写好的spring controller在spring应用运行时，都会启用，而flask不是这样。需要一种手段来完成这种注册的行为。
下一篇文章将介绍： 如何使用blue print实现多个文件接口的启用。
