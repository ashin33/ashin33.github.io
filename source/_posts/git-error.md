---
title: 解决Fatal:Not a gitflow-enabled repo yet. Please run 'git flow init' first
date: 2021-04-08 10:14:38
tags: [2021,git]
categories: [error]
top_img: /img/error.jpeg
cover: /img/error.jpeg
---

>今天sourcetree操作git，报错Fatal: Not a gitflow-enabled repo yet. Please run 'git flow。记录一下解决方案。
### 问题![251617846939_.pic](/images/251617846939_.pic.jpg)

### 解决
#### 打开项目下的.git/config文件，并删除所有gitflow的配置
![](/images/16178492522634.jpg)
#### 删除后保存config，并重启sourcetree，重新点击git工作流=>初始化仓库![261617847178_.pic_hd](/images/261617847178_.pic_hd.jpg)

#### 重新执行之前报错的git操作，不再报错