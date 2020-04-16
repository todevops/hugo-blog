---
title: Hexo 打造静态博客
date: 2018-12-09
categories:
  - tools
tags:
  - hexo
  - nodejs
---

使用 Hexo 打造静态博客
<!--more-->

```
npm install hexo-deployer-git --save
```


## 修改主页配置文件 _config.yml
```
post_asset_folder : true
```
## 安装本地图片插件
```
npm install hexo-asset-image --save
```
## 生成新的博文
```
hexo n "xxx"
```
此时 /source/_posts/ 下会生成同名文件夹

## 博文引入图片
```
![](xxx/xxx.jpg)
```
