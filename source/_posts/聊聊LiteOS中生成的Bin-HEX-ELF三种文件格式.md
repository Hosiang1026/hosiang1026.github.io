---
title: 推荐系列-聊聊LiteOS中生成的Bin-HEX-ELF三种文件格式
categories: 热门文章
tags:
  - Popular
author: OSChina
top: 1946
cover_picture: 'https://pic1.zhimg.com/80/v2-99c9253171cf7e48197c36f357307cd0_720w.jpg'
abbrlink: a78e999b
date: 2021-04-15 09:46:45
---

&emsp;&emsp;摘要：我们在使用编译器在编译工程后会要求生成可执行文件，将这些文件烧录到MCU进行运行，达到我们测试和使用程序的目的，再使用工具链进行编译的时候往往生成.bin、.hex 、.elf 、.alf等文...
<!-- more -->

                                                                                                                                                                                         
本文分享自华为云社区《LiteOS 下载到MCU中的三种文件格式Bin、HEX、ELF》，原文作者：o0龙龙0o。 
我们在使用编译器在编译工程后会要求生成可执行文件，将这些文件烧录到MCU进行运行，达到我们测试和使用程序的目的，再使用工具链进行编译的时候往往生成.bin、.hex 、.elf 、.alf等文件，这些文件有什么区别呢？可以互相转换吗？LiteOS 有哪些可执行文件呢？本文意义进行阐述。 
 
#### BIN 
bin文件，是基本的二进制文件，是flash中IO保存的基本信息，是有汇编程序直接汇编得到的二进制代码，bin文件采用顺序记录flash中的信息，文本本身包含任任何地址信息，bin文件烧录就是指定flash开始地址后逐个拷贝即可。利用STM32CubeProm将LiteOS编译后生成的bin文件显示如下图，我们需要设定flash写入地址才能进行烧录。 
![Test](https://pic1.zhimg.com/80/v2-99c9253171cf7e48197c36f357307cd0_720w.jpg  '聊聊LiteOS中生成的Bin-HEX-ELF三种文件格式') 
 
#### HEX 
hex文件格式是可以烧写到单片机中，被单片机执行的一种文件格式，生成Hex文件的方式有很多种，可以通过不同的编译器将C程序或者汇编程序编译生成hex；最常用���Hex��式是Intel HEX文件格式，即遵循Intel HEX文件格式的ASCII文本文件，文件的每一行都包含了 一个HEX记录。这些记录是由一些代表机器语言代码和常量的16进制数据组成的。Intel HEX文件常用来传输要存储在ROM 或者 EPROM中的程序和数据。大部分的EPROM编程器和FLASH能使用Intel HEX文件。 
上面的Huawei_LiteOS.bin对应的HEX文件如下(用notepad++打开) 
 
  
 ```java 
  :020000040800F2
:2000000000000820F50E0008650F0008650F0008650F0008650F0008650F00080000000041
:20002000000000000000000000000000650F0008650F000800000000650F0008650F0008D0
.........................................................................
.........................................................................
.........................................................................
:208E0000D883050828830508D4820508148505081C8505082485050868CC03082C850508C8
:0C8E20003485050804CD030804CD0308C8
:00000001FF
  ``` 
  
 
文件会有头尾部的的说明。 
文件头部的信息 
 
  
 ```java 
  :020000040800F2
  ``` 
  
 
02带边数据长度；紧跟着后面的0x00 0x00 为地址；再后面的0x04为数据类型，类型共分以下几类： 
'00' //数据记录 '01' //文件结束记录 '02' //扩展段地址记录 '03' //开始段地址记录 '04' //扩展线性地址记录 '05' //开始线性地址记录 接着0x04后面的两个 0x08 0x00就是数据，表示偏移地址，最后一个0xF2是校验码。 
第二行开始的记录地址和所对应的数据其格式是 
：开始代码|地址|数据类型|数据|校验 
 
  
 ```java 
  :20|0000|00|00000820F50E0008650F0008650F0008650F0008650F0008650F000800000000|41
  ``` 
  
 
:20 记录数据长度为20个字节； 0000 数据在内中的起始地址 00 记录类型00(是一个数据记录) 00000820F50E0008650F0008650F0008650F0008650F0008650F000800000000 数据内容 41 这一行的校验 
最后一行的内容表示文件结束记录 
 
  
 ```java 
  :00000001FF
  ``` 
  
 
hex文件同一样可以在STM32CubeProm打印出内存的内容（与之前的bin打印是一致的）。 
 
#### ELF 
在计算机科学中，是一种用于二进制文件、可执行文件、目标代码、共享库和核心转储格式文件，是UNIX系统实验室（USL）作为应用程序二进制接口（Application Binary Interface，ABI）而开发和发布的，也是Linux的主要可执行文件格式。 
elf（Executable and Linkable Format）可执行与可链接格式，是有别于hex和bin通过记录数据的格式，elf更多而记录程序的连接转储的格式文件，elf目标文件是由汇编器(assembler)和连接编辑器(link editor)生成的，内容是二进制，而非可读的文本形式，是可以直接在处理器上运行的代码。 
简单的理解，elf文件将二进制（bin）文件和程序描述文件打包后的一种执行文件，下载到程序里的依然是bin文件的部分，但是仿真器可以依靠其余程序表述文件来获取程序执行的位置和二进制的对应。表意文件可以利用readelf在linux下读取，因为我系统的原因就不赘述了。 
其他可执行文件： 
.asf、.o、.out这些文件都是编译后的可执行文件，和elf以宣扬都是具有连接格式进行描述，可以利用仿真器进行仿真使用，只是编译格式和编译器设置的不同可以选择不同的文件格式。 
 
#### 可转换性 
因为bin、hex都是只是记录数据的，但elf类型不仅记录数据还有程序描述，所以，elf可以转成bin和hex使用，但是反转。 
对比一下，发现bin文件最小最简单，但是安全性差，功能性差，hex包含头尾和检验，就有很好的安全性，但是文件比bin大，功能没有elf强大；elf功能多，但是文件最大。 
 
#### LIteOS如何生成这些文件的 
liteOS通过makefile进行文件编译，也是通过makefile进行设置gcc编译文件的输出格式，在工程目录下的makefile代码中： 
 
  
 ```java 
  	$(LD) $(LITEOS_LDFLAGS) $(LITEOS_TABLES_LDFLAGS) $(LITEOS_DYNLDFLAGS) -Map=$(OUT)/$@.map -o $(OUT)/$@.elf --start-group $(LITEOS_BASELIB) --end-group
	$(OBJCOPY) -O binary $(OUT)/$@.elf $(OUT)/$@.bin
	$(OBJDUMP) -t $(OUT)/$@.elf |sort >$(OUT)/$@.sym.sorted
	$(OBJDUMP) -d $(OUT)/$@.elf >$(OUT)/$@.asm
	$(SIZE) $(OUT)/$@.elf
  ``` 
  
 
代码中的解释后的代码 
 
  
 ```java 
  arm-none-eabi-gcc  -o xx.elf
arm-none-eabi-objcopy -O ihex xx.elf xx.hex
arm-none-eabi-objcopy -O binary xx.elf xx.bin
out --format ihex write xx.hex
  ``` 
  
 
通过gcc编译的命令将结果生成为xx.elf的格式，在通过elf生成bin和hex的目标文件。 
 
#### 结论 
在使用工程编译结果是，最好有bin或者hex同时具有elf文件，elf用于仿真和调试，但输出的到工厂的文件可以使用hex和bin。 
  
点击关注，第一时间了解华为云新鲜技术~
                                        