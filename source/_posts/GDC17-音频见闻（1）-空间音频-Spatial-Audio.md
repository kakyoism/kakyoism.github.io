---
title: GDC17 音频见闻 - 空间音频 Spatial Audio（1）
date: 2017-03-13 13:50:33
categories: 

- GameAudio
tags: conference, gdc17, reverb, obstruction-occlusion, spatialaudio
---

## 《Gears of War 4》中的环境声学预处理技术

继 Binaural 和 Ambisonics 之后，今年 GDC 上的空间音频热门关键字多了*混响*（Reverberation）、*声障和声笼*（Obstruction & Occlusion）这些环境声学概念，标志着对沉浸音频体验发起的又一次冲锋。这些游戏音频界的旧瓶里最受瞩目的新酒之一当数微软研究院和《战争机器4》（GoW4）开发组的 [Project Triton](http://schedule.gdconf.com/session/gears-of-war-4-project-triton-pre-computed-environmental-wave-acoustics)，即通过预先演算声波传播物理参数来辅助人工设计的环境声学效果方案。本次的演讲实际上是微软已启动六年之久系列研究的首次产品化成果。演讲幻灯片[已放出](http://www.nikunjr.com/Projects/Triton/Triton-GDC2017.pptx)。以下尝试稍加总结。

### 成果

这乍一看很学术范的技术主要是为了解决游戏环境声学设计中的一大痛点：传统设计方法依赖在游戏地图中手工标注或绘制声学区域，再挂接音频效果器来实现室内混响和声障／声笼效果；音频中间件如 Wwise 的效果器控制参数一般是够用的，但在游戏中手工标注的工作量巨大且容易出错，实际开发工期紧，很难打磨调优。

不巧，玩家极易通过生活经验察觉这类设计错误，后果严重，参见《刺客信条：大革命》中曾出现的[全局 bug](http://forums.ubi.com/showthread.php/973716-AC-Unity-Sound-Bug-Forums)。音频设计界早就希望能像物理渲染界一样自动化这个过程，为创意设计赢得时间。

演讲中放出的*最终*解决方案：

{% asset_img sa-triton-game-integration.jpg Triton 系统整合示意图 %}

- 离线预处理：在游戏地图中自动选取采样位置，对各个位置上的发声体-听者结对（emitter-listener pair）根据几何数据自动做 3D 声波传播模拟并生成原始声学数据库；
- 解析：从原始声学数据自动提炼出一组感知参数值：直达波能量、反射波能量、反射波衰减率（wetness）和混响衰减率；
- 接入音频中间件：运行时通过查询数据库获取感知参数值，用来控制 Wwise 中的声障／声笼滤波器和辅助总线上的混响效果器，产生实时互动的声学效果。

用这种方法，设计师不再需要手动标注环境声学区域，可以专注于艺术效果设计，即通过感知参数来控制音频效果输出。结果不但错误少，且游戏中的实时性能也达到甚至优于项目初始要求。

一些细节：

- 声障值和声笼值分别用初始能量（直达声比例）和反射能量来代表；
- 区分室外和室内情况，方法是通过在室外设置理想边界，并测量从玩家位置发声后能抵达这条边界的能量占总能量的比例来推算*室内-室外比*，以此来做到平滑过渡；
- 原始声学数据计算用 100 台机器需要约 4 小时计算时间，初始数据为 50 TB，做解析后降为 100 MB。

### 经历

很多学术报告大概也就到此为止了，让一部分听众感觉高山仰止，另一部分不知所云。然而这次两位主讲人–微软研究院的 Nikunj Raghuvanshi 和 Coalition 工作室的 John Tennant –继续披露了研发和产品化的详细经过，以机器人领域的[*恐怖谷*](http://baike.baidu.com/item/恐怖谷理论)现象为纲，讲述了从真实到艺术真实的探索（微信游戏音频群里尾巴老师语），体现了对基础研究和产品化过程深刻的理解，是演讲中我个人最喜欢的部分。

{% asset_img  sa-triton-uncanny-valley.jpg %}

从 2011 年的 V1.0 到 2014 年的 V2.0 直到最终版，项目经历了标准的恐怖谷历程：

1. V1.0 中初尝预处理甜头后，发现一些问题亟待解决，比如区分室内外和消除混响效果中的浑浊；
2. 于是寄希望于自动化和仿真路线，火力全开，将模型复杂化：从 FDN 混响换成卷积混响，声学数据因此变成冲激响应，在增加室外效果器组分支的同时还增加了对早期反射和后期混响的区分，导致要控制 12 个混响单元；
3. 不幸跌入恐怖谷，为了自救而后退一步，着重解决艺术真实问题，最后简化了模型，按时保质完成了产品化。

一些真实和艺术真实的折衷处理：

- 冲激响应路线很难控制质量：录音和测量人员不同，因此高度依赖配准，为后期调试带来困难，终弃；
- 混响效果受输入声音动态范围影响，将输入的*真实*动态范围做了艺术限制之后，清晰度得到提升；
- 与传统影视建立的艺术效果标准相比，物理计算中得出的直达声能量以及衰减时间，结合游戏具体玩法后，会出现不合心理期望的情况，终弃。

有意思的是，上面的道理很多其实是马后炮：是在主动作出简化混响模型的决定后从实践中领悟到的，又一次说明奥卡姆剃刀原则的普适性。

### 未来

Triton 项目的未来计划包括在预处理模型中加入：

- 直达分量方向性、早期反射、户外回声；
- 动态几何结构，比如活动的门窗和物理毁坏。

巧的是，Triton 和 Wwise 不谋而合，这几条正是 GDC17 上 Wwise 2017.1 版本展出的新 Spatial Audio 功能的一部分，请看下篇。

### 相关资料

- 演讲幻灯片: ['Gears of War 4', Project Triton: Pre-Computed Environmental Wave Acoustics](http://www.nikunjr.com/Projects/Triton/Triton-GDC2017.pptx)
- Engadget 文章：[Microsoft Research helped 'Gears of War 4' sound so good](https://www.engadget.com/2016/10/25/gears-of-war-4-microsoft-research-triton/)
- Triton V1.0 论文：[Raghuvanshi, et. al., SIGGRAPH 2010, “Precomputed wave simulation for real-time sound propagation of dynamic sources in complex scenes”
](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/10/6.pdf)
- Triton V2.0 论文：[Raghuvanshi & Snyder, SIGGRAPH 2014, “Parametric wave field coding for precomputed sound propagation”
](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/ParametricWaveField.pdf)
