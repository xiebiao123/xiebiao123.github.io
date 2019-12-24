---
title: Vue常见问题
date: 2019-07-19
categories:
    - 学习
tags:
    - vue
---

### [*!!vue-style-loader!css-loader?{"sourceMap":true}!../../../../vue-loader类似问题的](https://blog.csdn.net/genius_yym/article/details/82222424)
* 问题描述

*!!vue-style-loader!css-loader?{“sourceMap”:true}!../../../../vue-loader/lib/style-compiler/index?{“vue”:true,”id”:”data-v-570115ee”,”scoped”:false,”hasInlineConfig”:false}!../../../../vux-loader/src/after-less-loader.js!less-loader?{“sourceMap”:true}!../../../../vux-loader/src/style-loader.js!../../../../vue-loader/lib/selector?type=styles&index=0!./index.vue in ./node_modules/vux/src/components/alert/index.vue


* 解决方案

此类问题一般是缺少相关依赖而导致的，对于本例，仔细看一下报错提示信息，抓住关键词，vue-style-loader!css-loader，说明是css解析的时候出了问题。 
所以，解决方案就要根据情况而定，看你使用的CSS语言是什么,是 常规的 或者 less 或者 sass。

如果是 常规 的，执行 npm install stylus-loader css-loader style-loader --save-dev 安装依赖就行。
如果是 less 的，执行 npm install less less-loader --save-dev 安装依赖就行。
如果是 sass 的，执行 npm install sass sass-loader --save-dev 安装依赖就行。或者（$npm intall sass-loader --save ; $npm install node-sass --save）
如果你不知道，好吧，你三个都执行吧。 
一般只有在初始化配置的的时候才会出现这个问题，如果是已经完好的项目都会在package.json中已经配好，直接install即可。

### [强大的css预编译stylus以及在vue中使用stylus](https://www.jianshu.com/p/8601ccf91225)

### [解决webstorm中vue语法没有提示](https://blog.csdn.net/weixin_42795449/article/details/84112876)

### [解决webstorm中vue语法高亮显示](https://blog.csdn.net/weixin_42795449/article/details/84112876)