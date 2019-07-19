---
title: Hexo + Github Pages + yilia 搭建个人博客
date: 2019-07-02
categories:
	- 学习
tags:
    - Hexo
---
#### Github Pages 是什么
GitHub Pages 本用于介绍托管在GitHub的项目，不过由于他的空间免费稳定，很适合用来搭建一个博客。每个帐号只能有一个仓库
来存放个人主页，而且仓库的名字必须是username/username.github.io，这是特殊的命名约定。你可以通过http://username.github.io
来访问你的个人主页，比如我的就是xiebiao123.github.io

    这里特别提醒一下，需要注意的个人主页的网站内容是在master分支下的

#### 创建自己的Github Pages

    新建代码仓库(repository),此处命名格式有限制的，形如username.github.io
    
#### 环境搭建
1. 安装node.js
2. 安装git

<!-- more -->

##### Hexo 安装
```
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo g # 或者hexo generate
$ hexo s # 或者hexo server，可以在http://localhost:4000/ 查看（hexo s -p 8023端口被占用时）
```


##### 切换主题
* 安装主题
```text
Hexo默认主题是landscape，大家可以切换成为next或者yilia的风格
$ hexo clean
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
* 切换主题
```
修改Hexo目录下的_config.yml配置文件中的theme属性，将其设置为yilia
```
* 更新主题
```text
$ cd themes/yilia
$ git pull
$ hexo g # 生成
$ hexo s # 启动本地web服务器
```
 
#### 部署Hexo到Github Pages
* 部署Hexo到Github Pages上的原理
```text
1. 第一步中我们在Github上创建了一个特殊的repo（gladysgong.github.io）一个最大的特点就是master中的html静态文件，可以通过
链接http://username.github.io来直接访问。
2. Hexo -g 会生成一个静态网站（第一次会生成一个public目录），这个静态文件可以直接访问。
3. 将Hexo生成的静态网站，提交(git commit)到github上。
```
* 安装拓展插件（不安装会出错）
```text
$ npm install hexo-deployer-git --save
```
* 发布到远程仓库
```text
hexo d
```

#### Hexo 部署后把原来的仓库覆盖
* 使用Hexo搭建博客需要区分【博客源代码】和【博客生成代码】
```text
『博客源代码』：Hexo的源码，包括themes目录（博客模板），source目录(使用MarkDown写的博客)等
『博客生成代码』：执行hexo generate或者hexo server命令生成的代码，是Hexo自动生成的，在public目录里面。
『博客源代码』需要使用Git做版本管理，而『博客生成代码』需要使用Git部署。因此容易混淆。
```
* 方法一：使用两个不同的git仓库分别管理
```text
在GitHub新建一个仓库，然后将『博客源代码』同步到新项目。『博客生成代码』仍然由gladysgong/gladysgong.github.io部署。
```
* 方法二：使用同一个仓库的两个不同的git分支管理
```text
修改Hexo的配置文件_config.yml，将『博客生成代码』部署到gladysgong/gladysgong.github.io仓库的develop分支:
deploy:
    type: git
    branch: develop
    repo: https://github.com/xiebiao123/xiebiao123.github.com
```