---
layout: post
title: "CCNet: Criss-Cross Attention for Semantic Segmentation笔记"
date: 2019-10-16 16:23:06 
description: ""
tag: Semantic Segmentation
---

`CCnet的点睛之笔在于Criss-Cross Attention，这个操作十分漂亮，一个词形容就是事半功倍，主要的贡献是干掉了non-local module(proposed by Kaiming He 膜).[1]`

### Abstract

首先，作者提出了自己的追求的目标。

> Long-range dependencies can capture useful contextual information to benefit visual understanding problems.

Long-range dependencies 我的理解是俩个像素可能相隔很远但是依旧有依赖关系，这种对于基于卷积的语义分割网络来说并不是一个好的情况，因为卷积更多是local的，后面作者也谈到有人做一定的努力，在Introduction的部分我会详细的写到，这篇文章的Introduction和Related work个人感觉还是不错的，至少让我这种刚入门的菜鸡了解到了很多思路，之后作者说到自己的提出的Module可以更好的提取上下文关系，同时让一个像素最后和所有的像素有关。

作者列出了一些优点：

- GPU memory friendly. Compared with the non-local block, the recurrent criss-cross attention module requires 11× less GPU memory usage. （减少显存，开始捶non-local了）

- High computational efficiency. The recurrent criss-cross attention significantly reduces FLOPs by about 85% ofthe non-local block in computing long-range dependencies.（减少计算量，事半）

- The state-of-the-art performance. We conduct extensive experiments on popular semantic segmentation benchmarks including Cityscapes, ADE20K, and instance segmentation benchmark COCO. In particular, our CCNet achieves the mIoU score of 81.4 and 45.22 on Cityscapes test set and ADE20K validation set, respectively, which are the new state-of-the-art results. （表现也还不错，功倍）

### Introduction

首先说语义分割很火很火，有一些用途，然后开始说有问题，主要在于基于FCN的模型有天生的局限性,对于提取上下文信息有一定的限制。

>Due to the fixed geometric structures, they are inherently limited to local receptive fields and short-range contextual information. These limitations impose a great adverse effect on FCN-based methods due to insufficient contextual information.

然后开始说有人提出了一些解决办法，但是呢，都有局限性，主要这里提到了俩种解决办法。

- dilated convolution 空洞卷积：局限性在就算是空洞卷积，但是想把握全局依旧很难，逃脱不了卷积的固有特性（local）

- pyramid pooling 池化金字塔：池化金字塔确实可以提取到全局特征了，但是，但是，但是，本文作者认为，这并不符合我们想要的每个像素跟特定的像素有联系的需求，池化金字塔有点像是一锅端，但是我们想要的是特定的一些像素之间有一定的联系。

然后又开始捶non-local net了，说这个网络虽然确实让每一个像素与其他像素都又联系，但是算法复杂度非常高，计算量和显存占用都挺吓人都，于是作者开始解决这个问题，有没有一种方法可以事半功倍呢？于是提出了criss-cross attention module十字交叉注意力模块，这个东西把算法复杂度从O((H×W)×(H×W)) 降到了 O((H×W)×(H+W−1))，基本上相当于加快了(HxW)/(H+W-1)倍,而且显存友好。所以在Introduction的最后，作者总结到本文到贡献主要是：

- We propose a novel criss-cross attention module in this work, which can be leveraged to capture contextual information from long-range dependencies in a more efficient and effective way.

- We propose a CCNet by taking advantages of two recurrent criss-cross attention modules, achieving leading performance on segmentation-based benchmarks, including Cityscapes, ADE20K and MSCOCO.

总结起来就是，我们提出了十字交叉注意力机制，机制有效，因为该机制，所以我们的网络表现很好。

### Related work

- Semantic segmentation

首先说明了语义分割的一些有代表性网络的机制，然后说明了一些网络对于上下文信息提取所做出了一些努力和尝试，这里面总结的挺多的有代表性的网络，对于刚进入这个领域的小白来说还是能收获不少的，建议可以仔细看一下。

- Attention model

这里面也主要说的是语义分割的一些注意力模型。

### Approach

- Overall

![CCNet structure](/images/posts/CCNet_structure.png)
先放网络结构，实际上CCnet,基本相当于在卷积提取特征之后（用的resnet），加上了一个自己的head,也就是所谓的十字交叉注意力机制/module。

- Criss-Cross Attention

`划重点`

![criss-cross attention](/images/posts/criss-cross_attention.png)

这个地方我要说清要花好久，但是知乎上有位大哥写的挺好的，我这里直接放上连接，看的会很明白。
[CCNet--于"阡陌交通"处超越恺明Non-local](https://zhuanlan.zhihu.com/p/51393573),这个想法真的相当棒，但是计算时间可能会比较久，就是Q和K算相关矩阵的那一块，华科大哥手撸cuda代码也是强。

- Recurrent Criss-Cross Attention

这个东西我的理解就是过俩遍criss-cross attention module,因为过一次每个点跟他的十字区域有关，过俩次每个点就跟全部的像素有关了！

### Experiments

略

### Conclusion and future work

我们本文做了什么，有什么效果，criss-cross attention module显存少，计算量少，在各个数据集上结果很好。

`future work我怎么没有看到......疯狂吐槽，我找了几遍还是没有看到 future work`

[1]Wang X, Girshick R, Gupta A, et al. Non-local Neural Networks[J]. 2017.

`引用这一块我写的不是很规范，下次文章尽量注意！`
