---
layout: post
title: "ICCV2019 TextDragon: An End-to-End Framework for Arbitrary Shaped Text Spotting 笔记"
date: 2019-11-01 14:51:06 
description: ""
tag: Scene Text Detection
---

`这篇文章看起来也有点费劲，感觉自己开的领域有一点点多，导致每个领域入门的时候都很头疼，希望头疼的经历可以对以后的学习有一定对帮助～ps:英语不好真难受`

### Abstract

> Most existing text spotting methods either focus on horizontal/oriented texts or perform arbitrary shaped text spotting with character-level annotations.In this paper, we propose a novel text spotting framework to detect and recognize text of arbitrary shapes in an end-to-end manner, using only word/line-level annotations for training.

Author show status quo about existing text spotting methods and proposed a novel text spotting framework —— an end-to-end manner to detect and recognize text of arbitrary shapes. RoISlide is a differentiable operator which is key to connect arbitrary shaped text detection and recognition.

### Introduction

Most recently works divide the detection and recognition as two parts suffering heavy time cost and correlation ignorance between text detection and recognition. However, some methods unifying detection and recognition can't fit the real-world scene and TextSnake(a method proposed by pku) only focus on the detection.

Author concentrate on the reading mechanism of human.

>First, a local area of the text is detected. After that, the contents of the local area are recognized. Finally, the eyes move along the centerline of the text and repeat the above three steps.

Based on this mechanism, author proposed TextSnake and make three contributions as follow:

> - A novel end-to-end scene text spotter named TextDragon is proposed, which is flexible for text of arbitrary shapes.
> - A new differentiable operator RoISlide is designed to unify arbitrary shaped text detection and recognition into an end-to-end pipeline.
> - The proposed model can be trained in weakly supervised manner with only word/line-level annotations, which improves the model’s practicability.

### Related Work

- Scene Text Detection
  
  - Existing methods can't handle arbitrary shaped text or can't do recognition at a time.

- Scene Text Recognition
  
  - 
