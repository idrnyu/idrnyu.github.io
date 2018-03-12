---
title: 使用Cordova打包vue-cli构建的项目为APP配置
date: 2018-03-12 22:35:15
tags: [Web前端,Vue,Cordova]
categories: 
- Vue
- Cordova
---

<img src="/Dom/imgs/2018_03_12/c4.jpg" width="15%"/>
## Cordova 打包 Vue-cli 项目配置
>Vue-cli 搭建出来的基于 `weipack` 的Vue项目，

------------
<!-- more -->
## 配置 index.html 中的 head 中的 meta 标签
```html
<!-- <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *; img-src 'self' data: content:;"> 可选项-->
  <meta name="format-detection" content="telephone=no">
  <meta name="msapplication-tap-highlight" content="no">
  <meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1, width=device-width">
```
## 配置 index.html 中引入Cordova
```javascript
<script type="text/javascript" src="cordova.js"></script>
```
### 如下图 1 配置 index.html 文件:
<img src="/Dom/imgs/2018_03_12/c1.png" width="100%"/>
## 配置 main.js 文件
`在 项目目录下 src/main.js 中插入代码如下`
```javascript
//用于判断运行环境，好用于调试
if ('ontouchstart' in window) {
  document.addEventListener('deviceready', function () {
    router.start(App, '#app'); //启动路由
  }, false);
} else {
  router.start(App, '#app');
}
```
### 如下图 2 配置 main.js 文件:
<img src="/Dom/imgs/2018_03_12/c2.png" width="100%"/>
## 配置 config/index.js 文件
`在 项目目录下 config/index.js 中修改代码如下`
```javascript
// Template for index.html
index: path.resolve(__dirname, '../www/index.html'),

// Paths
assetsRoot: path.resolve(__dirname, '../www'),
assetsSubDirectory: '',
assetsPublicPath: './',
```
### 如下图 3 配置 index.js 文件:
<img src="/Dom/imgs/2018_03_12/c3.png" width="100%"/>

    使用 npm run build 打包后 将生成的 www 文件替换 Cordova 项目中的 www 文件夹

## 效果图1：
<img src="/Dom/imgs/2018_03_12/c4.jpg" width="15%"/>
## 效果图2：
<img src="/Dom/imgs/2018_03_12/c5.jpg" width="50%"/>