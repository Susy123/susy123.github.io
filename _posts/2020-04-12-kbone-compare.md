---
layout: post
title: Kbone与其他框架原理比较
date: 2020-04-12
Author: Susy123
tags: [vue, kbone]
comments: true
toc: true
---

## 整体流程图

![整体流程图](https://gitee.com/sanchuanhi/imgbed/raw/master/img/kbone同构方案.png)

## 常见做法

业界常见做法是将 template 直接转成小程序的 WXML 模板。这种做法更接近小程序原生性能，但也作出了很多妥协牺牲。Vue 模板和 WXML 模板的语法并不是直接对等的，Vue 的特性设计也和小程序的设计无法划等号，这自然就导致了部分 Vue 特性的丢失。比如像 Vue 中的 v-html 指令、ref 获取 Dom 节点、过滤器等就通通用不了。当然不止是 Vue 自身的特性，一些原本依赖 Dom/Bom 接口的 Vue 插件也无法使用，比如 Vue-router 等。

只能取 web 和小程序能力的并集。好在 web 类技术大同小异，仅仅是并集也能满足绝大多数普通应用场景。

## Kbone 处理方式

Kbone 的这种方式实现了 web 端的整套体系，对于开发者来说，只需要按照 web 端的方式进行开发即可，而无须关注同构差异。

Kbone 对各个特性的支持全面、对代码编写的要求少以及自由度高、不需要魔改 Web 框架的底层实现，这样对于代码的维护、升级也都更为简单方便。

Kbone 本身是通过牺牲性能来换取更全面的 Web 端兼容。在 500 节点内的两个方案本身性能差不多，不过因为自定义组件实例创建的开销，在千节点往上的情况下会落后于静态模板方案。通常一个小程序页面的节点数在 100-500 这个区间浮动，因此这个表现是符合预期的。

## 小程序组件树 和 web DOM 树

组件树和 Dom 树很类似，区别是组件树是由小程序内置组件或自定义组件拼接而成，而不是 由我们熟悉的 Dom 节点拼接而成。

## 仿照 DOM 树 -> 组件树

仿照 Dom 树此时仍在逻辑层中。

如何处理这种动态的、无法预知各分支深度的树形结构？大家肯定会想到“递归”。

正好小程序的自定义组件有自引用的特性，我们就可以利用这个特性来实现递归处理。终止条件是特定节点、文本节点或者孩子节点为空。

## 性能对比

页面渲染耗时(动态测试)
| | 页面渲染耗时 |
| ------------ | ------------ |
| native | 64 |
|wepy2 | 64 |
| uniapp | 56.4 |
| mpx | 52.6 |
| chameleon | 56.4 |
| mpvue | 117.8 |
| kbone | 98.6 |
| taro next | 89.6 |

## reference

[小程序框架运行时性能大测评](https://juejin.im/post/5e868ac751882573ab44f5d4 '小程序框架运行时性能大测评')

[Kbone 解决 Web 端和小程序同构痛点](https://mp.weixin.qq.com/s/OcaCg9ru94rHieTEV7wS0w 'Kbone解决Web 端和小程序同构痛点')

[个人 blog](https://susy123.github.io/)