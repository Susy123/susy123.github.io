---
layout: post
title: vue-cli利用webpack-bundle-analyzer进行包分析
date: 2020-03-31
Author: Susy123
tags: [vue-cli, webpack-bundle-analyzer, --report]
comments: true
toc: true
---

## webpack-bundle-analyzer 分析报告

大家都看过这种 webpack 分析报告吧，是由插件 webpack-bundle-analyzer 生成，用来在论文、ppt 或者博客里炫耀是最好用不过了。

当然它的主要功能还是帮助我们分析 webpack 打包之后的结果，观察各个静态资源的大小。

![bundle-report](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160217.png)

好消息是 vue-cli 官方已经集成了这个插件功能。

![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331154428.png)

可是第一次使用 npm run build --report 却发现并没有生成 report.html;

傻眼。。。

## vue-cli 中使用

网上搜索一下，都是让在 vue.config.js 里 require webpack-bundle-analyzer, 然后 configureWebpack 里增加 plugins。

![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160237.png)

改完 vue.config.js 后，运行 npm run build 就能自动打开一个网页，运行在 8888 端口，显示 webpack 分析报告。

可是之前就是这样用啊，vue-cli 官网说已经支持了，不可能还需要我们自己手动修改 vue.config.js 吧？这个功能实现很简单，不可能留给我们提 pr 的机会吧。。。

找到 cli-service 的执行代码，
![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160309.png)

看这个情况是明显是已经支持了，不需要手动改 vue.config.js。

那问题出在哪里呢。加点 log 看一下，args 的 report 根本是 false 啊。

![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160326.png)

再仔细一看，npm 根本就没有把 --report 这个选项传下去。

![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160345.png)

换用 yarn 试一下：

![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160402.png)

人家 yarn 可以啊，npm 为什么不行？

不可能吧，这个全球工具不会给我留 pr 机会吧。。。

## 正确使用姿势

扒一扒官方文档，

![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160420.png)
npm 对不起，是我不够了解你 。再来一次：
![](https://gitee.com/sanchuanhi/imgbed/raw/master/img/20200331160434.png)
这才是正确的姿势。查看 dist 目录下，已经有了 report.html。

（还是觉得这个用法有点不符合习惯。）

当然除了用 yarn build --report, 也可以在 package.json 里面增加 scripts. 只要姿势对，repoort.html 就会出现。

[个人 blog](https://susy123.github.io/)
