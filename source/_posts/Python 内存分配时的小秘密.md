---
title: 推荐系列-Python 内存分配时的小秘密
categories: 热门文章
tags:
  - Popular
author: OSChina
top: 1699
cover_picture: 'https://static.oschina.net/uploads/img/201909/10234900_uysX.jpg'
abbrlink: 92b41ca7
date: 2021-04-15 09:19:21
---

&emsp;&emsp;Python 中的sys 模块极为基础而重要，它主要提供了一些给解释器使用（或由它维护）的变量，以及一些与解释器强交互的函数。 本文将会频繁地使用该模块的getsizeof() 方法，因此，我先简要介绍...
<!-- more -->

                                                                                                                                                                                         
###   
Python 中的 ```java 
  sys
  ```  模块极为基础而重要，它主要提供了一些给解释器使用（或由它维护）的变量，以及一些与解释器强交互的函数。 
本文将会频繁地使用该模块的 ```java 
  getsizeof()
  ```  方法，因此，我先简要介绍一下： 
 
 该方法用于获取一个对象的字节大小（bytes） 
 它只计算直接占用的内存，而不计算对象内所引用对象的内存 
 
这里有个直观的例子： 
 ```java 
  import sys

a = [1, 2]
b = [a, a]  # 即 [[1, 2], [1, 2]]

# a、b 都只有两个元素，所以直接占用的大小相等
sys.getsizeof(a) # 结果：80
sys.getsizeof(b) # 结果：80

  ```  
上例说明了一件事：一个静态创建的列表，如果只包含两个元素，那它自身占用的内存就是 80 字节，不管其元素���指向的对象是什么。 
好了，拥有这把测量工具，我们就来探究一下 Python 的内置对象都藏了哪些小秘密吧。 
![Test](https://oscimg.oschina.net/oscnet/u7y1GDwELoSaMrk.jpg  'Python 内存分配时的小秘密') 
 
#### 1、空对象不是“空”的！ 
对于我们熟知的一些空对象，例如空字符串、空列表、空字典等等，不知道大家是否曾好奇过，是否曾思考过这些问题：空的对象是不是不占用内存呢？如果占内存，那占用多少呢？为什么是这样分配的呢？ 
直接上代码吧，一起来看看几类基本数据结构的空对象的大小： 
 ```java 
  import sys
sys.getsizeof("")      # 49
sys.getsizeof([])      # 64
sys.getsizeof(())      # 48
sys.getsizeof(set())   # 224
sys.getsizeof(dict())  # 240

# 作为参照：
sys.getsizeof(1)       # 28
sys.getsizeof(True)    # 28

  ```  
可见，虽然都是空对象，但是这些对象在内存分配上并不为“空”，而且分配得还挺大（记住这几个数字哦，后面会考）。 
排一下序：基础数字<空元组 < 空字符串 < 空列表 < 空集合 < 空字典。 
这个小秘密该怎么解释呢？ 
因为这些空对象都是容器，我们可以抽象地理解：它们的一部分内存用于创建容器的骨架、记录容器的信息（如引用计数、使用量信息等等）、还有一部分内存则是预分配的。 
 
#### 2、内存扩充不是均匀的！ 
空对象并不为空，一部分原因是 Python 解释器为它们预分配了一些初始空间。在不超出初始内存的情况下，每次新增元素，就使用已有内存，因而避免了再去申请新的内存。 
那么，如果初始内存被分配完之后，新的内存是怎么分配的呢？ 
 ```java 
  import sys
letters = "abcdefghijklmnopqrstuvwxyz"

a = []
for i in letters:
    a.append(i)
    print(f'{len(a)}, sys.getsizeof(a) = {sys.getsizeof(a)}')

b = set()
for j in letters:
    b.add(j)
    print(f'{len(b)}, sys.getsizeof(b) = {sys.getsizeof(b)}')

c = dict()
for k in letters:
    c[k] = k
    print(f'{len(c)}, sys.getsizeof(c) = {sys.getsizeof(c)}')

  ```  
分别给三类可变对象添加 26 个元素，看看结果如何： 
![Test](https://oscimg.oschina.net/oscnet/u7y1GDwELoSaMrk.jpg  'Python 内存分配时的小秘密') 
由此能看出可变对象在扩充时的秘密： 
 
 超额分配机制： 申请新内存时并不是按需分配的，而是多分配一些，因此当再添加少量元素时，不需要马上去申请新内存 
 非均匀分配机制： 三类对象申请新内存的频率是不同的，而同一类对象每次超额分配的内存并不是均匀的，而是逐渐扩大的 
 
 
#### 3、列表不等于列表！ 
以上的可变对象在扩充时，有相似的分配机制，在动态扩容时可明显看出效果。 
那么，静态创建的对象是否也有这样的分配机制呢？它跟动态扩容比，是否有所区别呢？ 
先看看集合与字典： 
 ```java 
  # 静态创建对象
set_1 = {1, 2, 3, 4}
set_2 = {1, 2, 3, 4, 5}
dict_1 = {'a':1, 'b':2, 'c':3, 'd':4, 'e':5}
dict_2 = {'a':1, 'b':2, 'c':3, 'd':4, 'e':5, 'f':6}

sys.getsizeof(set_1)  # 224
sys.getsizeof(set_2)  # 736
sys.getsizeof(dict_1) # 240
sys.getsizeof(dict_2) # 368

  ```  
看到这个结果，再对比上一节的���图，可以看出：在元素个数相等时，静态创建的集合/字典所占的内存跟动态扩容时完全一样。 
这个结论是否适用于列表对象呢？一起看看： 
 ```java 
  list_1 = ['a', 'b']
list_2 = ['a', 'b', 'c']
list_3 = ['a', 'b', 'c', 'd']
list_4 = ['a', 'b', 'c', 'd', 'e']

sys.getsizeof(list_1)  # 80
sys.getsizeof(list_2)  # 88
sys.getsizeof(list_3)  # 96
sys.getsizeof(list_4)  # 104

  ```  
上一节的截图显示，列表在前 4 个元素时都占 96 字节，在 5 个元素时占 128 字节，与这里明显矛盾。 
所以，这个秘密昭然若揭：在元素个数相等时，静态创建的列表所占的内存有可能小于动态扩容时的内存！ 
也就是说，这两种列表看似相同，实际却不同！列表不等于列表！ 
![Test](https://oscimg.oschina.net/oscnet/u7y1GDwELoSaMrk.jpg  'Python 内存分配时的小秘密') 
 
#### 4、消减元素并不会释放内存！ 
前面提到了，扩充可变对象时，可能会申请新的内存。 
那么，如果反过来缩减可变对象，减掉一些元素后，新申请的内存是否会自动回收掉呢？ 
 ```java 
  import sys
a = [1, 2, 3, 4]
sys.getsizeof(a) # 初始值：96
a.append(5)      # 扩充后：[1, 2, 3, 4, 5]
sys.getsizeof(a) # 扩充后：128
a.pop()          # 缩减后：[1, 2, 3, 4]
sys.getsizeof(a) # 缩减后：128

  ```  
如代码所示，列表在一扩一缩后，虽然回到了原样，但是所占用的内存空间可没有自动释放啊。其它的可变对象同理。 
这就是 Python 的小秘密了，“胖子无法减重原理” ：瘦子变胖容易，缩减身型也容易，但是体重减不掉，哈哈~~~ 
![Test](https://oscimg.oschina.net/oscnet/u7y1GDwELoSaMrk.jpg  'Python 内存分配时的小秘密') 
 
#### 5、空字典不等于空字典！ 
使用 pop() 方法，只会缩减可变对象中的元素，但并不会释放已申请的内存空间。 
还有个 clear() 方法，它会清空可变对象的所有元素，让我们试试看吧： 
 ```java 
  import sys
a = [1, 2, 3]
b = {1, 2, 3}
c = {'a':1, 'b':2, 'c':3}

sys.getsizeof(a) # 88
sys.getsizeof(b) # 224
sys.getsizeof(c) # 240

a.clear()        # 清空后：[]
b.clear()        # 清空后：set()
c.clear()        # 清空后：{}，也即 dict()

  ```  
调用 clear() 方法，我们就获得了几个空对象。 
在第一小节里，它们的内存大小已经被查验过了。（前面说过会考的，请默写 回看下） 
但是，如果这时再去查验的话，你会惊讶地发现，这些空对象的大小跟前面查的并不完全一样！ 
 ```java 
  # 承接前面的清空操作：
sys.getsizeof(a) # 64
sys.getsizeof(b) # 224
sys.getsizeof(c) # 72

  ```  
空列表与空元组的大小不变，然而空字典（72）竟然比前面的空字典（240）要小很多！ 
也就是说，列表与元组在清空元素后，回到起点不变初心，然而，字典这家伙却是“赔了夫人又折兵”，不仅把“吃”进去的全吐出来了，还把自己的老本给亏掉了！ 
![Test](https://oscimg.oschina.net/oscnet/u7y1GDwELoSaMrk.jpg  'Python 内存分配时的小秘密') 
字典的这个秘密藏得挺深的，说实话我也是刚刚获���，百思不得其解…… 
以上就是 Python 在分配内存时的几个小秘密啦，看完之后，你是否觉得涨见识了呢？ 
你想明白了几个呢，又产生了多少新的谜团呢？欢迎留言一起交流哦~ 
对于那些没有充分解释的小秘密，今后我们再慢慢揭秘…… 
作者简介： 豌豆花下猫，生于广东毕业于武大，现为苏漂程序员，有一些极客思维，也有一些人文情怀，有一些温度，还有一些态度。公众号：「Python猫」（python_cat） 
![Test](https://oscimg.oschina.net/oscnet/u7y1GDwELoSaMrk.jpg  'Python 内存分配时的小秘密')
                                        