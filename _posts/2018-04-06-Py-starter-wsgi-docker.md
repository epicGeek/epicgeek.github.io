---
layout: post
title: "Python开发之路系列外传：WSGI的Docker镜像制作"
date: 2018-04-06
excerpt: "如何实现一个最简单的的WSGI docker镜像并运行。"
tags: [Python,Docker,gunicorn,WSGI]
slug: py-starter-wsgi-docker
category: Python
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1523008293770&di=b1d74f32233859eba8ac8951b4d7483a&imgtype=jpg&src=http%3A%2F%2Fimg4.imgtn.bdimg.com%2Fit%2Fu%3D519907769%2C3233832181%26fm%3D214%26gp%3D0.jpg
---

SQLAlchemy的文章还在写。。。因为官方文档有点多，而且这个东西也比较重要，还是想好好写着，所以先跳票了。。。

虽然开发出来的Flask应用可以直接运行，但是缺少了应用服务器，就像tomcat之于war包。
经过学习，python使用WSGI服务器。
WSGI是web service gateway interface的缩写。关于WSGI的作用不在这里赘述了，总之，python web 应用服务需要跑在WSGI里。这里的理解可能有些问题，还是请大神纠正。
WSGI选择使用gunicorn,web应用选择基于flask开发的简单程序。代码可以参考之前的文章。

#### 步骤：
##### 安装pipreqs
因为flask项目使用了很多模块，手动的去收集这些模块名称和版本信息首先是很费事的，而且也不准确。我们的目标是把收集好的模块信息写在requirements.txt里面。便于维护并且在后面制作docker镜像的时候需要按照这里的依赖信息来安装各个python模块。

{% highlight shell %}
pip install pipreqs
{% endhighlight %}

需要注意的是，使用pip freeze也可以得到模块信息。
区别是，pipreqs只会导出这个目录下面的python项目依赖的模块。而pip freeze会导出整个环境的全部python模块包信息。如果想安装一个比较完整版的python环境，可以不安装pipreqs.
##### 安装gunicorn

{% highlight shell %}
pip install gunicorn
{% endhighlight %}

##### 生成requirements.txt

使用pipreqs:

{% highlight shell %}
cd /home/neil/neil-restful-interface-project     # 切到项目代码所在目录
pipreqs ./                                                         # 导出requirements.txt
{% endhighlight %}
稍微等一会儿后，requirements.txt就导出了。需要注意的是，我们的gunicorn还没有写入到这里。

查询gunicorn相关信息:
{% highlight shell %}
pip show gunicorn
{% endhighlight %}
显示：
```
Name: gunicorn
Version: 19.7.1
Summary: WSGI HTTP Server for UNIX
Home-page: http://gunicorn.org
Author: Benoit Chesneau
Author-email: benoitc@e-engura.com
License: MIT
Location: /root/anaconda2/lib/python2.7/site-packages
Requires:
```
我们之需要知道版本号就可以了，在requirements.txt里追加一行:
```
gunicorn==19.7.1
```

如果不使用pipreqs:

{% highlight python %}
pip freeze > requirements.txt
{% endhighlight %}

##### 准备镜像
这里我声明一下： 我使用的python版本为2.7.14。所以先去docker hub上查一下你需要的python镜像版本。

下载python2.7.14镜像:
```
docker pull python:2.7.14
```
下载完成后，写Dockerfile
```
From python:2.7.14              # 基于python2.7.14镜像


WORKDIR /usr/src/app       # 设定这个目录为工作目录，所有执行的命令都在这里执行

COPY requirements.txt .     #把刚才写好的requirements.txt复制到工作目录

RUN pip install --no-cache-dir -r requirements.txt  # 安装requirements.txt里的模块

COPY . .                             # 把当前宿主机的（dockerfile所在目录）文件复制到工作目录

EXPOSE 8123                   # 暴露8123端口

CMD ["/bin/bash"]              # 使用bash

ENTRYPOINT gunicorn -w 4 manager:app -b 0.0.0.0:8123          # 使用gunicorn启动flask应用

```
这个是最简单的dockerfile的写法了。
需要注意的是gunicorn命令这里,稍后解释。

##### 构造自定义镜像
```
docker build -t nt:0.9 /home/neil/neil-restful-interface-project/
```
意思是 构造一个镜像，镜像名为nt,镜像标签（-t）为0.9，dockerfile所在目录为 ： /home/neil/neil-restful-interface-project/
执行后，日志如下：
```
[root@bjc-vm-bi-2 neil-restful-interface-project]# docker build -t nt:0.9 /home/neil/neil-restful-interface-project/
Sending build context to Docker daemon  92.16kB
Step 1/8 : From python:2.7.14
 ---> 2863c80c418c
Step 2/8 : WORKDIR /usr/src/app
Removing intermediate container a511ab3f3edb
 ---> 780c7ebdd30e
Step 3/8 : COPY requirements.txt .
 ---> 48e70b12f720
Step 4/8 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in 50deeda397b7
Collecting Flask==0.12.2 (from -r requirements.txt (line 1))
  Downloading Flask-0.12.2-py2.py3-none-any.whl (83kB)
Collecting Flask_RESTful==0.3.6 (from -r requirements.txt (line 2))
  Downloading Flask_RESTful-0.3.6-py2.py3-none-any.whl
Collecting gunicorn==19.7.1 (from -r requirements.txt (line 3))
  Downloading gunicorn-19.7.1-py2.py3-none-any.whl (111kB)
Collecting itsdangerous>=0.21 (from Flask==0.12.2->-r requirements.txt (line 1))
  Downloading itsdangerous-0.24.tar.gz (46kB)
Collecting Jinja2>=2.4 (from Flask==0.12.2->-r requirements.txt (line 1))
  Downloading Jinja2-2.10-py2.py3-none-any.whl (126kB)
Collecting Werkzeug>=0.7 (from Flask==0.12.2->-r requirements.txt (line 1))
  Downloading Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
Collecting click>=2.0 (from Flask==0.12.2->-r requirements.txt (line 1))
  Downloading click-6.7-py2.py3-none-any.whl (71kB)
Collecting pytz (from Flask_RESTful==0.3.6->-r requirements.txt (line 2))
  Downloading pytz-2018.3-py2.py3-none-any.whl (509kB)
Collecting six>=1.3.0 (from Flask_RESTful==0.3.6->-r requirements.txt (line 2))
  Downloading six-1.11.0-py2.py3-none-any.whl
Collecting aniso8601>=0.82 (from Flask_RESTful==0.3.6->-r requirements.txt (line 2))
  Downloading aniso8601-3.0.0-py2.py3-none-any.whl
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->Flask==0.12.2->-r requirements.txt (line 1))
  Downloading MarkupSafe-1.0.tar.gz
Installing collected packages: itsdangerous, MarkupSafe, Jinja2, Werkzeug, click, Flask, pytz, six, aniso8601, Flask-RESTful, gunicorn
  Running setup.py install for itsdangerous: started
    Running setup.py install for itsdangerous: finished with status 'done'
  Running setup.py install for MarkupSafe: started
    Running setup.py install for MarkupSafe: finished with status 'done'
Successfully installed Flask-0.12.2 Flask-RESTful-0.3.6 Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 aniso8601-3.0.0 click-6.7 gunicorn-19.7.1 itsdangerous-0.24 pytz-2018.3 six-1.11.0
Removing intermediate container 50deeda397b7
 ---> 396511eadc1f
Step 5/8 : COPY . .
 ---> 2a4377c4dae3
Step 6/8 : EXPOSE 8123
 ---> Running in 80efc289373e
Removing intermediate container 80efc289373e
 ---> b7c7611f7e98
Step 7/8 : CMD ["/bin/bash"]
 ---> Running in 0ba55cd5de15
Removing intermediate container 0ba55cd5de15
 ---> a6449eb7e8bc
Step 8/8 : ENTRYPOINT gunicorn -w 4 manager:app -b 0.0.0.0:8123
 ---> Running in 501852c7bc59
Removing intermediate container 501852c7bc59
 ---> 060d5b055ef3
Successfully built 060d5b055ef3
Successfully tagged nt:0.9
```
说明镜像已经成功构造。

##### 运行容器
{% highlight shell%}
docker run -d --name player1 -p 8123:8123 -p 27016:27016 nt:0.9
{% endhighlight %}
以后台守护进程的模式(-d)启动容器 :player1 ,暴露出端口8123和27016，使用镜像:nt:0.9

查看容器情况:
{% highlight shell%}
docker ps
{% endhighlight %} 

{% highlight shell%}
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                              NAMES
08e6cb4f3504        nt:0.9              "/bin/sh -c 'gunicor…"   2 hours ago         Up 7 minutes        0.0.0.0:8123->8123/tcp, 0.0.0.0:27016->27016/tcp   player1
{% endhighlight %}
这说明容器已成功运行。

访问API
{% highlight shell%}
[root@bjc-vm-bi-2 neil-restful-interface-project]# curl http://10.16.2.84:8123/api/leaves
This tree has leaves
[root@bjc-vm-bi-2 neil-restful-interface-project]# curl http://10.16.2.84:27016/api/leaves
curl: (7) Failed to connect to 10.16.2.84 port 27016: Connection refused
{% endhighlight %}
查看容器日志:
{% highlight shell%}
docker logs -f --tail=100 player1
{% endhighlight %}

{% highlight shell%}
[root@bjc-vm-bi-2 neil-restful-interface-project]# docker logs -f --tail=100 player1
[2018-04-06 05:20:05 +0000] [5] [INFO] Starting gunicorn 19.7.1
[2018-04-06 05:20:05 +0000] [5] [INFO] Listening at: http://0.0.0.0:8123 (5)
[2018-04-06 05:20:05 +0000] [5] [INFO] Using worker: sync
[2018-04-06 05:20:05 +0000] [9] [INFO] Booting worker with pid: 9
[2018-04-06 05:20:05 +0000] [10] [INFO] Booting worker with pid: 10
[2018-04-06 05:20:05 +0000] [11] [INFO] Booting worker with pid: 11
[2018-04-06 05:20:05 +0000] [12] [INFO] Booting worker with pid: 12
[2018-04-06 06:52:17 +0000] [5] [INFO] Starting gunicorn 19.7.1
[2018-04-06 06:52:17 +0000] [5] [INFO] Listening at: http://0.0.0.0:8123 (5)
[2018-04-06 06:52:17 +0000] [5] [INFO] Using worker: sync
[2018-04-06 06:52:17 +0000] [9] [INFO] Booting worker with pid: 9
[2018-04-06 06:52:17 +0000] [10] [INFO] Booting worker with pid: 10
[2018-04-06 06:52:17 +0000] [12] [INFO] Booting worker with pid: 12
[2018-04-06 06:52:17 +0000] [13] [INFO] Booting worker with pid: 13
{% endhighlight %}
访问API是可以的。

##### gunicorn命令说明
{% highlight shell%}
gunicorn -w 4 manager:app -b 0.0.0.0:8123 
{% endhighlight %}
-w 参数为4 意味着工作进程数为4.
manager:app 以app的方式启动manager.py
-b 绑定地址,端口为8123
0.0.0.0表示任何ip

执行```ps -ef|grep gunicorn```: （宿主机）
{% highlight shell%}
root     17120 17105  0 13:20 ?        00:00:00 /bin/sh -c gunicorn -w 4 manager:app -b 0.0.0.0:8123 /bin/bash
root     17145 17120  0 13:20 ?        00:00:04 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root     17154 17145  0 13:20 ?        00:00:01 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root     17155 17145  0 13:20 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root     17156 17145  0 13:20 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root     17158 17145  0 13:20 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root     18335 25295  0 14:49 pts/2    00:00:00 grep --color=auto gunicorn
{% endhighlight %}

进入容器并查看进程:
{% highlight shell%}
[root@bjc-vm-bi-2 neil-restful-interface-project]# docker exec -it player1 /bin/bash
root@08e6cb4f3504:/usr/src/app# ps -ef|grep gunicorn
root         1     0  0 06:52 ?        00:00:00 /bin/sh -c gunicorn -w 4 manager:app -b 0.0.0.0:8123 /bin/bash
root         5     1  0 06:52 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root         9     5  0 06:52 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root        10     5  0 06:52 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root        12     5  0 06:52 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root        13     5  0 06:52 ?        00:00:00 /usr/local/bin/python /usr/local/bin/gunicorn -w 4 manager:app -b 0.0.0.0:8123
root        29    23  0 07:04 pts/0    00:00:00 grep gunicorn
root@08e6cb4f3504:/usr/src/app# 
{% endhighlight %}

这里我们可以得出以下结论，基于进程和gunicorn日志：
* pid为5的进程是pid为9 10 12 13这四个work的父进程
* worker之间使用异步模式进行处理，会让http的响应更快。-w 参数越多响应越快。

我的manager.py代码：
{% highlight python%}
from app import app

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=27016, debug='true')
{% endhighlight %}
显然是以27016端口启动的应用。经过gunicorn后，已经把服务转为开放8123端口了，通过8123访问数据。