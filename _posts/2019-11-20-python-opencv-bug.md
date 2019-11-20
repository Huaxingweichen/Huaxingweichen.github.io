---
layout: post
title: "python-opencv 使用之坑"
date: 2019-11-01 14:51:06 
description: ""
tag: bug
---

### cv2.imwrite()

cv2.imwrite()的时候，如果保存成jpg格式，会导致图片像素扩散，例如，一张图片的灰度值只有30，35，如果用该方法保存成jpg的话，可能30-35区间中的灰度值都有，这太难顶了！
保存成png啥事没有，这个坑对于语义分割来说有点难受，实验全部重来了。
