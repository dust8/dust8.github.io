---
title: 使用 tensorflowjs 的 handpose 模型做体感游戏
date: 2020-03-17 05:15:31
tags:
  - tensorflow
  - handpose
  - 手势
  - threejs
---

前不久看到 `tensorflow` 微信公众号上的一篇文章说用 `posenet` 来玩体感游戏的, 就中了草,
离我买 `kinnet` 已经快 10 年的了, 时间过得真快. 最近追剧 `完美关系` , 在里面看到张馨予玩的
体感游戏就去搜了下, 叫 `体感碰碰球` , 感觉很有意思. 还有最近很火的 `健身环大冒险`,
体感越来越接近生活. 刚好 `tensorflow 2020 开发大会` 上看到 `tenorflow hub` 上了 2 个新模型,
有个叫 `handpose` 追踪手势轨迹的, 就想用来做个利用摄像头追逐手势的体感碰碰球.
先上了个简单的, 利用一个点来控制飞机飞行.

源码: [HandPlaneGame](https://github.com/dust8/HandPlaneGame)  
查看 demo: [HandPlaneGame](https://dust8.github.io/HandPlaneGame/)

![截图](/blog/assert/2020-03-17.gif)

## 参考链接

- [MediaPipe Handpose](https://github.com/tensorflow/tfjs-models/tree/605fc8578b55b792a0acc6a94f242ac242387b33/handpose)
- [The Making of “The Aviator”: Animating a Basic 3D Scene with Three.js](https://tympanus.net/codrops/2016/04/26/the-aviator-animating-basic-3d-scene-threejs/)
- [用 PoseNet + TensorFlow.js 在浏览器实现体感游戏](https://mp.weixin.qq.com/s?__biz=MzU1OTMyNDcxMQ==&mid=2247487624&idx=1&sn=c8fc996120c08d55a46934bc904acd8b)
- [three.js 官网](https://threejs.org/)
- [Three.js 零基础入门教程](http://www.webgl3d.cn/Three.js/)
- [TensorFlow.js 遇到小程序](https://ke.qq.com/course/428263)
