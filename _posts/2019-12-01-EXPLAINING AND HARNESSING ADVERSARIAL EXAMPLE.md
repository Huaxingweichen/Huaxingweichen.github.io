---
layout: post
title: "ICLR2015 Explaining and Harnessing Adversarial Examples"
date: 2019-12-01 10:56:06 
description: ""
tag: Adversarial Examples
---

`对抗样本自从2014年被提出来之后，逐渐引发人们的关注,在Ian J. Goodfellow大佬的推动下，对抗样本攻防逐渐成为一个小领域，本文是Ian J. Goodfellow大佬在2015年ICLR上发表的论文，解释了对抗样本的机理以及通过实验论证了他的假设，同时借助该假设提出了一个快速梯度标记算法来产生对抗样本.`

推荐俩篇论文笔记，对我有所帮助，也可以结合着看看：

- [论文笔记Explaining & Harnessing Adversarial Examples](https://zhuanlan.zhihu.com/p/32784766)

- [《Explaining and Harnessing Adversarial Examples》阅读笔记](https://zhuanlan.zhihu.com/p/33875223)

### Abstract

一些机器学习方法，包括神经网络都会被对抗样本（输入含有小的但是坏的扰动）误导。这种对抗样本的输入会让神经网络得出一个置信度高并且错误的答案。早期的工作尝试用非线性特征和过拟合去解释它，我们缺任务神经网络对于对抗样本的弱点主要体现在它的线性特质。`新的定量实验证明了这个观点，并且证明了一个神奇的事实，对抗样本是跨结构和训练集的。`（这句话不是很确定是否理解正确。）基于这个假设，我们提出了快速梯度标记算法，通过产生对抗样本，降低了测试集的错误率（MNIST maxout network）.

### Introduction

Szegedy et al. 2014年发现了对抗样本的现象，同时发现对于不同的结构的模型都会被对抗样本影响，这表明了对抗样本揭示了我们算法的基本盲点。产生对抗样本的原因是神秘的，猜测可能是因为神经网络的非线性特性，也许也因为纯监督学习模型不够平均和正则化（insufficient model averaging and insufficient regularization）。作者证明这种假设不是必然的，高维空间的线性行为足以造成对抗样本，同时，对抗样本可以提供额外的正则化好处（超过了dropout）,作者最后提出了一个希望，长远来看，也许会有更有力的优化方法去训练更多的非线性模型来解决这个问题。

---
我的思考：所谓对抗样本实际上是模型并没有学习到本质的数据特质而产生，至于为什么没有学习到本质特征可能有多种解释，但是最直观的一种表示如下图所示：

![Adversarial_Examples_active_map](/images/posts/Adversarial_Examples_active_map.png)
![Adversarial_Examples_active_map](/images/posts/Adversarial_Examples_active_map1.png)

`图摘自西交利物浦大学 Kaizhu Huang 老师组 tutorial`

没有正则化的激活图的分布很混乱，容易浑水摸鱼，但是正则化的就很平滑，这幅图一方面反应了对抗样本本质上是为什么作用于分类模型的，另一方面也阐述了对抗样本是如何让提高训练的准确率和鲁棒性的。

### Related Work

这边列举了一些相关工作，但是因为当时这个领域还比较新，所以没有太多引用，很多东西作者放了一些结论性质的陈述，很多叙述有点重复了，于是leave out一下。

### The Linear Explanation of Adversarial Examples

这一部分作者开始解释对抗样本存在的原因（对于线性模型）。

之后作者叙说了样本输入特质有一定精度，例如一个像素是255，那么低于1/255的精度的扰动将会被忽略（但是我没看出来这一段有什么用......）

这一段作者想要表达的内容是对于一个线性模型来说，只要有足够深的维度，就可以有对抗样本，并且一个小的扰动，会随着维度的加深造成逐渐变大的影响。作者说线性解释相对于非线性解释更加简单，也可以解决softmax回归的对于对抗样本比较脆弱的问题。

### Linear Perturbation of Non-Linear Models

作者先假设神经网络具有过多的线性性质以至于不能抵抗非线性的对抗扰动，LSTM族，ReLU族，maxout networks是基于线性性质的，sigmoid等方法是基于非线性的。而这些线性行为所带来的对抗样本的脆弱性也将摧毁神经网络。

w⊤x ̃ = w⊤x + w⊤η
η = εsign (∇xJ(θ, x, y))

之前对这个方法一直表示疑惑，机器学习对底子太薄弱了，看到下面对博客之后豁然开朗。

> 常规的分类模型训练在更新参数时都是将参数减去计算得到的梯度，这样就能使得损失值越来越小，从而模型预测对的概率越来越大。既然无目标攻击是希望模型将输入图像错分类成正确类别以外的其他任何一个类别都算攻击成功，那么只需要损失值越来越大就可以达到这个目标，也就是模型预测的概率中对应于真实标签的概率越小越好，这和原来的参数更新目的正好相反。因此我只需要在输入图像中加上计算得到的梯度方向，这样修改后的图像经过分类网络时的损失值就比修改前的图像经过分类网络时的损失值要大，换句话说，模型预测对的概率变小了。这就是FGSM算法的内容，一方面是基于输入图像计算梯度，另一方面更新输入图像时是加上梯度，而不是减去梯度，这和常见的分类模型更新参数正好背道而驰。
前面我们提到之所以采用梯度方向而不是采用梯度值是为了控制扰动的L∞距离，这只是其中的一部分。在Figure1中，梯度方向前一般会有一个权重参数e，这个权重参数可以用来控制攻击噪声的幅值，参数值越大，攻击强度也越大，肉眼也更容易观察到攻击噪声（因为输入图像归一化成0到1，所以图中e值只有0.07，换算成0到255的话差不多18个像素值），因此最终的攻击噪声就如下所示。因为FGSM算法的攻击噪声幅值评价指标是L∞，因此权重参数e和梯度方向就可以控制每个像素的最大变化值。
————————————————
版权声明：本文为CSDN博主「AI之路」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014380165/article/details/90723948

作者说这个算法也证明了他假设的正确性同时也有助于对抗训练和分析训练网络。

### Adversarial Training of Linear Models Versus Weight Decay

这里举出了一个例子，例子略。

### Adversarial Training of Deep Networks

这段作者首先说明老是说深度学习怕对抗样本是不明智的，至少相对于浅的线性网络来说，深度学习有能力对抗扰动。又提及了`The universal approximator theorem`这个理论，说明神经网络只要有足够的神经元就可以拟合任何函数，但是这个理论并没有说明神经网络学习的特质是不是我们想要的（本质的特质），同时提及，对抗样本也是一种不同的数据增强。

之后作者在maxout network做了一个实验，用了这个之后损失从0.94%降低到了0.84%。（这个点太少了，说服力有待商榷）。

之后作者注意到在训练数据上没有做到0误差，于是做了俩点改进：

- 将神经元数目从240个变成了1600个。
- 同时使用early stopping在对抗样本到测试集上。

用了之后跑了五次，结果都非常好，这里有一个小trick，作者想让神经网络变得过拟合了一点点，再这样做涨了点，但是作者同时使用了两个tricks，所以就不是很好说到底是哪一个work还是两个都work，也没有详细的对比实验来论证这个地方。

之后作者做了一个实验，细想很有意思。

用原来的模型生成的对抗样本作用在原来的模型上，错误率是89.4%，但是作用在使用对抗样本训练的模型上，错误率是17.9%。用对抗样本训练的模型去生成对抗样本，作用在原来的模型上，错误率是40.9%，作用在用对抗样本训练的模型上，错误率是19.6。
（我当时感觉不对啊，为什么用对抗样本训练的模型生成的对抗样本作用在反而效果差了呢，不是应该更牛逼嘛，后来想了想，发现对抗样本本来是就是让模型更加鲁棒，用这个模型再强行对抗样本，肯定有些样本就不像原始数据了，这样想一想也对。）

但是对抗样本训练的模型对于还是预测不对的对抗样本有很高的置信度（这里可能是因为模型能力确实不够吧，干掉了小的错误，大的错误还是不好改变）。

![Adversarial_training](/images/posts/Adversarial_training.png)
这个就是直观的用对抗样本训练的图，还是比较明显能看出来，用对抗样本训练的更干净了。

> Our view of adversarial training is that it is only clearly useful when the model has the capacity to learn to resist adversarial examples. This is only clearly the case when a universal approximator theorem applies. Because the last layer of a neural network, the linear-sigmoid or linear-softmax layer, is not a universal approximator of functions of the final hidden layer, this suggests that one is likely to encounter problems with underfitting when applying adversarial perturbations to the final hidden layer. We indeed found this effect. Our best results with training using perturbations of hidden layers never involved perturbations of the final hidden layer.

这个地方我看到很多笔记都没有注意，具体实现的时候还是要注意一下。

......要去健身了，未完待续，这篇文章真的好硬核啊！

### Different Kinds of Model Capacity

未完待续......