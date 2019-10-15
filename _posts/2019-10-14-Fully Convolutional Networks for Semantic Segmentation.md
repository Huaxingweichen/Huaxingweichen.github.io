---
layout: post
title: "Fully Convolutional Networks for Semantic Segmentation 笔记"
date: 2019-10-14 19:58:06 
description: ""
tag: Semantic Segmentation
---

正在入门语义分割领域，如果有写的不对的地方，欢迎指证。

`Fully Convolutional Networks for Semantic Segmentation`是里程碑意义的工作，其论文个人感觉写的很细，值得研究，因此撰写笔记。

### Abstrct

摘要大致介绍了关于卷积的背景和本文的贡献，并且大致介绍了文章内容以及结果。

>Our key insight is to build “fully convolutional” networks that take input of arbitrary size and produce correspondingly-sized output with efficient inference and learning.

作者在这里点出FCN是接受任意尺寸的输入，并且产生一致输出的网络。

>We then de- fine a novel architecture that combines semantic informa- tion from a deep, coarse layer with appearance information from a shallow, fine layer to produce accurate and detailed segmentations.

作者意图结合深层抽象信息和浅层粗糙的信息让网络学的更好，但是FCN对于全局信息的把握稍差，后面的PSPnet通过池化金字塔机制对其进行了一定的提高。[1]

### Introduction

Introduction中大致介绍了卷积的背景，以及语义是自然而然的发展结果，之后就开吹自己了，主要总结如下：

>To our knowledge, this is the first work to train FCNs end-to-end (1) for pixelwise prediction and (2) from super-vised pre-training.

- end-to-end
  - 端到端我的理解是跟x-steps对应，即数据进去结果出来，不需要人为做更多的工作，一切变换都靠网络完成。在工程中，可能分步走更容易拿到实验结果。端到端的优点是：1、避免各个模块训练目标不一致的缺陷。2、减少工程复杂度。（only工程复杂度，不是说做的更快）缺点是：1、灵活性低2、训练黑盒，可解释性差。[参考知乎](https://www.zhihu.com/question/349900338/answer/851434329)

- pixelwise prediction
  - >In-network upsampling layers enable pixelwise prediction and learning in nets with subsampled pooling.
  
  - 像素级别的预测，语义分割是对每一个像素的归类进行预测。

- supervised pre-training
  - 这里说的是将分类网络重新解释为全卷积网络，进行fine tune,文章还提到了有人的工作结合了卷积网络但是没有fine tune.

> Semantic segmentation faces an inherent tension between semantics and location: global information resolves what while local information resolves where.

- 之后作者抛出了都知道但是很难做到的judgement。

### Related work

主要说了关于FCN和Dense prediction with convnets Several的一些工作。

### Fully convolutional networks Each

- Adapting classifiers for dense prediction

![FCN structure](/images/posts/FCN_structure.png)
分类网络的最后几层全连接网络换成卷积了，依旧包含空间信息。

- Shift-and-stitch is filter rarefaction

这一段比较复杂，而且作者最后并没有用。

- Upsampling is backwards strided convolution Another

这一块说了反卷积，关于反卷积的原理可以参考这篇文章[反卷积详细推导]('https://zhuanlan.zhihu.com/p/48501100'),我个人的理解是，卷积操作如果被看成

$$Y = CX$$

> Y 是卷积得到的矩阵，C是卷积矩阵，X是原始卷积区域

反卷积的目的是由Y得到X，所以关键条件是C。但是讲道理我们是不可能得到C的，因为C是卷积核在X上卷积而产生的，所以我们要假装有一个X，但是我们手上只有Y，所以我们对Y进行扩增（间隔补0），然后在Y上卷积得到C，再通过C将Y变成X。但是注意，反卷积并不能得到完全一样对结果，毕竟补0卷积再倒退对操作对结果肯定有一定的影响，我上面推荐的文章有图，讲的很详细，但是文章有一点小错误。

- Patchwise training is loss sampling

这个地方看的不是很明白emmmmmm,不急，过段时间再看看

### Segmentation Architecture

这个地方主要说了网络的详细结构，英语还是有点卡我阅读，幸好由师姐之前做的笔记。
这个地方的思想其实就是在信息提取的不同层次上都关注信息以获取到更好到效果，借用原文的一句话就是。
> Combining what and where
作者做了多个版本，最终在8x版本上效果最好，原文的图其实做了直观的说明，但是作者具体是怎么实现的可能看一下代码会更清楚。从下面的代码里面可以比较轻松的看到实际上他是在不断的合并，再不断的反卷积。

``` python
def forward(self, x):
        h = x
        h = self.relu1_1(self.conv1_1(h))
        h = self.relu1_2(self.conv1_2(h))
        h = self.pool1(h)

        h = self.relu2_1(self.conv2_1(h))
        h = self.relu2_2(self.conv2_2(h))
        h = self.pool2(h)

        h = self.relu3_1(self.conv3_1(h))
        h = self.relu3_2(self.conv3_2(h))
        h = self.relu3_3(self.conv3_3(h))
        h = self.pool3(h)
        pool3 = h  # 1/8

        h = self.relu4_1(self.conv4_1(h))
        h = self.relu4_2(self.conv4_2(h))
        h = self.relu4_3(self.conv4_3(h))
        h = self.pool4(h)
        pool4 = h  # 1/16

        h = self.relu5_1(self.conv5_1(h))
        h = self.relu5_2(self.conv5_2(h))
        h = self.relu5_3(self.conv5_3(h))
        h = self.pool5(h)

        h = self.relu6(self.fc6(h))
        h = self.drop6(h)

        h = self.relu7(self.fc7(h))
        h = self.drop7(h)

        h = self.score_fr(h)
        h = self.upscore2(h)
        upscore2 = h  # 1/16

        h = self.score_pool4(pool4)
        h = h[:, :, 5:5 + upscore2.size()[2], 5:5 + upscore2.size()[3]]
        score_pool4c = h  # 1/16

        h = upscore2 + score_pool4c  # 1/16
        h = self.upscore_pool4(h)
        upscore_pool4 = h  # 1/8

        h = self.score_pool3(pool3)
        h = h[:, :,
              9:9 + upscore_pool4.size()[2],
              9:9 + upscore_pool4.size()[3]]
        score_pool3c = h  # 1/8

        h = upscore_pool4 + score_pool3c  # 1/8

        h = self.upscore8(h)
        h = h[:, :, 31:31 + x.size()[2], 31:31 + x.size()[3]].contiguous()

        return h
```

`笔记不写不知道，一写吓一跳，文章还有很多地方没有明白，又看了一下，还是有地方不是很清楚，打算先往后看，以后再回头看看，刚进入语义分割领域，很多地方可能不太对，希望大家不令赐教。`

[1] Zhao H, Shi J, Qi X, et al. Pyramid Scene Parsing Network[J]. 2016.
