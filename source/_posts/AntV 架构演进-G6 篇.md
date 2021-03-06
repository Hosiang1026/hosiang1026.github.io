---
title: 推荐系列-AntV 架构演进-G6 篇
categories: 热门文章
tags:
  - Popular
author: OSChina
top: 1715
cover_picture: 'https://static.oschina.net/uploads/img/202003/05105913_xVc4.jpeg'
abbrlink: 9786661f
date: 2021-04-15 09:19:21
---

&emsp;&emsp;本文作者：AntV 架构师-萧庆 简介 G6 是一个图关系可视化引擎，起始于我们的业务需求，历经波折，每次改版其架构都有很大的变化，这些变化背后都有来自业务上的思考和我们对 G6 定位的调整，...
<!-- more -->

                                                                                                                                                                                        ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
 
#### 简介 
G6 是一个图关系可视化引擎，起始于我们的业务需求，历经波折，每次改版其架构都有很大的变化，这些变化背后都有来自业务上的思考和我们对 G6 定位的调整，今天我们一起来回顾： 
 
 G6 之前的关系可视化 
 V1.0 关系映射 
 V2.0 图编辑器 
 V3.0 图分析引擎 
 
G6 发展的时间线如下： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
#### G6 之前的关系可视化 
早在做 G2 之前我们就接触了集团内部一些关系图的项目，以安全和风控的业务为主，也有一些动态的流程图，但是团队迟迟没有决定编写一套关系图框架，很大的一个原因在于：有太多失败的关系图项目。 
往往是项目一开始得到各个方面的大力支持，我们配合设计师做了一套好看炫酷的关系图展示页面，初期开发者、设计者都很满意，但是真正的使用者依然解决不了问题，大都类似于这类图： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
一方面用户很难完成业务上的任务，看起来好看但是不好用，另一方面使用的技术栈很零散，一旦我们退出这个项目，后期基本处于维护乏力的状况。 
当我们开始做 G2 后，需要在 G2 中实现一些关系图： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
这时候很多部门的开发同学希望我们在 G2 中也能支持流程图，例如： ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
但是 G2 的架构来做关系图的展示还行，增加很多交互、支持各种布局、各种复杂的关系节点，那就非常不合适了，恰巧当时发生其他两件事，使我决定来开发一个关系图框架： 
 
 第一件事是当时一个业务上面临数千个流程图的定义，不同的商家需要不同的流程，业务上的开发没有自动布局的基础，他们只能用代码一个个的来标注 x,y 坐标绘制流程图，想想都心疼。 
 第二件事是一个业务方之前使用了 jointjs 进行开发，中间更换了各种流程图框架，后面维护和开发不下去了，他们写了一封几千字，包括很多张图的邮件给我。 
 
作为一个可视化的开发人员面临这种情况让我坐立不安，我找到玉伯去申请开始研发一个新的关系图的框架，经过大家讨论后起名为 G6（六度分离理论） 
#### v0.1-v1.0 关系映射 
通过对几个关系图业务的支持，我发现静态的关系图对业务上几乎没价值，大部分场景都是用户有业务数据，需要自动的布局方案来展示出关系图（流程图），但是我们提供的布局方案往往满足不了业务上的需求，所以我们在 G6 的第一个版本中，对关系图的业务做了一个假设，用户有两个关系图视图： 
 
 第一个是编辑视图，可以自动布局，然后辅助手工布局，让关系图更美观好看 
 第二个展示视图，呈现给最终用户的视图，可以做一些简单的交互（展开、收缩、隐藏边等） 
 
所以 G6 v1.0 是沿着 G2 的思路，以关系映射为核心，支持常见的布局算法，支持简单的拖拽等交互，其架构如下： ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
 
 扩展 Shape 的机制保持同 G2 一致 
 所有的交互通过 mixin 内置，不可更改 
 
第一个版本开发了 3 个月，后面又迭代了 1.1 和 1.2 两个版本，持续了 8-9 个月，实现了常见的树图、桑基图、流程图，也实现了一些动态效果，集团内部迅速增长了一批用户，当时的版本缺乏设计师的参与，可以看到基本都是程序员的审美： ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
这一代的 G6 更像是 G2 的一个变种,通过下面的代码就可以看出： 
 ```java 
  graph.source(data.nodes, data.edges); 
graph.node().label('name').color('type'); graph.render();

  ```  
这个版本我们持续开发的时间并不长，但是至今为止依然有数百个系统使用这个版本，没有升级到最新的版本。我的理解是：很多业务场景下的流程图并不复杂，变化的频率并不高，简单、易用、不需要复杂布局的一个精简版本的关系图就够用了。 
#### v2.0 图编辑器 
作关系图躲避不了拖拖拽拽的编辑交互，而编辑交互又是极其复杂的： 
 
 添加节点/边的方式有多种，节点可以编辑、改变大小、移动、删除等 
 选中有多种形式，边需要在节点调整是自动布局 
 一个节点有多个锚点，每个锚点的含义都不同，不同的情况下可以禁用，也可能禁止连入等等 
 
业务上的需求使得我们无处可躲，实现这些交互在 G6 v1.0 的体系下根本不可能，所以我们开始了 2.0 的重构，于 18 年 1月开始了 2.0 的开发，6月份对外开源。这个版本增加了锚点机制、控制点机制，同时支持自定义交互（Behavior）, 这个版本的架构如下： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
 
 插件机制提供了布局、组件等支持 
 自定义交互（Behavior），可以插入式的实现交互，这一方式一直到现在依然保持 
 锚点和控制点是专门针对编辑场景设计的机制 
 
这个版本开发完成后，一些常见的编辑场景都得以实现： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇')![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇')![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
但是我们马上发现用户的接入成本非常高，这个只能算半个编辑器，用户进行添加节点/边、编辑节点、拖拽等操作时需要工具栏、各种面板，需要前进、后退、复制、黏贴等操作，同时上面的界面用户纷纷表示太难看。 
为了解决这些问题，我们找到设计师一起同我们对常见的流程图（流程图、建模、脑图）进行了设计，每种流程图的交互都进行了精心的设计，默认给用户提供了流程图编辑器需要的各种模块，形成了 G6-Editor，其架构非常简单： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
我们来看一看 G6-Editor 的界面和操作： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇')![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
G6-Editor 我们于 2018 年下半年对外推出（未开源，但允许商用），以其良好的设计、开箱即用的接入体验收到非常多的公司商用申请，我们本来计划对这几个场景进行更精细的设计、更精细的实现，但是我们太小瞧编辑器的难度。 
从我们内部业务的反馈来看，这些编辑器减轻了首次开发的成本，使得接入更加简单，但是由于交互和设计我们都已经固定，用户慢慢的就开始增加各种编辑的交互，例如：如何控制边的连接方式、某些锚点禁用的时机、边的布局方式不满足需求等等，更加雪上加霜的是维护 G6 和 G6-Editor 的只有一个同学，面对内部500+ 的系统，外部数百封邮件，经过半年痛苦的答疑、业务支持、改进交互，这位同学最终选择了转岗，最终我们决定 G6-Editor 编辑器不再开源😂，暂缓发展，这是过去两年中非常大的一个遗憾。（当然今年 2020 年 3 月我们还会给大家带来惊喜）。 
#### v3.0 图分析 
时间来到 2018 年底，G6 2.0 的同学离开后，我同一位新同学开始了 G6 3.0 的开发，其业务背景是越来越多的非编辑场景（分析场景）使得 G6 2.0 支持起来非常吃力，通过关系图发现异常、进行关系扩散以及对流量进行可视化等场景都是一些重要的业务。使得我们开始 3.0 开发时侧重于分析场景进行了设计，对布局、组件和节点/边的状态管理进行更多的设计，更加侧重展示更多节点和边的性能，而锚点和控制点机制在这些场景太重（很少使用），所以在3.0 最初的版本并没有进行设计，而这一版的架构如下： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
我们两个人投入了不到 2 个月的时间开发出了第一版，几个重要的业务在短短的2个月内上线： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') ![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
19 年初和年中又有多名同学参与了 G6，业务上的同学结合自己在关系分析方面的经验和实践，在 G6 的上层开发了一套开箱即用的工具 Graphin  
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
有流量分析场景的业务方同学也基于流量分析和时序分析，做出了精彩的产品： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
在过去的一年中 G6 v3.0 发展了 3 个版本，最新的 3.3 版本，进行了底层引擎替换，同时全面拥抱了 typescript，已经在年前发布了 beta 版，2020-02-11 日正式对外发布。 
我们的脚步不会停止，在新的一年我们将开始 G6 4.0 的设计和开发，将对其组件、交互和计算进行大刀破斧的改造，提升其交互能力、性能、布局能力，同时在关系图的时序可视化、地理可视化方面进行探索，下面是其架构： 
![Test](https://oscimg.oschina.net/oscnet/up-dbfc1351fe0ae8316a203cce9c7dc0087c4.JPEG  'AntV 架构演进-G6 篇') 
#### 总结 
参与 G6 的这几年，跌宕起伏，遇到过各种困难，面临过各种挫折。虽然架构一直在调整��但我们一直在努力的改进 ，给业务、给社区带来更多的可能。感谢集团对外所有 G6 用户的支持，感谢这几年被我们的不成熟导致的不兼容升级所痛的小伙伴们，同时向那些试用了 G6-Editor 但因为我们无法开源而所烦恼的用户们致歉。 
我们的步伐不会停止，知源致远！ 
G6 官网地址：https://g6.antv.vision/zh 
G6 的 github 地址：https://github.com/antvis/g6
                                        