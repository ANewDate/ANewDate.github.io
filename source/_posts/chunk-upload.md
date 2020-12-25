---
title: 大文件分片上传学习笔记
date: 2020-09-19 14:27:32
updated: 2020-12-19 14:27:32
tags: vue
---
最近公司业务需要做PC端大文件分片上传, 这里记录一下自己对大文件分片上传的学习

## 思路
  ![chunk](/img/rp/fileUpload.png)
  **分片验证**: 在每个分片上传前都验证此分片是否已上传, 后端返回此分片是否已上传; 又或者是, 后端一次返回所有已上传分片的index, 前端根据此判断哪些分片该跳过, 哪些该上传

## 优秀的开源插件
  - [simple-uploader](https://github.com/simple-uploader/Uploader): 优秀上传库, 支持多并发上传，文件夹、拖拽、可暂停继续、秒传、分块上传、出错自动重传、手工重传、进度、剩余时间、上传速度等特性
  - [vue-simple-uploader](https://github.com/simple-uploader/vue-uploader/blob/master/README_zh-CN.md): 基于`simple-uploader`的文件上传封装, 加了一些额外的options
  - [SpringBoot+Vue.js前后端分离实现大文件分块上传](https://github.com/LuoLiangDSGA/spring-learning/tree/master/boot-uploader): 基于`vue-simple-uploader` 实现 大文件分片上传

## 念叨念叨
  1. 在做文件分片上传之前, 最先看的是 [el-upload](https://github.com/ElemeFE/element/blob/dev/packages/upload/src/upload.vue)的源码,  发现把 变量fileList 放在 upload.vue, 很有react状态提升的味道 (我什么时候才能学会用react做项目啊)
  2. 最后因为公司情况 *或许某时某刻有些东西真的并不需要太完美*,  用了 [webuploader](http://fex.baidu.com/webuploader/) , 怎么说呢，这个库并不支持单个文件的暂停/重新上传(即便文档说有支持, 但是测试之后发现并不支持), 这个是不我不太喜欢的地方, 但最后我还是把`webuploader`封装到我司的开发平台, 毕竟能跑的东西都不会太差

