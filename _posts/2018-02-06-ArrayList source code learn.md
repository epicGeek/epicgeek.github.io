---
layout: post
title: "ArrayList源码学习"
date: 2018-02-06
excerpt: "学习Java经典集合框架里的ArrayList."
tags: [Java]
slug: arraylist-source
feature: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522528924487&di=5b9504483e5ec80c3df5350feb58d5d3&imgtype=0&src=http%3A%2F%2Fp2.gexing.com%2Fshaitu%2F2012%2F4%2F24%2F201244174572106.jpg
---

*ArrayList在平时工作中几乎天天都在用。回想起三四年前在刚开始学编程时，只使用数组的恐怖支配。正好最近项目不忙，抽出来点时间看看源码是怎样的*

Java version: 1.8
## 类说明
ArrayList顾名思义，数组表。属于一种线性列表。
让我们来看一下官方文档给出的介绍：
```    
Resizable-array implementation of the List interface.
Implements all optional list operations, and permits all elements, including null.
In addition to implementing the List interface, this class provides methods to manipulate the size of the array that is used internally to store the list.
(This class is roughly equivalent to Vector, except that it is unsynchronized.)

```
从这段文字中，有两点重要的信息：
1. 可变的长度是ArrayList的重要特征。
1. ArrayList线程不安全,而Vector是线程安全的。要记住这一点。


```
The size, isEmpty, get, set, iterator, and listIterator operations run in constant time.
The add operation runs in amortized constant time, that is, adding n elements requires O(n) time.
All of the other operations run in linear time (roughly speaking).
The constant factor is low compared to that for the LinkedList implementation. 
```
这段信息告诉我们：
1. size, isEmpty, get, set, iterator等操作效率极高。
1. 添加元素的效率也还不错。
1. 其他操作基本复杂度都是线性的。
1. 比较起LinkedList来说，效率要高一些。

```
Each ArrayList instance has a capacity. 
The capacity is the size of the array used to store the elements in the list.
It is always at least as large as the list size. As elements are added to an ArrayList, its capacity grows automatically.
The details of the growth policy are not specified beyond the fact that adding an element has constant amortized time cost. 
```
ArrayList的size的变化是由一套逻辑来控制的，对它进行增删操作的时候，size会自动变化。

ArrayList类的定义：

{% highlight java %}
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{% endhighlight %}

这是因为：
* AbstractList这个抽象类提供了常用方法的实现，继承它避免重复代码。

* List接口定义了列表必须实现的方法。

*  RandomAccess是一个标记接口，接口内没有定义任何内容。

* 实现了Cloneable接口的类，可以调用Object.clone方法返回该对象的浅拷贝。

* 通过实现 java.io.Serializable 接口以启用其序列化功能。未实现此接口的类将无法使其任何状态序列化或反序列化。序列化接口没有方法或字段，仅用于标识可序列化的语义。

## 成员变量：
{% highlight java %}
   /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10; //默认容量为10

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size; //list的大小，包含元素的数量
{% endhighlight %}
空数组：
{% highlight java %}

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};  //区别上一个空数组，这个数组使用默认容量长度。

{% endhighlight %}
真正保存数据的数组：
{% highlight java %}
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

{% endhighlight %}
## 构造方法：

{% highlight java %}
public ArrayList();
public ArrayList(int initialCapacity);
public ArrayList(Collection<? extends E> c);
{% endhighlight %}

第一种:
{% highlight java %}
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
{% endhighlight %}
我们最常用的构造方法:
{% highlight java %}
List<E> list = new ArrayList<>();
{% endhighlight %}
我们可以看到，当不指定容量参数时，内部给出的是容量为10的空数组。

第二种构造，指定容量：

{% highlight java %}
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
{% endhighlight %}
1.当指定的容量大于0时，新建长度为容量参数的空数组。
2.当容量参数为0，数据数组为长度为0的数组。
3.参数为负数，抛出异常。

第三种构造，参数为集合：
{% highlight java %}
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
{% endhighlight %}
逻辑也非常简单，就是将这个集合转为数组数据。
## 常用操作：
### add():
{% highlight java %}
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
{% endhighlight %}
add() 方法继承自 AbstractList类。从代码上看到，在赋值前，内部确认了一下size+1这个数据是否
trimToSize():
{% highlight java %}
    /**
     * Trims the capacity of this <tt>ArrayList</tt> instance to be the
     * list's current size.  An application can use this operation to minimize
     * the storage of an <tt>ArrayList</tt> instance.
     */
    public void trimToSize() {
        modCount++; //继承自AbstractList的变量，表示改变次数，trimToSize操作认为是改变了一次list
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        } //如果目前size小于装载数据的数组时，需要重新设置elementData的size
    }
{% endhighlight %}
关于这个方法的解释在这里：
<https://www.zhihu.com/question/51057675/answer/123911537>

* capacity 容量，指的是这个arrayList实例最多可装多少个元素
* size 指的是这个arrayList实例已经装了多少个元素

现在写一段代码来debug一下：
{% highlight java %}
		ArrayList<String> list = new ArrayList<>();
		list.add("1");
		list.trimToSize();

{% endhighlight %}
执行完第一行的变量状况：
![1.png](http://upload-images.jianshu.io/upload_images/9774769-351b299ea0c647d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
执行完第二行的变量状况：
![1.png](http://upload-images.jianshu.io/upload_images/9774769-6cbcd586e14f6668.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们看到，elementData是一个长度为10的Object数组，第一个数据为“1”，其他为0.

执行trimToSize后:
![1.png](http://upload-images.jianshu.io/upload_images/9774769-b68ca28adf350742.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，所有null元素都被清空了。
这样做的好处是可以节约一部分内存。


再往下看我们就看到：
{% highlight java %}
public void ensureCapacity(int minCapacity)
private void ensureCapacityInternal(int minCapacity)
private void ensureExplicitCapacity(int minCapacity)
{% endhighlight %}
从这三个方法的名字上来看，应该是确保list容量时使用的方法。
读完这三个方法，还是有些不清楚。但是发现核心方法为：
{% highlight java %}
ensureExplicitCapacity(minCapacity);
grow(minCapacity);
{% endhighlight %}

下面研究一下ArrayList的动态扩容机制：

{% highlight java %}
		ArrayList<String> list = new ArrayList<>();
		while(true){
			list.add("1");
		}
{% endhighlight %}
debug这段代码，发现list里的elementData在第一个元素进入时，elementData长度为10,第一个元素为1,剩下的全都为null;
当10个元素全都入满后，数组长度变为15，再次将其入满数据，elementData的长度变为22,33,49,73...不难发现，每次扩容都会使elementData的长度变为原来的1.5倍。每次扩容还需要将elementData重新拷贝到新的数组中。这意味着，ArrayList中元素不要过多，否则在扩容数组的时候有可能会出效率问题。但是一般不会出现这种问题，在平时工作中，我用的破笔记本，arrayList里面放几万条数据也没啥事。

grow方法的代码：
{% highlight java %}
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); //除以二并向下取整
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);//拷贝数据数组
    }
{% endhighlight %}