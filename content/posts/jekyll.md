---
title: jekyll 打造静态博客
date: 2018-12-07
categories:
  - tools
tags:
  - ruby
  - jekyll
---

使用 jekyll 打造静态博客
<!--more-->

## 1. 安装之前先安装依赖包

```
sudo apt-get install ruby ruby-dev build-essential
```

## 2. 添加环境变量

```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
## 3. 安装 Jekyll

```bash
gem install jekyll bundler
```

## 4. 使用 bundle 更新 Jekyll

```
jekyll --version
gem list jekyll
bundle update jekyll
```
## 5. 使用 gem 更新 Jekyll

```
gem update jekyll
```
## 6. 更新 Rubygems

```
gem update --system
```
## 7. 安装预览版

```bash
## 安装最新预览版
gem install jekyll --pre
## 安装特定版本的 Jekyll
gem install jekyll -v '2.0.0.alpha.1'
```
## 8. 源码安装

```bash
git clone git://github.com/jekyll/jekyll.git
cd jekyll
script/bootstrap
bundle exec rake build
ls pkg/*.gem | head -n 1 | xargs gem install -l
```
