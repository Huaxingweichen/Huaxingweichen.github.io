---
layout: post
title: "OCNet: Object Context Network for Scene Parsing笔记"
date: 2019-10-18 13:35:06 
description: ""
tag: Semantic Segmentation
---

`这篇文章思路跟感觉跟non-local[1]的很像,文章提到的观点与之前看过的文章并不是很一样，但是也是一个不错的想法，有点可惜没有中ICCV，CVPR。后来与师姐讨论了一下，觉得观点是不是真的可靠有待商榷。以下笔记基本是作者原文翻译，有我的理解以及我的判断，仅作为参考。`

### Abstract

> Motivated by that the label of a pixel is the category of the object that the pixel belongs to, we introduce an object context pooling (OCP) scheme, which represents each pixel by exploiting the set of pixels that belong to the same object category with such a pixel, and we call the set of pixels as object context.

被“一个像素的label类别一定是这个像素属于的物体的类别”这一点所启发，我们引入OCP机制，让每个像素被属于相同类别物体的像素集合所表示，我们称这个像素集合为物体的上下文关系。（翻译的会有偏差，建议读原文）作者之后说被self-attention所启发,提出了自己的实现步骤：

1. compute the similarities between each pixel and all the pixels, forming a so-called object context map for each pixel served as a surrogate for the true object context（计算每个像素和所有像素的相似性，填充一个so-called object context map作为true object context的替代品）
2. represent the pixel by aggregating the features of all the pixels weighted by the similarities.（通过融合所有像素的相似性权重特征来表示一个像素）

看到这地方感觉和non-local的区别就是是否有选择性，这篇OCNet是有选择性的选取像素之间的联系，而non-local是全部来，孰优孰劣个人感觉不好说，当然这篇文章肯定有experiments去prove自己好。
作者从理论上说比PSPNet[2]和AASP[3]这一种网络鲁棒性更好，因为这些网络没有选择是否跟这个需要表示的像素是一类的像素，所以reliability limited。

### Introduction

作者提到了目前有俩条路去应对分割所出现的问题。

- The first path is to raise the resolution of response maps for improving the spatial precision, e.g., through diated convolutions.

  第一种是提高response maps的空间精度（个人觉得有点类似于感受范围，可能我的术语还不到位），例如空洞卷积。

- The second path is to exploit the context for improving the labeling robustness, which our work belongs to.

  第二种是利用语义信息来提高鲁棒性，这也是这篇文章基于的思路。

之后作者说，现在的网络问题是mixture of pixels，所以标签预测的可靠度受限制。

> We compute a similarity map for each pixel p, where each similarity score indicates the degree that the corresponding pixel and the pixel p belongs to the same category. We call such similarity map as object context map, which serves as a surrogate of the true object context.

我们计算了每个像素p的相似map,每个相似分数预示着这个像素是否和像素p属于一个类别，我们叫这些相似map为object context map，作为真实物体语义信息的替代品。之后作者做了PSPNet和ASPP的拓展版。

### Related Work

- Semantic Segmentation

  作者列出了一些网络并且指出学长存在俩个主要的挑战：

  - resolution: there exists a huge gap between the output feature map’s resolution and the input image’s resolution.卷积池化会让图片分辨率变低。

  - multi-scale: there exist objects of various scales, especially in the urban scene images such as Cityscapes[4].物体的尺寸不一样，特别是在Cityscapes数据集上。

  之后写了各个网络对这些问题对解决办法。

- Context

  语义信息这边主要写了ParseNet,PSPNet,Deeplabv3是如何提取语义信息的，最后说我们不一样～

- Attention
  
  简述了self-attention机制的用途,以及语义分割方向用self-attention的一些文章。
  （有时间要看一下 attention is all your need[5], 这篇文章最近一直出现于我阅读的论文中，组会中老师也提及了）

### Approach

`这里我不按文章结构写了，主要是理解这个module`
![OCNet_structure.png](/images/posts/OCNet_structure.png)

$$W_{pi} = \frac{1}{Z_p}exp(f_q(X_p)^\mathrm Tf_k(X_i))$$

![OCNet_formula_explanation.png](/images/posts/OCNet_formula_explanation.png)

$$C_p = \sum_{i=1}^{N}w_{pi}\phi(X_i)$$

解释打latex公式太麻烦了，直接截图了。
这个东西吧，感觉有点像self-attention里面的 q k v ，但是我还没有看相关的文章，所以可能理解的不是很到位，我目前的理解是q和k应该是产生相关性矩阵,v应该是原来feature map的增强，其实就与这个对应起来了。$\phi(X_i)$就是v.
`好的，写到这里我突然发现了一个问题，这个跟non-local[1]的Embedded Gaussian好像没什么区别啊emmmmmmmm,个人感觉就是non-local的语义分割应用+redefine文章关注点。`

### Experiments

略

### Conclusions

> In this work, we present the concept of object context and propose the object context pooling (OCP) scheme to construct more robust context information for semantic segmentation tasks.
嗯，就这样。

### My References

[1] Wang X, Girshick R, Gupta A, et al. Non-local Neural Networks[J]. 2017.

[2] Zhao H, Shi J, Qi X, et al. Pyramid Scene Parsing Network[J]. 2016.

[3] Chen L C , Papandreou G , Schroff F , et al. Rethinking Atrous Convolution for Semantic Image Segmentation[J]. 2017.

[4] Cordts M , Omran M , Ramos S , et al. The Cityscapes Dataset for Semantic Urban Scene Understanding[J]. 2016.

[5] Vaswani A, Shazeer N, Parmar N, et al. Attention is all you need[C]//Advances in neural information processing systems. 2017: 5998-6008.
