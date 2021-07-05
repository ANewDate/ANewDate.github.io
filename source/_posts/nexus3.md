---
title: nexus私服
date: 2021-07-05 10:48:16
updated: 2021-07-05 10:48:16
tags: [docker, nexus]
excerpt: win10下使用docker部署nexus3; 写一个简单的前端脚手架并发布到nexus
index_img: /img/nexus/10001.jpg
categories: 
- web前端
---

写了一个简单版(git clone)功能的node脚手架工具, 在部署到公司nexus私服的过程中遇到了一些问题, 发现自己对nexus私服不是很熟悉(我司的nexus私服, 是乙方人员搭建的, 架构师疲于业务), 遂自己在本地练习了一波nexus搭建npm repository, 发布脚手架, 记录自己部署过程遇到的问题
### nexus搭建

1. 起容器
```shell
docker run -d -p 10001:8081 --name nexus3 --mount src=nexus-data,target=/nexus-data sonatype/nexus3
d16fae5c6f7f27c9de96efdc20eddd2c8ef7b9aaf8a39c317716d380a7e82c56
docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                                         NAMES
d16fae5c6f7f   sonatype/nexus3   "sh -c ${SONATYPE_DI…"   6 seconds ago   Up 4 seconds   0.0.0.0:10001->8081/tcp, :::10001->8081/tcp   nexus3
```
2. 查看容器日志是否启动成功
```shell
docker logs d16f -f
```
查看容器日志启动成功后, 访问本地10001端口即可看到nexus界面
{% gi total n1-n2... %}
  ![](/img/nexus/start.jpg)
  ![](/img/nexus/10001.jpg)
{% endgi %}

3. 登录nexus
默认账号: admin, 密码需要到**挂载的卷nexus-data中找到admin.password文件**
<a href="#win10" style="text-decoration: underline; cursor: pointer">*win10如何查看docker数据卷*</a>
查看卷下的admin.password文件便可查看密码; 登录,登录后设置新密码
{% gi total n1-n2-n3... %}
  ![](/img/nexus/psd.jpg)
  ![](/img/nexus/login.jpg)
  ![](/img/nexus/newpsd.jpg)
{% endgi %}
**就这样, nexus部署成功**

### npm repository创建
1. 先创建一个新的Blob Stores,相当于一个新的文件夹(存放npm hosted类型repository的内容), 可以在数据卷找到这个文件夹
{% gi total n1-n2-n3... %}
  ![](/img/nexus/blob1.jpg)
  ![](/img/nexus/blob2.jpg)
  ![](/img/nexus/blob3.png)
{% endgi %}
2. 创建三种类型的 npm repository
<p class="note note-primary">npm hosted: 私有仓库, 用于存储内部发布的npm包</p>
<p class="note note-primary">npm proxy: 代理仓库, 代理例如淘宝源镜像</p>
<p class="note note-primary">npm group: 仓库组, 组合其他仓库</p>

![](/img/nexus/npm3.jpg)
创建三种类型的 npm repository
{% gi total n1-n2-n3... %}
  ![](/img/nexus/hosted.jpg)
  ![](/img/nexus/proxy.jpg)
  ![](/img/nexus/group.jpg)
  ![](/img/nexus/cpy.jpg)
{% endgi %}
### 写一个简单的脚手架,
这个脚手架的用法: `arrow init demo`, 相当于 `git clone xxx.git demo`
1. `npm init -y` 初始化
2. 主要代码实现(虽然只有 git clone 功能)
{% gi total n1-n2-n3... %}
  ![](/img/nexus/pkg.png)
  ![](/img/nexus/sh.png)
{% endgi %}
3. 本地测试
`npm link` 相当于发布到本地全局node_modules包, 包名是package.json里的name, 脚本名为package.json#bin的属性名
{% gi total n1-n2... %}
![](/img/nexus/link.png)
![](/img/nexus/suc.png)
{% endgi %}

### npm publish

<a href="#error">必看: **发布npm包遇到的错误**</a>

1. `npm login`, 注意此处仓库地址是npm hosted 类型的仓库(否则会报400的错误)
![](/img/nexus/npmlogin.png)
2. `npm publish`, 以及nexus添加前后对比
{% gi total n1-n2-n3... %}
![](/img/nexus/npmpublish.png)
![](/img/nexus/bp.jpg)
![](/img/nexus/ap.jpg)
{% endgi %}
3. 本地测试
首先unlink本地全局包, nrm添加源(此处用npm group类型的仓库地址), 安装脚手架
{% gi total n1-n2-n3... %}
![](/img/nexus/ul.png)
![](/img/nexus/nrm.png)
![](/img/nexus/end.png)
{% endgi %}

### TIPS
### <span id="win10">win10查看docker数据卷</span>
1. [microsoft官方文档开启Hyper-V](https://docs.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)
2. 打开Hyper-V,连接dockerDesktop
![](/img/nexus/hyper-v.jpg)
3. `cd /var/lib/docker/volumes` 便能查看到docker的数据卷 

### <span id="error">发布npm包遇到的错误</span>
1. 404, 无法找到包
<p class="note note-danger"> Not Found - PUT https://registry.npmjs.org/arrow-cli - Not found</p>

**出现原因左图, 解决办法: 右图**
{% gi total n1-n2-n3... %}
![](/img/nexus/404.png)
![](/img/nexus/pkg2.png)
{% endgi %}

2. 401
<p class="note note-danger">Unable to authenticate, need: BASIC realm="Sonatype Nexus Repository Manager"</p>

**出现原因左图, 解决办法: 右图**
{% gi total n1-n2-n3... %}
![](/img/nexus/401.png)
![](/img/nexus/E401.jpg)
{% endgi %}





