---
layout:     post
title:      "namespace Terminating 的陷阱"
subtitle:   "namespace Terminating"
date:       2024-05-16
author:     "moradudu"
header-img: "img/post-bg-web.jpg"
tags:
  - 运维
  - kubernetes
---

## namespace Terminating 的陷阱

网上绝大部分关于namespace Terminating的解决方式来自于[github](https://github.com/kubernetes/kubernetes/issues/60807)

但是这个方法有陷阱，通过删除finalize，确实可以删除namespace。但是这个也破坏了kubernetes的级联删除。导致出现如下结果

![image-20200921111324012](/img/blog/image-20200921111324012.png)


## 资源回收

kubernetes 的资源有三种删除方式，也叫kubernetes资源回收

* 直接删除。运行kubelet delete时，立刻删除资源，不管资源的下属资源

* 前台级联删除。运行kubelet delete时，先删除下属资源，最后删除本资源

* 后台级联删除。运行kubelet delete时，先删除本资源，然后删除下属资源

## 前台级联删除

当运行kubelet delete时，步骤如下

1. kubernetes会给本资源打上两个标签：finalize、deletetimestamp。
2. 删除下属资源，如果下属资源删除完成，就删除finalize标签
3. 因为finalize标签没有，deletetimestamp标签有。所以删除本资源

**当资源上有finalize标签时，就不能删除资源。**

所以当你通过删除finalize来删除namespace就会出现问题。你的namespace下属资源没有删除干净。这将会导致两个问题：1. 无效的资源不断的集聚。2. 更深层次的原因不及时发现。几乎所有的Terminating都不是资源本身的问题。