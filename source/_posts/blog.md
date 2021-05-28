---
layout: _posts
title: 通过hexo+github+trvis-ci搭建个人博客
date: 2021-03-18 18:09:62
tags: [2021,代码]
categories: [代码]
top_img: /img/hexo-github.jpeg
cover: /img/hexo-github.jpeg
---
## github
### 1. 在github中新建仓库并命名为  github用户名.github.io的仓库

![1](https://i.loli.net/2021/03/26/Q1DoKuJkzL48s7S.jpg)


### 2.新增token

![2](https://i.loli.net/2021/03/26/Przm8NMSjgyO1Q9.jpg)
![3](https://i.loli.net/2021/03/26/V2EOLmQd9o5puJ6.jpg)
![4](https://i.loli.net/2021/03/26/LMAYFTIH26Baxhr.jpg)

保存好生成的 token，后面用于配置 travis-ci

### 3.新建一个hexo分支用于存储后面的项目（分支可以自定义，后面配置中保持一致即可）

## 安装node.js
mac下载地址：[https://nodejs.org/en/](https://nodejs.org/en/)  
其他系统，请自行寻找，如不本地调试，node.js也可以不装
>node.js我使用的12.21.0版本,最新版的node.js在hexo构建后会报错
```
(node:25044) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(Use `node --trace-warnings ...` to show where the warning was created)
(node:25044) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:25044) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
(node:25044) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(node:25044) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:25044) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
```

## 全局安装hexo
```
npm install -g hexo-cli
```

## 添加环境变量
```
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
```

## 运行
```
$ hexo init ashin33 
$ cd ashin33
$ npm install
```
在指定目录下(上面为我的文件夹，请根据个人实际情况修改)生成hexo项目

## 运行以下命令就可以预览你的博客了
```
hexo clean ## 用于清除已构建的文件  
hexo generate ##或者 hexo g  构建项目  
hexo server ##或者 hexo s
##也可以`hexo s -g` 一步更比两步强
```

##  安装hexo主题
```
git submodule add https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```
* 此处选择了butterfly主题，更多主题请移步 [https://hexo.io/themes/](https://hexo.io/themes/)
* 此处使用了git submodule 子模块加载命令，用于后续我们主项目推送时，将主题可以一起推到项目里，否则后面travis-ci自动构建会报错，找不到标签等错误

##  安装pug和stylus渲染器,用于渲染主题
```
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

## hexo博客发布与butterfly主题的具体使用，参考各自文档
hexo文档：[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)  
butterfly文档：[https://butterfly.js.org/posts/21cfbf15/](https://butterfly.js.org/posts/21cfbf15/)

## hexo配置
修改项目根目录下的_config.yml文件
```
title: 神的孩子都在跳舞  
subtitle: ''  
description: 神的孩子都在跳舞  
keywords:  
author: Alon  
language: zh-CN  
timezone: Asia/Shanghai  
url: https://ashin33.github.io/  ##github分配的静态页面地址
```

![5.jpg](https://i.loli.net/2021/03/26/TrIVbHum86j1d5K.jpg)
```
theme: butterfly  ##themes目录下的主题目录
deploy:  
type: git  
repo: https://github.com/ashin33/ashin33.github.io.git  ##仓库地址
branch: hexo  ##代码要推送的分支
```

![6](https://i.loli.net/2021/03/26/cLE3u4IDjmJQreF.jpg)

## butterfly配置
* 复制themes/butterfly下的_config.yml文件到项目根目录下并重命名为_config.butterfly.yml
* hexo构建时会合并_config.yml和_config.butterfly.yml的配置
* 在此修改butterfly的配置，在后面更新主题是就不会影响配置了

## 配置travis-ci用于自动构建
*  不使用travis-ci手动构建也是可以的，可以直接使用  
`hexo deploy -g`  或者  `hexo d -g`,此命令会根据第9步中配置的deploy的参数直接构建并推送

### 1. 在[travis-ci官网](https://travis-ci.com/)用github账号登录 ，并激活，如图

![7](https://i.loli.net/2021/03/26/3k7ORZsuLyAXYGM.jpg)
![8](https://i.loli.net/2021/03/26/mFqDLaOQPgoJTRZ.jpg)

### 2. 进入travis-ci中的自己的项目中进入setting中，按下图进行设置

![9](https://i.loli.net/2021/03/26/FPs3cCAg2hpndUE.jpg)

![10](https://i.loli.net/2021/03/26/rAjgG3QofNq4PJb.jpg)

### 3.在项目根目录新增travis-ci的配置文件`.travis.yml`，并写入以下内容

``` 
# 指定语言环境
language: node_js
# 指定需要sudo权限
sudo: required
# 指定node_js版本,版本过低的话构建时travis会报错Uncaught SyntaxError: Unexpected token....
node_js:
  - 12.21.0
# 指定缓存模块，可选。缓存可加快编译速度。
cache:
  directories:
    - node_modules

# 指定博客源码分支，因人而异。hexo博客源码托管在独立repo则不用设置此项
branches:
  only:
    - hexo

before_install:
  - npm install -g hexo-cli

# Start: Build Lifecycle
install:
  - npm install
  - npm install hexo-deployer-git --save
# 执行清缓存，生成网页操作
script:
  - hexo clean
  - hexo generate

# user.name替换为自己的github用户名
# user.email替换为自己的github邮箱
# ${travis_ci}中的travis替换为自己github中的token名
# 这样每次推送代码，travis会自动在master分支中构建hexo
after_script:
  - cd ./public
  - git init
  - git config user.name "ashin33"
  - git config user.email "ashin_33@163.com"
  - git add .
  - git commit -m "Travis ci push"
  - git push --force --quiet "https://${travis_ci}@${gh_repo}" master:master
env:
  global:
    - gh_repo: github.com/ashin33/ashin33.github.io.git
# End: Build LifeCycle
```
### 5.推送干净的项目代码（hexo clean过的项目代码）到hexo分支
### 6.代码推送后，进入travis中可以看到构建过程，等待构建成功，访问[https://ashin33.github.io](https://ashin33.github.io/) 即可，网址中的ashin33.github.io替换为你的仓库名即可