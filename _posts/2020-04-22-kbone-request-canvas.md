---
layout: post
title: kbone request、canvas及屏幕适配同构方案
date: 2020-04-22
Author: Susy123
tags: [vue, kbone]
comments: true
toc: true
---

## 一 canvas

一个好消息是小程序基础库 v2.9.0 正式开放一套全新的 Canvas 接口。该接口符合 HTML Canvas 2D 的标准，实现上采用 GPU 硬件加速，渲染性能相比于现有的 Canvas 接口有一倍左右的提升。

新版 Canvas 2D 接口与 Web 几乎完全一致了。这对 H5 和小程序的同构有很大的便利。

要注意的就是新版的 Canvas 可以不需要 canvas-id，但是必须要 type="2d"；

目前是两种 context 是共存的。

kbone 里通过$$getContext的方式画图，canvas节点里必须要有canvas-id；通过 $$prepare 的需要有 type="2d"

### canvas 在小程序里空白

初次使用 canvas，常会遇到在小程序里空白的问题，踩过的坑如下：

#### 1 canvas 没显式定义宽高

没有显式定义宽高的情况下，web 还能显式正常，默认宽高是 300\*\*150：
![](https://user-gold-cdn.xitu.io/2020/4/22/171a2538616599d7?w=1040&h=612&f=png&s=64298)

miniprogram 里没有显式定义宽高的情况下，虽然默认宽高也是 300\*\*150，但是画布空白（这一点 kbone 可以做到才对，计划有时间就去看下这个问题）：
![](https://user-gold-cdn.xitu.io/2020/4/22/171a253861918125?w=830&h=914&f=png&s=86425)

以下 2 种方式设置宽高都可以，看业务场景选择，但是注意不能用 style 设置宽高，内联的也不行：

1、
![](https://user-gold-cdn.xitu.io/2020/4/22/171a2538680666a6?w=1252&h=52&f=png&s=22995)

2、
![](https://user-gold-cdn.xitu.io/2020/4/22/171a2538689b800b?w=1308&h=1226&f=png&s=266551)

显式设置后再看下 web 和 mp 的表现：
![](https://user-gold-cdn.xitu.io/2020/4/22/171a25386f84f8e8?w=1086&h=1014&f=png&s=87016)
![](https://user-gold-cdn.xitu.io/2020/4/22/171a25386bc1a10a?w=768&h=1164&f=png&s=95500)

#### 2 canvas 获取 dom 未成功

还有一种情况是在小程序里获取 dom 没有成功。
可以将在获取 dom 后将 dom 打印出来看看。
遇到的一个没有结论的问题是：

通过 ref 获取 canvas dom 还是通过 querySelector/getElementById
![](https://user-gold-cdn.xitu.io/2020/4/22/171a25389ff7529e?w=992&h=238&f=png&s=58246)

通过 ref 获取，可以看到 web 是没有问题的，和用 id 获取是一致的。但是 mp 里，曾经遇到过 canvas.js 里 res 节点为 null 的问题，即\$\$prepare 回调函数的参数 domNode 为 null。但是后面再跑又莫名消失了。尚未定位原因。
![](https://user-gold-cdn.xitu.io/2020/4/22/171a2538a028a634?w=2042&h=1288&f=png&s=403526)

## 二 request

request：本来是计划用 kbone-api 里面提供的 request api，但是它的 request 里对 url 要求是 http https 完整路径的请求格式：
![kbone-api-request-validUrl](https://user-gold-cdn.xitu.io/2020/4/22/171a2538a29e09f5?w=940&h=318&f=png&s=60231)

这种情况下无法用 devserver 的 proxy 功能（除非自己起一个接口服务）。为了开发方便，更换为了 axios 和 axios-miniprogram-adapter，下面是我们二次封装的 request 模块：

```javascript
// src/request/http.js
/* eslint-disable */
import axios from 'axios';
import mpAdapter from 'axios-miniprogram-adapter';
import QS from 'qs';

if (process.env.isMiniprogram) {
  axios.defaults.adapter = mpAdapter;
}

// 请求拦截
axios.interceptors.request.use(
  config => {
    // config.headers = {
    //   'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8',
    // }
    return config;
  },
  error => Promise.error(error)
);

// 响应拦截
axios.interceptors.response.use(
  response => {
    let { data } = response;
    if (response.status === 200) {
      if (data && data.code != '0') {
        // data.code非0则接口异常
        return Promise.reject(data);
      }
      return Promise.resolve(data.resultData);
    } else {
      return Promise.reject(data);
    }
  },
  // 服务器状态码不是2开头的的情况
  // 这里可以跟后台开发人员协商好统一的错误状态码
  // 然后根据返回的状态码进行一些操作，例如登录过期提示，错误提示等等
  // 下面列举几个常见的操作，其他需求可自行扩展
  error => {
    const { response } = error;
    if (response) {
      switch (response.status) {
        // 401: 未登录
        case 401:
          // 未登录则跳转登录页面，并携带当前页面的路径
          // 在登录成功后返回当前页面，这一步需要在登录页操作。
          break;
        // 403 token过期
        case 403:
          // 登录过期对用户进行提示toast或者dialog
          // 清除本地token和清空vuex中token对象
          // 跳转登录页面
          break;

        // 404请求不存在
        case 404:
          // 对用户进行提示toast或者dialog
          break;
        // 其他错误，直接抛出错误提示
        default:
        // 其他对应处理或者直接对用户进行提示toast或者dialog
      }
      return Promise.reject(response.data);
    } else {
      // 这里处理断网的情况 进行提示
    }
  }
);

/**
 * get方法，对应get请求
 * @param {String} url [请求的url地址]
 * @param {Object} params [请求时携带的参数]
 */
export function get(url, params) {
  return new Promise((resolve, reject) => {
    axios
      .get(url, {
        params: params
      })
      .then(res => {
        resolve(res);
      })
      .catch(err => {
        reject(err);
      });
  });
}

/**
 * post方法，对应post请求
 * @param {String} url [请求的url地址]
 * @param {Object} params [请求时携带的参数]
 */
export function post(url, params) {
  return new Promise((resolve, reject) => {
    axios
      .post(url, QS.stringify(params))
      .then(res => {
        resolve(res);
      })
      .catch(err => {
        reject(err);
      });
  });
}
```

```javascript
// src/request/api.js
const url = {
  test: '/api/forward?apiType=micro_service'
};
console.log(process.env.NODE_ENV);
if (process.env.NODE_ENV === 'production' || process.env.isMiniprogram) {
  Object.keys(url).forEach(item => {
    url[item] = `https://xxxx.com${url[item]}`;
  });
}

export default url;
```

在 main.js 里绑定为 vue 的全局 api
![](https://user-gold-cdn.xitu.io/2020/4/22/171a2538a485e5c0?w=760&h=664&f=png&s=80818)

在业务中调用：

```javascript
import url from '../../request/api';
```

```javascript
this.$get(url.test, {})
  .then(res => {
    this.$api.showToast({
      title: '请求成功'
    });
    console.log('--------requestTest----------');
    console.log('url.test:' + url.test);
    console.log(res);
  })
  .catch(() => console.log('promise catch err'));
```

## 三 屏幕适配

kbone 的官网上介绍它没有用小程序的 rpx 进行屏幕适配，取而代之的是使用更为传统的 rem 进行开发。

理论上支持 vw 没问题。这里采用 postcss-px-to-viewport 自动将 px 转换为 vw，进行屏幕适配。

首先

```
npm install postcss-px-to-viewport --save-dev
```

或者使用 yarn 安装

```
yarn add postcss-px-to-viewport -D
```

在工程目录下建 postcss.config.js

postcss-loader 默认是会从项目根目录获取这个配置项。但是有优先级，webpack 里配置的 options 优先级是高于独立配置文件的。

而 kbone 的几个构建方式有差异，option 在每个 webpack 里不完全相同，无法公用同一个 postcss.config.js.所以我们在 postcss.config.js 里只存放公用的插件配置，引入 webpack 的几个配置文件中。

需要注意的是，postcss-loader 的 options 里配置 plugins 时只接受 array 或者以及 function，不接受 object（与自动加载项目根路径的 postcss.config.js 接受 object 的常见写法不同）。

kbone 的配置里已经是数组形式，所以我们也继续写为数组形式，以匹配原来的配置。

具体代码如下：

```javascript
// ./postcss.config.js
const pxtovw = require('postcss-px-to-viewport');
module.exports = {
  plugins: [
    new pxtovw({
      unitToConvert: 'px', //需要转换的单位，默认为"px"；
      viewportWidth: 375, //设计稿的视口宽度
      unitPrecision: 3, //单位转换后保留的小数位数
      propList: ['*'], //要进行转换的属性列表,*表示匹配所有,!表示不转换
      viewportUnit: 'vw', //转换后的视口单位
      fontViewportUnit: 'vw', //转换后字体使用的视口单位
      selectorBlackList: ['.ignore', '.hairlines'], //不进行转换的css选择器，继续使用原有单位
      minPixelValue: 1, //设置最小的转换数值
      mediaQuery: false, //设置媒体查询里的单位是否需要转换单位
      replace: true, //是否直接更换属性值，而不添加备用属性
      exclude: [/node_modules/] //忽略某些文件夹下的文件
    })
  ]
};
```

以 webpack.dev.config.js 为例：

```javascript
// ./build/webpack.dev.config.js
const postcssConfig = require('../postcss.config');
```

```javascript
...

module: {
    rules: [{
      test: /\.(less|css)$/,
      use: [{
        loader: 'vue-style-loader',
      }, {
        loader: 'css-loader',
      }, {
        loader: 'postcss-loader',
        options: {
          plugins: [
            autoprefixer, postcssConfig.plugins[0]
          ],
        }
      }, {
        loader: 'less-loader',
      }],
    }],
  },

...

```

reference：

[kbone 使用 rem](https://wechat-miniprogram.github.io/kbone/docs/guide/advanced.html#%E4%BD%BF%E7%94%A8-rem)

[vue-cli 中使用 postcss-px-to-viewport](https://blog.csdn.net/Charissa2017/article/details/105420971/)

[webpack4 配置之 postcss-loader](https://blog.csdn.net/kai_vin/article/details/89109272)

[postcss 配置文件优先级问题](https://www.cnblogs.com/Brose/p/12598862.html)

[个人 blog](https://susy123.github.io/)
