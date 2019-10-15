---
layout: post
title: "Pyramid Scene Parsing Network笔记"
date: 2019-10-15 18:48:06 
description: ""
tag: Semantic Segmentation
---

`PSPnet是师姐推荐的第二篇论文，讲道理阅读难度比FCN那篇开山之作小不少（个人感觉），继续肝～`

### Abstract

>In this paper, we exploit the capability of global context information by different-region-based context aggregation through our pyramid pooling module together with the proposed pyramid scene parsing network (PSPNet).

一句话说清了文章在干什么，然后说自己的网络怎么怎么好～

### Introduction

开头简单介绍场景解析任务，然后提出
>Difficulty of scene parsing is closely related to scene and
label variety.

语义分析的难度主要跟标签的种类多少有直接关系，简单介绍完之后开捶FCN，提及FCN对于全局信息的把握不够好，例如下图中的房子旁边的船，FCN认为它是车子，就是因为没有看环境的结果。

>We found that the major issue for current FCN based models is lack of suitable strategy to utilize global scene category clues.

![pspnet对比图](/images/posts/PSPnet_comparation1.png)

最后给出了自己的三点贡献：

- We propose a pyramid scene parsing network to em- bed difficult scenery context features in an FCN based pixel prediction framework.

- We develop an effective optimization strategy for deep ResNet based on deeply supervised loss.

- We build a practical system for state-of-the-art scene parsing and semantic segmentation where all crucial implementation details are included.

### Related Work

作者主要提了继承FCN的工作，继续往下走，同时提到了其他工作的俩个方向。

- One line is with multi-scale feature ensembling. Since in deep networks, higher-layer feature contains more semantic meaning and less location information. Combining multi-scale features can improve the performance.

- The other direction is based on structure prediction. The
pioneer work used conditional random field (CRF) as post processing to refine the segmentation result.

第一条路多尺度，尽量获得不同尺度的信息进行组合。第二条路暂时没有看过相关的文章，感觉有点像是修正结果的那种，就像是目标检测任务中对box进行修正。

### Pyramid Scene Parsing Network

重点来了。

- Important Observations
作者先观察了现存的问题，再对现存问题进行了分析，提出解决方案，解决问题。

  - Mismatched Relationship
  
    不同的物体因为一定其他的物体的存在可能会收到类别影响，例如那只很像车的船，如果不是在河流上，真的可能以为是车子。

  - Confusion Categories

    物品的类别太多了field and earth; mountain and hill; wall, house, building and skyscraper这些东西都不是很好区分。

  - Inconspicuous Classes

    小物体/不引人注意的物体的识别可能会被忽略，惨兮兮的FCN又被拿来暴捶。

    ![pspnet对比图](/images/posts/PSPnet_comparation2.png)

    枕头的那个例子。

- Pyramid Pooling Module

    提完了现在网络的不足，作者开始摆解决方案了，也就是大名鼎鼎的池化金字塔。
    ![池化金字塔结构](/images/posts/PyramidPoolingModule.png)
    我看了PSPnet的代码，感觉池化金字塔（如果是基于resnet）就像一个小尾巴，尾巴分为四个尺寸最小的那个尾巴提取最粗的特征，更像是全局最重要的特征，以此类推，再与resnet卷积出来的特征结合。

### Deep Supervision for ResNet-Based FCN

这个地方是说多加了一个损失函数，对训练有帮助，具体为什么以及损失函数的机制不是很明白，篇幅也比较少，先占坑，我看看相关解读再补上。

### Experiments

实验自己看哈～
