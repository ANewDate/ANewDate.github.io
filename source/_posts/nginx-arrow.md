---
title: nginx代理
date: 2021-08-09 10:22:32
updated: 2021-08-09 10:22:32
tags: [前端自动发布, nginx]
excerpt: 业务需求:为了加快图片加载速度;授权码登录中的代理请求头设置; 业务场景:公司有一移动端H5页面图片比较多, 即便做了图片压缩, 图片懒加载, 加载速度还是不够理想, 需要在Nginx做了图片缓存加快加载速度; 另一个安全性的问题是在做授权码登录的过程中需要隐藏应用的`client_secret`
index_img: /img/nginx/hit.jpg
categories: 
- web前端
---
业务场景: 
1. 公司有一移动端H5页面图片比较多, 即便做了图片压缩, 图片懒加载, 加载速度还是不够理想, 需要在Nginx做了图片缓存加快加载速度
2. 公司系统过多, 为解决一个登录页可登录不同应用(每个系统对应不同的`client_id`), 在公用的登陆页面中登录的时候, 带上获得的授权码`code`重定向到对应系统A, 系统A获取到授权码`code` 发送`code&&client_id&&client_secret`到后端进行安全校验, 基于安全性的考虑,系统A的`client_secret`不可放在前端代码中, 需要设置Nginx代理请求头

## 1. 图片缓存
### 1.1 nginx路径匹配优先级
1. 全路径匹配 `location = path`
2. 前缀匹配  `location ^~ path`
3. 正则匹配 区分大小写: `location ~ path` 不区分大小写: `location ~* path`
4. 路径匹配 `location path`

### 1.2 设置图片缓存区
通过 `http.proxy_cache_path` 配置项设置图片缓存区
```nginx
# /home/nginx 缓存内容存放路径
# levels 存放目录的结构, 可以使用任意的1位或2位数字作为目录结构, 最多三级
# keys_zone=images:10m 内存缓冲区命名为images,内存大小10m
# max_size=2000m 磁盘最多缓存2000m文件, 如果超过则会删除最少使用的文件
# inactive=10 如果文件10分钟内没有被访问, 将会从缓存中删除
# ----下面两个配置是加载器的启动策略, 防止nginx在启动后的几分钟里变得缓慢
# loader_threshold=300 缓存加载器每次迭代过程最多执行300毫秒
# loader_files=200  缓存加载器加载器每次迭代过程中最多加载200个文件
http{
  proxy_cache_path /home/nginx levels=1:2 keys_zone=images:10m loader_threshold=300 loader_files=200 max_size=2000m inactive=10d;
}
```
### 1.3 设置图片缓存
```nginx
#expires 可设置浏览器设置的过期时间
#proxy_pass 可设置代理的文件服务器路径
#add_header Nginx-Cache "$upstream_cache_status"; 可用于查看缓存是否命中 MISS/HIT
  location ~ /group.*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz)$ {
      proxy_pass ${FILE_REQUEST_DOMAIN_PREFIX};
      proxy_cache images;
      expires 30d;
      add_header Nginx-Cache "$upstream_cache_status";
  }
```

## 2. location.proxy_set_header
```nginx
http {
  server {
    location /api{
      proxy_set_header Authorization "client_secret";
    }
  }
}
```
