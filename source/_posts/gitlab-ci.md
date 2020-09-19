---
title: gitlab-ci前端自动发布(简单版)
date: 2020-09-16 0:51:05
tags: 前端自动发布
---
我司所用的自动部署技术栈是gitlab-ci + docker + rancher + harbor, 遂产生了自己手搭一个简单的前端自动部署的念头.
*虚拟机直接使用root用户了, 懒得换*

## 流程概述
  系统: centos7 
  1. 安装 gitlab
  2. 安装 nginx
  3. 安装配置 gitlab-runner

## 安装并部署 gitlab
  win10下Hyper-V启动一个centos7虚拟机, 参照以下下命令安装gitlab
  官方地址: [gitlab官网](https://about.gitlab.com/) [gitlab-ce](https://gitlab.com/gitlab-org/gitlab-ce)
  ### 1. 安装依赖
  ```shell
  yum install -y curl policycoreutils-python openssh-server
  #启动ssh服务&设置为开机启动
  systemctl enable sshd
  systemctl start sshd 
  ```
  ### 2. 安装 Postfix(可跳过)
  ```shell
  yum install -y postfix
  #启动ssh服务&设置为开机启动
  systemctl enable postfix
  systemctl start postfix 
  ```
  ### 3. 开放ssh以及http服务端口(80,8095)
  ```shell
  firewall-cmd --add-service=ssh --permanent
  firewall-cmd --add-service=http --permanent
  firewall-cmd --zone=public --add-port=8095/tcp --permanent
  firewall-cmd --reload
  #重载防火墙规则
  firewall-cmd --reload
  ```
  ### 4. yum 安装gitlab
  ```shell
  # 添加 package
  curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

  # 安装 gitlab-ce
  yum install -y gitlab-ce
  ```
  安装成功之后会打印gitlab图形
  ![gitlab-success](/img/gitlab/gitlab-success.png)
  ### 5. 配置gitlab访问地址
  gitlab的默认配置文件路径 `/etc/gitlab/gitlab.rb`
  修改`external_url 'http://gitlab.example.com'`
  ```shell
  # 打开配置文件
  vi /etc/gitlab/gitlab.rb

  # 修改配置文件 这里我设置为本机IP:8095
  external_url 'http://192.168.245.156:8095'
  ```
  ![gitlab-url](/img/gitlab/gitlab-url.png)
  ### 6. 重新配置并启动gitlab
  ```shell
  # 重新配置
  gitlab-ctl reconfigure
  # 完成后会看到以下输出
  Running handlers complete
  Chef Client finished, 432/613 resources updated in 04 minutes 20 seconds
  gitlab Reconfigured!
  # 重启
  gitlab-ctl restart
  ```
  ![reconfigure](/img/gitlab/reconfigure.png)
  ### 7. 访问gitlab
  浏览器地址栏输入 `http://192.168.245.156:8095/` 即可看到gitlab
  **第一次登录默认账号 root, 需设置密码**
  ### 8. 新建一个vue项目
  新建一个vue项目, 方便后面测试
  ![vue-demo](/img/gitlab/vue-demo.png)

## 安装 nginx
  ### 1. 添加源
  ```shell
  rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
  ```
  ### 2. install
  ```shell
  yum install nginx

  # 启动
  systemctl start nginx.service
  # 设置开机启动
  systemctl enable nginx.service
  ```
  ![nginx](/img/gitlab/nginx.png)
## 安装配置 gitlab-runner
  **ranner:** 读取并执行项目中的 `.gitlab-ci.yml文件`
  **gitlab 打开 setting > CI/CD 下runner选项**, 可以看到url, token的信息, 注册runner需要用到
  ![gitlab-runner](/img/gitlab/gitlab-runner.png)
  **不同版本的gitlab入口也不一样**
  ![gitlab-runner2](/img/gitlab/gitlab-runner2.png)
  ### 1. 安装
  ```shell
  # 添加package
  curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
  yum install gitlab-runner
  ```
  ### 2. 注册
  ```shell
  sudo gitlab-runner register
  # 填写 url token 相关信息, 选择执行器 shell

  # 注册成功后更改权限
  sudo chown -R gitlab-runner:gitlab-runner /home/gitlab-runner
  sudo chmod -R 777 /home/gitlab-runner
  ```
  ![runner-register.png](/img/gitlab/runner-register.png)
  
  成功之后在CI/CD界面看到
  ![runner-success.png](/img/gitlab/runner-success.png)

## 编写.gitlab-ci.yml 配置文件
  ### 1. 本地使用vue-cli初始化一个项目并关联远程仓库
  ### 2. 项目根目录创建.gitlab-ci.yml文件
  ```yml
    stages:
      - build
    cache:
      untracked: true
    build_dev:
      stage: build
      script:
        - npm --registry https://registry.npm.taobao.org install
        - npm run build
        - rm -fr /usr/share/nginx/html/*
        - mv -f dist/* /usr/share/nginx/html
      only:
        - master
      tags:
        - runner1
  ```
  ### 3. 查看gitlab构建
  ![success](/img/gitlab/success.png)
  ### 4. 访问 nginx 验证是否成功
  打开 http:192.168.245.154 即可看到vue界面

## Question
  ### 1. fatal: git fetch-pack: expected shallow list
  原因: git版本太低
  ![git](/img/gitlab/git.png)
  ```shell
  #安装源
  yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm
  #安装git
  yum install git
  #更新git
  yum update git
  ```
  ### 2. npm is not a command
  忘记安装node 及 npm
  ### 3. 没有权限读写html 目录
  ![nginx](/img/gitlab/Q2.png)
  开放权限
  ```shell
  chmod -R 777 /usr
  chmod -R 777 /usr/share
  chmod -R 777 /usr/share/nginx
  chmod -R 777 /usr/share/nginx/html
  ```
  ### 4. gitlab 502
  - 官方解决办法
  ![502](/img/gitlab/502.png)
  - unicorn[worker_timeout]时间设久一点
  - unicorn默认8080端口被占用
## 最后
  这是一个相对简单的前端自动部署, 无论是gitlab, gitlab-runner, nginx都是可以在docker容器中运行的。线下也尝试了以docker方式起nginx的方法, 但是由于发布都要修改镜像名称, 容器名称,(否则会出错), (又或者是在第一次发布之后, 在dockerfile中加入删除镜像, 容器的命令的方法)让我感觉很麻烦, 就没有使用这种方式, 有时间的话后续可以加入 harbor, rancher 做一个完整的自动发布。写着写着快凌晨一点了, ~good night~



