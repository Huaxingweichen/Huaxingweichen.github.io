---
layout: post
title: "Rethinking Atrous Convolution for Semantic Image Segmentation 笔记(DeepLabv3）"
date: 2019-10-20 11:06:06 
description: ""
tag: Semantic Segmentation
---

`本来看了几篇semantic segmentation的文章，以为自己的英语水平读起来并不费劲了，没想到这篇文章用了一些不同的方法，阅读起来还是比较吃力的，英语还是有的菜啊。总体来说这篇文章还是在讨论空洞卷积的在语义分割领域的应用，文章中提到的一些方法以及设计对我来说还是比较新颖的，毕竟刚接触这个领域。`

### Abstract

> In this work, we revisit atrous convolution, a powerful tool
to explicitly adjust filter’s field-of-view as well as control the resolution of feature responses computed by Deep Convolutional Neural Networks, in the application of semantic image segmentation. 

开宗明义，本文工作回顾atrous convolution，atrous convolution可以调整感受野，同时控制feature response的分辨率.

> To handle the problem of segmenting objects at multiple scales, we design modules which employ atrous convolution in cascade or in parallel to capture multi-scale context by adopting multiple atrous rates.

为了应对物体多尺度的问题，我们设计了空洞卷积module(串行 or 并行)通过采用不同的空洞卷积的atrous rates(卷积核之间的空有多大)来获取多尺度的语义信息。

> Furthermore, we propose to augment our previously proposed Atrous Spatial Pyramid Pooling module, which probes convolutional features at multiple scales, with image-level features encoding global context and further boost performance.

进一步，我们提高了我们之前提出的网络在图片层级编码全局语义信息的能力同时提高了表现。

> We also elaborate on implementation details and share our experience on training our system.

我们详细叙述了实现细节以及分享了我们的训练经验。

### Introduction

开始，作者写到了语义分割的俩大挑战。

> The first one is the reduced feature resolution caused by consecutive pooling operations or convolution striding, which allows DCNNs to learn increasingly abstract feature representations. However, this invariance to local image transformation may impede dense prediction tasks, where detailed spatial information is deired.

卷积和池化操作会减少feature的分辨率，会损害精密空间信息。（atrous convolution可以解决这个问题）

> Another difficulty comes from the existence of objects
at multiple scales.

另一个问题是物体有各种各有的尺度。（作者列出现有的四种解决这个问题的网络结构/方法，在related work中会分别阐述每一个方法）

![deeplabv3_solution_to_multiple_scale](/images/posts/deeplabv3_solution_to_multiple_scale.png)

### Related Work

- Image pyramid

> The same model, typically with shared weights, is applied to multi-scale inputs. Feature responses from the small scale inputs encode the long-range context, while the large scale inputs preserve the small object details.

大尺度小特征，小尺度大特征（long-range context）

> The main drawback of this type of models is that it does not scale well for larger/deeper DCNNs (e.g., networks like) due to limited GPU memory and thus it is usually applied during the inference stage.

受限显存，大多停留在理论阶段。

- Encoder-decoder

> the encoder where the spatial dimension of feature maps is gradually reduced and thus longer range information is more easily captured in the deeper encoder output

encoder的空间维度逐渐减小，能够提取到longer range 的特征。

>the decoder where object details and spatial dimension are gradually recovered.

decoder恢复健康物体细节信息以及空间维度。

- Context module

这个讲道理不是很清楚，但是个人理解来看CCNet, OCNet等论文中等module应该就是context module.

- Spatial pyramid pooling

空间金字塔池化了，感兴趣的可以看我的[PSPNet的阅读笔记](http://www.huanghao.site/2019/10/Pyramid-Scene-Parsing-Network%E7%AC%94%E8%AE%B0/)

- Atrous convolution

简述空洞卷积等应用，感兴趣可以阅读一下原文。

### Methods

- Atrous Convolution for Dense Feature Extraction

这一节主要说原来语义分割用卷积再转置卷积的模式，但是作者推荐用atrous convolution,然后列出了atrous convolution的公式，但是我觉得公式不是很好明白，建议知乎搜索一下空洞卷积，看一看动图就很容易明白了。

- Going Deeper with Atrous Convolution

持续的卷积池化会破坏特征，所以采用空洞卷积,既有池化的效果，又不会破坏detail feature。

![deeplabv3_atrous_module](/images/posts/deeplabv3_atrous_module.png)

- Atrous Spatial Pyramid Pooling

这个地方一开始没有理解，后来在GitHub上找到了deeplabv3的代码，有了自己的一点看法。
atrous rate过大会导致filters提取不到有价值的features，所以对于long-range context来说，需要一个方法去提取它，于是就有了一个全局池化再上采样的操作，这个地方我个人感觉达到了效果，但是显得有点僵硬，为了助于理解，截取一段代码。[Github 完整代码](https://github.com/fregu856/deeplabv3/blob/master/model/aspp.py)

``` python
def forward(self, feature_map):
        # (feature_map has shape (batch_size, 512, h/16, w/16)) (assuming self.resnet is ResNet18_OS16 or ResNet34_OS16. If self.resnet instead is ResNet18_OS8 or ResNet34_OS8, it will be (batch_size, 512, h/8, w/8))

        feature_map_h = feature_map.size()[2] # (== h/16)
        feature_map_w = feature_map.size()[3] # (== w/16)

        out_1x1 = F.relu(self.bn_conv_1x1_1(self.conv_1x1_1(feature_map))) # (shape: (batch_size, 256, h/16, w/16))
        out_3x3_1 = F.relu(self.bn_conv_3x3_1(self.conv_3x3_1(feature_map))) # (shape: (batch_size, 256, h/16, w/16))
        out_3x3_2 = F.relu(self.bn_conv_3x3_2(self.conv_3x3_2(feature_map))) # (shape: (batch_size, 256, h/16, w/16))
        out_3x3_3 = F.relu(self.bn_conv_3x3_3(self.conv_3x3_3(feature_map))) # (shape: (batch_size, 256, h/16, w/16))

        out_img = self.avg_pool(feature_map) # (shape: (batch_size, 512, 1, 1))
        out_img = F.relu(self.bn_conv_1x1_2(self.conv_1x1_2(out_img))) # (shape: (batch_size, 256, 1, 1))
        out_img = F.upsample(out_img, size=(feature_map_h, feature_map_w), mode="bilinear") # (shape: (batch_size, 256, h/16, w/16))

        out = torch.cat([out_1x1, out_3x3_1, out_3x3_2, out_3x3_3, out_img], 1) # (shape: (batch_size, 1280, h/16, w/16))
        out = F.relu(self.bn_conv_1x1_3(self.conv_1x1_3(out))) # (shape: (batch_size, 256, h/16, w/16))
        out = self.conv_1x1_4(out) # (shape: (batch_size, num_classes, h/16, w/16))

        return out
```

### Experimental Evaluation

目前重点关注思路，实验部分以及tricks暂时略过。

### Conclusion

> Our proposed model “DeepLabv3” employs atrous convolution with upsampled filters to extract dense feature maps and to capture long range context. Specifically, to encode multi-scale information, our proposed cascaded module gradually doubles the atrous rates while our proposed atrous spatial pyramid pooling module augmented with image-level features probes the features with filters at multiple sampling rates and effective field-of-views. Our experimental results show that the proposed model significantly improves over previous DeepLab versions and achieves comparable performance with other state-of-art models on the PASCAL VOC 2012 semantic image segmentation benchmark.
