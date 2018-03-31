---
layout: post
title: "Nginx配置简单的反向代理"
date: 2018-03-08
excerpt: "学习一下如何简单的配置Nginx反向代理"
tags: [Nginx,Java]
slug: nginx-reserve-config
---

客户提出需求将现在的项目改造为HTTPS的。由于现在的架构是前后端分离的形式，前端使用nginx服务器，后端是基于Spring Boot的微服务。拿一个功能模块举例，这个功能称为pgw。
## 目标：实现一个Nginx的反向代理
我本机的IP为 xxx.xx.xx.235
Nginx服务器IP xxx.xx.xx.48 
二者在同一个网段内
### 后端配置：
**由于客户那边需要对Url进行安全认证，而且后端微服务数据接口的Url需要带指定的上下文才能被Nginx反向代理，所以第一步应该是为微服务添加上下文。**

pgw原来提供的数据接口都是**RESTful**风格的，例如：
http://localhost:8090/api/vi/greetings
![1.png](http://upload-images.jianshu.io/upload_images/9774769-f8f69f042800ee67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是其他模块的URL也是类似的，不能进行有效的区分，因此需要为它增加上下文的配置。
在该模块的application.yml中加入如下配置：
![1.png](http://upload-images.jianshu.io/upload_images/9774769-70552482f7848711.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


需要注意的是，这里的context-path必须以斜线开头。
重启服务，测试接口URL： http://localhost:8090/pgw/api/vi/greetings
成功获取数据，说明上下文添加成功。
![1.png](http://upload-images.jianshu.io/upload_images/9774769-5d82b64fcd0f410d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Nginx 配置：
通过更改nginx.conf来完成。
![1.png](http://upload-images.jianshu.io/upload_images/9774769-fdaa41cc71b30467.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

改变location的配置:
![1.png](http://upload-images.jianshu.io/upload_images/9774769-4d63c4febff06127.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

保存，重启:
$ systemctl restart nginx


