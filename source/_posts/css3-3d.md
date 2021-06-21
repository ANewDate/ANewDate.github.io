---
title: css3 画一个长方体
date: 2021-06-21 15:21:36
updated: 2021-06-21 15:21:36
tags: css
index_img: /img/css/css3d.gif
excerpt: 效果:css3 画一个长方体, 并根据其中间轴进行旋转. 加强自己对css3transform属性的理解
categories: 
- web前端
---
![](/img/css/css3d.gif)
源代码查看: [codepen](https://codepen.io/honorgo/pen/PopgVrr)
注意点: 
1. **当元素设置rotate属性时, 其坐标系也会跟着旋转**
2. 坐标系方向:
![](/img/css/zuobiaoxi.png)
- x/y/z轴正方向分别为: 右, 下, 外
- 旋转方向判断: 左手握住旋转轴, 拇指指向旋转轴正方向, 手指卷曲方向即为旋转方向(顺时针方向)
3. perspective 视距, 模拟透视效果时人眼的位置
![](/img/css/perspective.jpg)
4. 设置`perspective`属性的时候, 要设置在具有`transform-style: preserve-3d`属性的块的父块上, 不然看到的就像一个梯形
![](/img/css/tixing.gif)
5. 要让长方体绕中间轴进行旋转, 需要两个点确定中心轴
- `transform-origin` 默认center / (0,0,0), 详细可看[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-origin)
- `transform: rotate3d(x, y, z, 0deg)`
旋转轴为连接`origin`的坐标(起始点), 到`rotate3d` 的点(终点)的向量
6. `transform-style: preserve-3d;` 设置**子元素**位于3D平面中




