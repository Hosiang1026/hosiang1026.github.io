---
title: 推荐系列-BIO-NIO-AIO 介绍和适用场景分析
categories: 热门文章
tags:
  - Popular
author: OSChina
top: 1731
cover_picture: 'https://static.oschina.net/uploads/img/202003/05105848_krWW.jpeg'
abbrlink: ad7357f8
date: 2021-04-15 09:19:21
---

&emsp;&emsp;IO的方式通常分为几种，同步阻塞的BIO、同步非阻塞的NIO、异步非阻塞的AIO。 一、同步阻塞的BIO 在JDK1.4之前，我们建立网络连接的时候采用BIO模式，需要先在服务端启动一个serverSocket，然...
<!-- more -->

                                                                                                                                                                                        IO的方式通常分为几种，同步阻塞的BIO、同步非阻塞的NIO、异步非阻塞的AIO。 
一、同步阻塞的BIO 
在JDK1.4之前，我们建立网络连接的时候采用BIO模式，需要先在服务端启动一个serverSocket，然后在客户端启动socket来对服务端进行通信，默认情况下服务端需要对每个请求建立一堆线程等待请求，而客户端发送请求后，先咨询服务端是否有线程相应，如果没有则会一直等待或者遭到拒绝请求，如果有的话，客户端线程会等待请求结束后才继续执行。 
![Test](https://oscimg.oschina.net/oscnet/up-5a7ea079ec1b41c3748e30df72fe90f3239.png  'BIO-NIO-AIO 介绍和适用场景分析') 
二、同步非阻塞的NIO 
NIO本身是基于事件驱动思想来完成的，其主要想解决的是BIO的大并发问题，在使用同步I/O的网络应用中，如果要同时处理多个客户端请求，或是在客户端要同时和多个服务器进行通讯，就必须使用多线程来处理。也就是说，将每一个客户端请求分配给一个线程来单独处理。这样做虽然可以达到我们的要求，但同时又带来另外一个问题。由于每创建一个线程，就要为这个线程分配一定的内存空间，而且操作系统本身对线程的总数有一定的限制。如果客户端的请求过多，服务端程序可能会因为不堪重负而拒绝客户端的请求，甚至服务器可能会因此而瘫痪。 
NIO基于Reactor，当socket有流可读或可写入socket时，操作系统会相应的通知引用程序进行处理，应用再将流读取到缓冲区或写入操作系统。也就是说，这个时候，已经不是一个连接就要对应一个处理线程了，而是有效的请求，对应一个线程，当连接没有数据时，是没有工作线程来处理的。 
BIO和NIO一个比较重要的不同，是我们使用BIO的时候往往会引入多线程，每个连接一个单独的线程；而NIO则是使用单线程或者使用少量线程，每个连接公用一个线程。 
![Test](https://oscimg.oschina.net/oscnet/up-5a7ea079ec1b41c3748e30df72fe90f3239.png  'BIO-NIO-AIO 介绍和适用场景分析') 
NIO的最重要的地方是当一个连接创建后，不需要对应一个线程，这个连接会被注册到多路复用器上，所以所有的连接只需要一个线程就能搞定，当这个线程中多路复用器进行轮询的时候，发现连接上有请求的话，才开启一个线程进行处理，也就是一个请求一个线程模式。 
在NIO的处理方式中，当一个请求来的时候，开启线程进行处理，可能会等待后端应用的资源（JDBC连接等），其实这个线程就被阻塞了，如果并发上来，还会有BIO一样的问题。 
HTTP/1.1出现后，有了Http长连接，这样除了超时和指明特定关闭的http header外，这个链接是一直打开的状态的，这样在NIO处理中可以进一步的进化，在后端资源中可以实现资源池或者队列，当请求来的话，开启的线程把请求和请求数据传送给后端资源池或者队列里面就返回，并且在全局的地方保持住这个现场(哪个连接的哪个请求等)，这样前面的线程还是可以去接受其他的请求，而后端的应用的处理只需要执行队列里面的就可以了，这样请求处理和后端应用是异步的.当后端处理完，到全局地方得到现场，产生响应，这个就实现了异步处理。 
三、异步非阻塞AIO 
![Test](https://oscimg.oschina.net/oscnet/up-5a7ea079ec1b41c3748e30df72fe90f3239.png  'BIO-NIO-AIO 介绍和适用场景分析') 
与NIO不同，当进行读写操作时，只需直接调用API的read或write方法即可。这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区，并通知应用程序；对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序。即可以理解为， read/write方法都是异步的，完成后会主动调用回调函数。   在JDK1.7中，这部分内容成为AIO， 
主要在java.nio.channels包下增加了下面四个异步通道： 
 
 AsynchronousSocketChannel 
 AsynchronousServerSocketChannel 
 AsynchronousFileChannel 
 AsynchronousDatagramChannel 
 
其中的read/write方法，会返回一个带回调函数的对象，当执行完读取/写入操作后，直接调用回调函数。   
BIO是一个连接一个线程。 
NIO是一个请求一个线程。 
AIO是一个有效请求一个线程。 
先来个例子理解一下概念，以银行取款为例：  
 
 同步 ： 自己亲自出马持银行卡到银行取钱（使用同步IO时，Java自己处理IO读写）； 
 异步 ： 委托一小弟拿银行卡到银行取钱，然后给你（使用异步IO时，Java将IO读写委托给OS处理，需要将数据缓冲区地址和大小传给OS(银行卡和密码)，OS需要支持异步IO操作API）； 
 阻塞 ： ATM排队取款，你只能等待（使用阻塞IO时，Java调用会一直阻塞到读写完成才返回）； 
 非阻塞 ： 柜台取款，取个号，然后坐在椅子上做其它事，等号广播会通知你办理，没到号你就不能去，你可以不断问大堂经理排到了没有，大堂经理如果说还没到你就不能去（使用非阻塞IO时，如果不能读写Java调用会马上返回，当IO事件分发器会通知可读写时再继续进行读写，不断循环直到读写完成） 
 
四、java对BIO、NIO、AIO的支持 
java BIO：同步并阻塞 
在此种方式下，用户进程在发起一个IO操作以后，必须等待IO操作的完成，只有当真正完成了IO操作以后，用户进程才能运行。JAVA传统的IO模型属于此种方式！ 
服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。 
java NIO：同步非阻塞 
在此种方式下，用户进程发起一个IO操作以后边可返回做其它事情，但是用户进程需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的CPU资源浪费。其中目前JAVA的NIO就属于同步非阻塞IO。 
服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。 
java AIO：异步非阻塞 
在此种模式下，用户进程只需要发起一个IO操作然后立即返回，等IO操作真正的完成以后，应用程序会得到IO操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由内核完成了。目前Java中还没有支持此种IO模型。  
服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是有OS先完成了再通知服务器应用去启动线程进行处理。 
![Test](https://oscimg.oschina.net/oscnet/up-5a7ea079ec1b41c3748e30df72fe90f3239.png  'BIO-NIO-AIO 介绍和适用场景分析') 
五、BIO、NIO、AIO适用场景分析 
BIO方式适用于连接数目比较少且固定的架构，这种方式对服务器资源要求���较高，并发局限于应用中，JDK1.4以前的唯一选择，程序只管简单易理解。 
NIO方式适用于连接数目多且比较短的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。 
AIO方式适用于连接数目多且连接比较长的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK1.7开始支持。 
六、 Tomcat（ BIO ）和Jetty（NIO） 
Tomcat和Jetty是目前全球范围内最著名的两款开源的webserver/servlet容器。 
相同点： 
1、Tomcat和Jetty都是一种servlet引擎，他们都支持标准的servlet规范和JavaEE的规范。 
不同点： 
1、架构比较 
（1）Jetty的架构比Tomcat简单。 
（2）Jetty的架构是基于handler来实现的，主要的扩展功能都可以使用handler来实现，扩展简单。 
（3）Tomcat的架构是基于容器设计的，进行扩展是需要了解Tomcat的整体设计结构，不易扩展。 
2、性能比较 
（1）Jetty和Tomcat性能方面差异不大。 
（2）Jetty可以同时处理大量连接而且可以长时间保持连接，适合web聊天应用等等。 
（3）Jetty的架构简单，因此作为服务器，Jetty可以按需加载组件，减少不必要的组件，减少了服务器内部开销，从而提高服务器性能。 
（4）Jetty默认采用NIO结束处理I/O请求上更占优势，在处理静态资源时，性能较高。 
3、Tomcat适合处理少数非常繁忙的链接，Tomcat的总体性能更高。Tomcat默认采用BIO处理I/O请求，在处理静态资源时，性能较差。 
4、其他比较 
（1）Jetty的应用更加快速，修改简单，对新的servlet规范的支持较好。 
（2）Tomcat目前应用比较广泛，对javaEE和servlet的支持更加全面，很多性能会直接集成进来。 
  
本博主已经全面进军CSDN，希望大家支持我一下，非常感谢！ 
【CSDN】BIO、NIO、AIO 介绍和适用场景分析（绝对经典） 
 
                                        