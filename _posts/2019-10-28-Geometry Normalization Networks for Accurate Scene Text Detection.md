---
layout: post
title: "ICCV2019 Geometry Normalization Networks for Accurate Scene Text Detection 笔记"
date: 2019-10-27 15:53:06 
description: ""
tag: Scene Text Detection
---

`iccv 2019的这篇关于场景文字检测的文章，从一个新颖的角度解决了基于CNN的网络只能处理有限集合变化的情况，并且做了较为充分的对比实验已支持自己的观点。本篇文章开始基本采用英语进行撰写笔记，fighting！`

### Abstract

> Large geometry (e.g., orientation) variances are the key challenges in the scene text detection.

Author come straight to the point，propose the key problem the paper focus on.

> In this work, we first conduct experiments to investigate the capacity ofnetworks for learning geometry variances on detecting scene texts, and find that networks can handle only limited text geometry variances. Then, we put forward a novel Geometry Normalization Module (GNM) with multiple branches, each of which is composed of one Scale Normalization Unit and one Orientation Normalization Unit, to normalize each text instance to one desired canonical geometry range through at least one branch.

Author do some experments and find CNN based Network can handle only limited text geometry variances. To solve this problem, Gemetry Normalization Module is proposed which has `one Scale Normalization Unit` and `one Orientation Normalization Unit`. It can normalize each text instance to one desired canonical geometry range.

> The GNM is general and readily plugged into existing convolutional neural network based text detectors to construct end-to-end Geometry Normalization Networks (GNNets).

GNM 有很好的适用性。

> Moreover, we propose a geometry-aware training scheme to effectively train the GNNets by sampling and augmenting text instances from a uniform geometry variance distribution.

提出了一些geometry-awarede的training scheme.

### Introduction

Despite the great success of CNNs, scene text detection hac its onw problem as texts have large geometry variances. There are some papers try to solve the problem and put forward to some methods. Every detection header learns all training samples with enormous geometry or only subset of them which lead to limitation of performance.
At the same time, text angles in some benchmarks mostly close to 0. So, Author make a rotated ICDAR 2015 by randomly rotating images, so that the orientations are distributed uniformly for better evalution.

>To solve the above dilemma, we propose a novel Geometry Normalization Module (GNM). It has multiple normalization branches, each of which is composed of one Scale Normalization Unit (SNU) and one Orientation Normalization Unit (ONU), and can learn geometry-specific feature maps.

- Geometry Normalization Moudlu (GMU) --> learn geometry-specific feature maps
  - Scale Normalization Unit (SMU)
  - Orientation Normalization Unit (ONU)

By GMU， Author thought that

> large geometry variances of all training samples are normalized to a limited distribution, so that we can train one shared text detection header on them effectively。

`To be honest, when I read here I can't understand why GMU can normalize the training samples. After I read this paper, I also don't realized why this structure can do it. Maybe The reason is cai. `

> Training GNNets is non-trivial. We further propose a geometry-aware training strategy to effectively train GN-Nets by randomly augmenting text instances so that they are sampled from a uniform geometry variance distribution.

Author also propose a training strategy named geometry-aware training strategy which can effectively train network.

### Related Work Scene

Author list two sections for related work and pay more attention to the scene text detection. I leave out some history work and if you want to learn more you can read the paper from iccv2019 open access. I focus on the idea proposed by this papers.

- Scene text detection
  Inspired by SSD [1], some people detected texts independently on multi-layews. However, low level features lack high level semantics which easily leads to missing detection or false alarms. Later work introduce FPN-like[2] or FCN-like[3] sturcture to compensate the absence of semantics in low-level features but features vary drasically which hinders to learn the mapping from features to bounding boxes well. Others also can't solve this problem well.

  Then, Author talk about their mothod:
  > our proposed GNNets achieve both scale and orientation robustness of text detection in a unified framework.

- Scale normalization for object detection

  `I'm not familiar with thr background about this section.`
   Author list two main mothods which have their own problem and increase the inference time and the mothod proposed by author don't has those problems.

### Motivations

In this Section, author said his team find state-of-the-art mothods cann't solve the problem well and they find these is still great room of improvements.

`I think observation is significant. If you can observe some points that others have not done and it actually can be achieve, you can be the regular of iccv... .`

Authors try to evaluate the impacts of the geometry variances and use three control groups.

- Geometry Specific Sampling (GSS)
  Author select one text per image in training set and transform the image so that the chosen text instance will lie in the desired geometry range.

- Geometry Variance Sampling (GVS)
  Like GSS but has large geometry range.

- Limited Geometry Specific Sampling (LGSS)
  Subset of GSS.

![GNN_evalution_result](/images/posts/GNN_evalution_result.png)

In sum, GSS has the greatest performance.

> The above observations motivate us to normalize geometry variances during training, so that a geometry specific detector only needs to handle limited geometry range.

### Geometry Normalization Networks

`Important Section`

- Geometry Normalization Module
  - Scale Normalization Unit
    S --> 1x1 conv
    S_1/2 --> a stack of 1×1 conv, 2×2 max-pooling with stride 2, and 3×3 conv operators.

  - Orientation Normalization Unit
    O --> 1×1 conv
    O_f --> a stack of 1×1 conv, flipping, and 3×3 conv operators
    O_r --> a stack of 1×1 conv, rotation, and 3×3 conv operators
    O_r+f --> a sequence of 1 × 1 conv, rotation, flipping and 3×3 conv

    ![GNM_ONU](/images/posts/GNM_ONU.png)

- Architecture of GNNets
  Author don't display the overall structure but calculate the result size/angle after GNM. If you want to look the figure, please go to the paper.

- Geometry-Aware Training and Test Strategy
  - Feasible geometry range
    In brief, we only select desired data to train.
  - Training sampling strategy
    Every batch, retating and resizing a text instance 7 times to ensure every branches are trained uniformly.
  - Test strategy
    Abandon the predict results which do not lie in branch's corresponding feasible geometry range and merge others via NMS.

### Experiments

I select some points from this section and show them in above so I leave out this section.

### Conclusion

The biggest contribution is GNM and the strategy to train. (My view!)

### My Reference

[1] Wei Liu, Dragomir Anguelov, Dumitru Erhan, Christian Szegedy, Scott Reed, Cheng Yang Fu, and Alexander C. Berg. SSD: Single Shot Multibox Detector. In ECCV, 2016.
[2] Tsung-Yi Lin, Piotr Doll´ar, Ross Girshick, Kaiming He, Bharath Hariharan, and Serge Belongie. Feature Pyramid Networks for Object Detection. In CVPR, 2017.
[3] Jonathan Long, Evan Shelhamer, and Trevor Darrell. Fully Convolutional Networks for Semantic Segmentation. In CVPR, 2015.
