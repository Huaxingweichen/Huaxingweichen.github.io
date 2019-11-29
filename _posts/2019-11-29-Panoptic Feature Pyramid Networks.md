---
layout: post
title: "CVPR2019 Panoptic Feature Pyramid Networks"
date: 2019-11-29 15:24:06 
description: ""
tag: Panoptic Segmatation
---

`Panoptic as a new task combine semantic segmentation and instance segmentation renewed some people's interest. Facebook AI Research(FAIR) proposed Panoptic FPN to solve the balance between semantic segmentation and instance segmantation.`

![PFPN_sample](/images/posts/PFPN_sample.jpg)

### Abstract

> current state-of-the-art methods for this joint task use separate and dissimilar networks for instance and semantic segmentation, without performing any shared computation.

Semantic segmentation is to generates dense-piexl outputs while instance segmantation is to generates region-based outputs. So, how to banlance the loss of the two branches and share computation is a main problem to solve. At the same time, we alse need to let the panoptic net has the state-of-the-art performance. FAIR proposed Panopic FPN which uses FPN as backbone than add two branchs to finish semantic and instance segmentation.

### Introduction

> For semantic segmentation, FCNs with specialized backbones enhanced by dilated convolutions dominate popular leaderboards.

> For instance segmentation, the region-based Mask R-CNN with a Feature Pyramid Network (FPN) backbone has been used as a foundation for all top entries in recent recognition challenges.

That's reason why current works use separate networks.

> Our approach starts with the FPN backbone popular for instance-level recognition and adds a branch for performing semantic segmentation in parallel with the existing region-based branch for instance segmentation.

![PFPN_structure](/images/posts/PFPN_structure.png)

(In my opinion, Panoptic FPN may be not successful in it's structure or maybe not the main point. Author also try many tricks which I think is the key for good results and author also said this.)

> Overall, while our approach is robust to exact design choices, properly addressing these issues is key for good results.

If trained for each task indenpendently, the two results are all excellent. However, How to balance them is a key issue.

### Related Work

leave out

### Panoptic Feature Pyramid Network

After discuss FPN and instance segmentation, author list three requirements for features:

- be of suitably high resolution to capture fine structures
- encode sufficiently rich semantics to accurately predict class labels
- capture multi-scale information to predict stuff regions at multiple resolutions

Author do lots of experiments to employ the which scale play an important role in this structure. 

`Joint training` To balance the losses

![PFPN_loss](/images/posts/PFPN_loss.png)

authoe use two scale and do some contrast experiments to employ it.

### Experiments

Actually, you shoule read this section carefully and a lot of interesting tricks and comparations are showed here.

### Conclusion

>We introduce a conceptually simple yet effective baseline for panoptic segmentation. The method starts with Mask R-CNN with FPN and adds to it a lightweight semantic segmentation branch for dense-pixel prediction. We hope it can serve as a strong foundation for future research.