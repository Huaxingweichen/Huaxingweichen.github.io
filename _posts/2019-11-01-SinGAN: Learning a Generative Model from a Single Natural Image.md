---
layout: post
title: "ICCV2019 bestpaper SinGAN: Learning a Generative Model from a Single Natural Image 笔记"
date: 2019-11-01 14:51:06 
description: ""
tag: GAN
---

`ICCV2019的best paper,非常好的一篇文章，同时因为对于该文章详细解读的中文的资料不是很多，因此本篇笔记也采用中文撰写，希望能够对大家的阅读有所帮助。（本处有个人观点，希望仅作为参考，仁者见仁，智者见智）我个人看到这篇文章非常兴奋，有很多新的idea，但是真正跑了office的代码之后反而冷静了下来，代码的效果并没有paper的图片那么惊艳，对于没有一个benchmark的生成任务来说，作者已经尽力在定性和定量俩个维度去证明自己的成果，但是最终好不好，网络是不是有较大局限性我目前持保留意见，但是感觉singan挖了一个小坑，同志们可以去填坑了，但是最终这个坑到底有多深，我的水平还是不够，不好说。以下有个人观点之处，我会提前声明。`

![SinGAN-sample](/images/posts/SinGAN-sample.png)

（上面的图片真好看啊 :) ）

### Abstract

> We introduce SinGAN, an unconditional generative model that can be learned from a single natural image. Our model is trained to capture the internal distribution of patches within the image, and is then able to generate high quality, diverse samples that carry the same visual content as the image.

首先，作者说明了自己在本文做的事情，SinGAN是一种无条件的生成模型，可以从一个单张图片获取patch的内在分布，然后生成出来高质量的不同的相同视觉内容的图片。

> SinGAN contains a pyramid of fully convolutional GANs, each responsible for learning the patch distribution at a different scale ofthe image. This allows generating new samples of arbitrary size and aspect ratio, that have significant variability, yet maintain both the global structure and the fine textures of the training image.

SinGAN有一个金字塔全卷积`GANs`,每个GAN在不同尺度上学习一点patch的分布，这可以生成任意尺寸和任意角度的任意纵横比的图片（因为全卷积网络，不懂的可以去搜全卷积神经网络)同时维持全局结构和纹理特征（多尺度的好处，也就是金字塔形结构）。

### Introduction

unconditional GANs 在生成图片方面取得了很大的成功，但是限制于数据集。对于高度变化的不同类别的物体生成一直是一个主要的挑战（本文也正是做了这方面的努力）。

> Here, we take the use of GANs into a new realm – un-conditional generation learned from a single natural image.

作者说到自己将GAN带入了一个新的领域，从单张图片学习并生成类似语义的图片。作者表示SinGAN可以让我们处理普通的包含复杂结构和纹理的图片，不依赖于一类数据集，进行生成任务。为其带来强大功能的原因是金字塔形全卷积轻量级GANs的应用。SinGAN 可以被用作多重图像编辑的任务，包括根据画图生成真实图像，编辑图像，图像融合，超分，电影制作。

### Related Work

最近一些网络采用“过拟合”的一些深度模型去针对于单张图片，但是这些方法被用于特定的任务或者不能被用来生成随机样本。Unconditional single image GANs只被用于提前纹理信息，这些方法不能生成一个没有纹理信息的图片，但是SinGAN可以处理这样的图片。如图所示：

![SinGAN_compare](/images/posts/SinGAN_compare.png)

GAN生成图片往往基于一定的数据集，但是作者表示不感兴趣在特定的数据集上提取一类特征，作者更喜欢去在一个图片上去提取不同尺度的patches分布。

### Method

`重点，在这里我通过图片详细叙述网络结构`

![SinGAN_structure](/images/posts/SinGAN_structure.png)

首先，$Z_n$是一个噪音，进入第一个generator，产生$\tilde{X_N}$,然后该产生的$\tilde{X_N}$向上传递，同时于上一轮的噪音进行融合（之前需要上采样），下图是具体细节：

![SinGAN_structure_details](/images/posts/SinGAN_structure_details.png)

在第n轮，$\tilde{X_{N+1}}$经过上采样之后，于$Z_n$相加,之后过一遍卷积层，并且与$\tilde{X_{N+1}}$经过上采样之后的结果再次相加，一直到$G_0$为止。那为什么要进行这个操作呢？
每一个尺度的$G_N$都可以生成上一个尺度没有的信息（得益于噪声的加入），同时后一个将$\tilde{X_{N+1}}$放回去，感觉有点是确保训练的稳定性。（个人观点）

训练过程中有俩个损失函数：

- Adversarial loss

> The adversarial loss Ladv penalizes for the distance between the distribution of patches in xn and the distribution of patches in generated samples ˜xn.

这个是正常的gan训练的损失

- Reconstruction loss

![SinGAN_Rec_loss](/images/posts/SinGAN_Rec_loss.png)

(我的个人理解是：即使没有后面噪声的加入，第一个噪声可以生成稳定的原图，这个损失是为了确保这个，而有了后面的噪声，一些变化就可以多姿多彩了)

### Result

略

### Applications

略

`文章还有一些细节，我大体的写出来了我觉得更重要的地方，如果有地方leave out了，欢迎私信我，我进行补充。`
