---
layout: post
title: Vue悬浮球效果
date: 2020-04-28
Author: Susy123
tags: [vue, floatball]
comments: true
toc: true
---

## 知识点：

悬浮球有两个知识点：

1）一个是吸边效果。位置判断，然后缓动到目标位置：

获取元素位置属性：this.\$refs.refNamexxxx.getBoundingClientRect()

2）一个是页面滚动过程中隐藏悬浮球的防抖。

## 实现思路：

1、created 里获取 document.documentElement.clientWidth 和 clientHeight，（这里插入知识点 document.documentElement 和 document.body 的差别）

2、mounted 里设置初始位置，并绑定悬浮球的 touchstart touchmove touchend 以及 window.scroll 事件

3、touchstart 里设置禁止点击以免 touch 和 click 事件冲突，同时让 transition 为 none 因为跟随手指移动时不需要缓动效果。

4、touchmove 中设置悬浮球跟随手指拖动实时改变位置，并且在这里可以设置悬浮球回复可点击。

5、touchend 里首先判断悬浮球是否可点击，如上所说以免 touch 和 click 事件冲突。然后把 transition 改为 all 0.3s，为了停止拖动后吸边动作的缓动效果。最后就是做吸边动作，将悬浮球的 left top 设置为目标位置。

6、页面 scroll 过程中，悬浮球隐藏操作，是在 scroll 里隐藏悬浮球，然后 scroll 里做一个防抖，等 scroll 停下来 200ms 后执行悬浮球回复显示。

7、最后很重要的是，别忘记在 beforeDestroy 里 removeEventListener，做一个有始有终良好习惯的程序员。

![float-ball](https://user-gold-cdn.xitu.io/2020/4/28/171bf4ea0a5f05e2?w=2006&h=1870&f=png&s=567275)

## reference

https://segmentfault.com/a/1190000022000839

https://blog.csdn.net/qq_41009742/article/details/101516232

https://blog.csdn.net/weixin_43494704/article/details/88753167

https://www.cnblogs.com/richard1015/p/9605115.html

https://juejin.im/post/5c237c5ce51d453603630837

https://github.com/5ibinbin/vue-draggable-float

https://github.com/vencent2017/vue-drag-ball

https://github.com/system-cpu/vue-cli3-float

[个人 blog](https://susy123.github.io/)
