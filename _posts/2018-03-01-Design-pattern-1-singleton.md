---
layout: post
title: "设计模式学习之一：单例模式"
date: 2018-03-01
excerpt: "学习什么是单例模式"
tags: [Design pattern,Java]
slug: dp-singleton
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522528829320&di=f877d8377ee506c503e5b7288a24d3b6&imgtype=0&src=http%3A%2F%2Fimgsrc.baidu.com%2Fimgad%2Fpic%2Fitem%2F9a504fc2d562853563cc110c9bef76c6a7ef63eb.jpg
---

## 为什么用单例模式
在实例化对象的时候，怎么样才能让这个对象在全局唯一，这个是单例模式解决的问题。
为什么对象全局唯一？因为有可能有这样的需求：
- new 出来这种对象非常浪费资源，频繁的new这个对象会引发各种问题。（次要需求）
- 为了解决数据唯一的问题。例如系统垃圾桶。每次我删除文件的时候，那么我不可能说每次new 一个新的垃圾桶吧，删几次文件new几次垃圾桶对象。过几天整个系统全是垃圾桶了。不管我删什么东西，进入的永远是那一个垃圾桶。（主要需求）

## 我遇上过的坑

其实到现在为止对单例模式的理解依然很薄弱，因为自己写东西的时候不会去用到。看过几次单例模式后没理解，只记得个 getInstance() 方法了。当时比较困惑，还去stackoverflow当过伸手党，结果问题扣了我好多分。然后自己把单例模式的代码写一遍，大概明白点意思了。换句话讲，学设计模式是急不来的（对我来说），这需要工作经验的积累和大量思考。刚上班那会儿，用时间的时候喜欢用Calendar这个类，觉得政治正确，因为书上就是这么教的。总所周知，当你想得到一个日历类的实例你通常会这样写：
{% highlight java %}
Calendar c = Calendar.getInstance();
{% endhighlight %}
哦！原来这就是单例模式！但是好像有些不对劲啊。。。Date 我们也经常用，Date怎么不是getInstance呢？Calendar很耗资源？不像啊。全局唯一？嗯。。。好像也不一定。

不是这样的。Calendar并不是单例模式，因为这个类并没有全局唯一性这种需求，你可以搞出很多个Calendar对象出来用，每个对象之间都不一样。

那么好，问题来了，我们怎么用代码去完成这种需求呢？

1. 首先，为了满足对象全局唯一的性质，我这个类是不允许外部去使用new 操作符来造出新对象，因此，在单例类中，必须讲构造方法设置为private。 （特点1）

1. 既然不允许外部new,那么只能单例类自己去new一个对象出来给外面用了。（特点2）
1. 单例类需要提供方法供其他类来得到这个唯一对象实例。但是呢，对象又不给外面new,还得提供方法。那就提供类的静态方法给出这个唯一的对象。（特点3）



好了，那么现在在脑海里构思一下这个类大概怎么写，然后动手开始写。


## 懒汉单例
{% highlight java %}
public class SingletonLazy {
	private static SingletonLazy singleton = null; // 唯一对象，所有单例类对象的需求调用都会且只会得到它。先不给值，外部调用再给，以节省时间。

	private SingletonLazy() { // 私有化构造器，特点1
	}

	public static SingletonLazy getInstance() { // 提供方法供外部得到这个唯一对象
		// 第一次来拿对象的时候，对象还没初始化呢。
		if (singleton == null) {
			singleton = new SingletonLazy();
		}
		return singleton;

	}
}
{% endhighlight %}

嗯，如果可以在没有外部提示的情况下写出来，说明理解的还行。
但是这段代码是有缺陷的。在多线程的情况下，如果两个线程同时进入了if块，那么这两个线程都会new 一个对象出来并赋值再返回，那拿到的将是两个不同对象。因此这段代码是线程不安全的。再由于单例实例再最开始为空，你不调用，不需要，我就不new对象出来，因此这种单例模式也叫做“懒汉单例”。

 - 优点：加载快。
 - 缺点：只支持单线程。


## 饿汉单例
为了克服线程问题，需要对懒汉模式稍微加以改动。让类刚加载的时候就把单例实例new出来，随时等待外部调用，就避免了现线程问题，因为在线程访问前，我对象已经有了，你拿就是了。

{% highlight java %}
public class SingletonStarve {
    private static SingletonStarve singleton = new SingletonStarve(); // 先new出来

    private SingletonStarve () { // 私有化构造器，特点1
    }

    public static SingletonStarve getInstance() { // 提供方法供外部得到这个唯一对象

        return singleton;

    }
}
{% endhighlight %}

这下大家都满意了，多线程也没问题了，这很显然。但是有个问题，由于对象最开始已经new出来了，性能上必定略逊于懒汉式。这个要具体问题具体分析。不过在性能要求不高的情况下，完全可以忽略了。

- 优点：支持多线程。
- 缺点：可能会有性能问题存在。
也就是说，在对性能要求不那么苛刻的情况下，饿汉式完全满足需求，是建议使用的。如果使用懒汉式，在单线程变多线程的情况下，又得回来改单例类的代码了。