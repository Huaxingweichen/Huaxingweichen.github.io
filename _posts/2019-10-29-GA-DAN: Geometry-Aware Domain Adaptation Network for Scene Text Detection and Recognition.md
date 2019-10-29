---
layout: post
title: "ICCV2019 GA-DAN: Geometry-Aware Domain Adaptation Network for Scene Text Detection and Recognition 笔记"
date: 2019-10-29 11:01:06 
description: ""
tag: Domain Adaptation / Cycle GAN
---

`本来之前按关键字搜索了数篇iccv19关于场景文字识别的paper，但是这一篇重点是在说 Cycle GAN, 而不是重点关注在场景文字识别本身，其思路等于是把cycle gan的一套拿过来，应用在场景文字识别的这个具体应用上，并且对应用场景做出一定的优化。`

### Abstract

> Recent adversarial learning research has achieved very impressive progress for modelling cross-domain data shifts in appearance space but its counterpart in modelling cross-domain shifts in geometry space lags far behind.

Author thought that modeling cross-domain data shifts in appearance space do better than in geometry space.

> This paper presents an innovative Geometry-Aware Domain Adaptation Network (GA-DAN) that is capable of modelling cross-domain shifts concurrently in both geometry space and appearance space and realistically converting images across domains with very different characteristics.

Author show what they do in this paper is that presents an innovative Geometry-Aware Domain Adaptation Network (GA-DAN) to solve the problem above.
Author list the contributions of this paper in `Introduction`, and I introduce them in `Abstract`.

>- First, it designs a novel network that models domain shifts in geometry and appearance spaces simultaneously.
>- Second, it designs an innovative multi-modal spatial learning mechanism and introduces a spatial transformation discriminator to achieve multi-modal adaptation in geometry space.
>- Third, it designs a disentangled cycle-consistency loss that balances the cycle-consistency for concurrent adaptation in appearance and geometry spaces and can also be applied to generic domain adaptation.

In my view, `network structure, spatial learning mechanism, loss` are the key points in this paper.

### Introduction

On the one hand collecting labelled data is extremely expensive, on the other hand DNNs will have clear performance drops when they meet different style data.
So, unsurprivised Domain Adaptation (DA) is introduced to transfers images and features from a source domain to a target domain and handle some cv problems successfully.

> State-of-the-art DA still faces various problems. In particular, most existing systems focus on learning feature shift in appearance space only whereas the feature shift in geometry space is largely neglected.

In this section, author stress that state-of-the-art DA mostly focus on appearance space and neglect geometry space. However, in my opinion, cycle GAN is designed to do some style transfer for images. So, it's natural that advanced DA focus on appearance space. :)

> On the other hand, images from different domains often differ in both appearance and geometry spaces.

Some cv tasks need to transer source domain to target demain both in appearance spaces and geometry spaces. So authors design GA-DAN to handle this problem and make some contributions.

### Related Works

`We only concentrate on the main idea and ignore the details. If you want to learn more about related work, please read the paper.`

- Domain Adaptation

  > Existing techniques can be broadly classified into two categories.

  - The first category focuses on minimizing discrepancies between the source domain and the target domain in the feature space.

  - The second category adopts Generative Adversarial Nets(GANs) to perform pixel-level adaptation via continuous adversarial learning between generators and discriminators which has achieved great success in image generation, image composition and image-to-image translation.

- Scene Text Detection and Recognition
  A main limitation for scene text detection is the dataset is not big enough to train accurate and robust scene text detectors. So, authors want to solve the problem via DA.

### Methodology

![GA-DAN_structure](/images/posts/GA-DAN_structure.png)

- GA-DAN Architecture
  Due to the complex descripetion of network architecture, I will not list detials in note. The significant thing to realize this sturture is to konw how infomation flow.

  Firstly, input the image `x` to Sx. The LNx is a localization network and the T is a module for transformation. Sx will produce a `Hxy` a matrix to transfer `X` to `Transformed X`. Then Gxa can complete the image background and Gxb futher adapts the completed images to have similar appearance as Y.

- Multi-Modal Spatial Learning

  - Spatial Code can be sampled to produce different T then Sx use it to generate different images.

  - Dx and Dx loss can not make this network train stable，author add a loss DT to calculate the loss of Hxy - Hyx.

- Adversarial Training
  This section mainly about loss and I firstly show the $L_{cyc}$

  ![GA-DAN_ACL_SCL](/images/posts/GA-DAN_ACL_SCL.png)
  ![GA-DAN_cyc_loss](/images/posts/GA-DAN_cyc_loss.png)
  
  - SCL spatial cycle-consistency loss
  - ACL appearance cycle-consistency loss
  - EML region missing loss (to solve bordering regions be lost in training)

  `I leave out the GAN loss`

  In fact, author design many tricks in training and some of them are very smart. However, author don't opensource their code and reimplementing it is so difficult for me.

### Experiments

  Because the experiment is add the module to other network and show the better performance so I think is not need to show it's detail. And if you want to look the result, please look the paper.

### Conclusions

> This paper presents a geometry-aware domain adaptation network that achieves domain adaptation in geometry and appearance spaces simultaneously. A multi-modal spatial learning technique is proposed which can generate multiple adapted images with different spatial views. A novel disentangle cycle-consistency loss is designed which greatly improves the stability and concurrent learning in both geometry and appearance spaces. The proposed network has been validated over scene text detection and recognition tasks and experiments show the superiority of the adapted images while applied to train deep networks.

`这篇文章看了好久，确实不太好理解cycly gan，如果没有之前接触过的话，但是文章还是挺有趣的，特别是在训练上做了很多有趣并且有意义的尝试。`
