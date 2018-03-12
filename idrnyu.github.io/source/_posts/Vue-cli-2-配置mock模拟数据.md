---
title: Vue-cli 2.* 配置mock模拟数据
date: 2018-03-12 21:14:19
tags: [Web前端,Vue,mock模拟请求数据]
categories: [Vue,Web]
---

<img src="/Dom/imgs/2018_03_12/m4.png" width="70%"/>

## mockjs介绍：
>你是否遇见过，前端开发过程中需要数据测试，但后端却迟迟没给你，没感情了分手吧。
是否遇见过需要亲自收集各种各样的数据。
那mock.js便可以很好的帮你解决问题。有了它前端就可以事先模拟数据，前提是和后端约定好了数据接口，怎样的数据。使用mock就可以生成你要的数据了，从而实现开发时前后端分离。
<!-- more -->
## mockjs主要功能：
>- 基于数据模板生成模拟数据。
- 基于HTML模板生成模拟数据。
- 拦截并模拟 ajax 请求。

## mockjs简单使用栗子：
` 1.引入mockjs，引入jquery（此处用jq封装好的ajax发送请求）`
```javascript
    <script type="text/javascript" src="jquery-3.0.0.min.js"></script>
    <script src="http://mockjs.com/dist/mock.js"></script>
```
`2.使用mock生成数据模板`
```javascript
    //这里的第一个参数http://api.cn 就是下面ajax请求的url，mock对该url进行拦截'
    //这里的第二个参数就是template数据模板，mock会返回模板生成的数据
    Mock.mock('http://api.cn', {
        'name': '@name',
        'age|1-100': 100,
        'city': '@city'
    });
```
`3.ajax发送请求`
```javascript
    //url需要和上面的mock的url相同
    $.ajax({
        url: 'http://api.cn',
        dataType: 'json'
    }).done(function(data, status, xhr) {
        alert(
            JSON.stringify(data, null, 4)
        )
    });
```
`4.效果：`
```javascript
键值对
```

## 那么Vue-cli 2.*怎么配置这个东西呢
>在使用vue开发过程中，难免需要去本地数据地址进行请求，而原版配置在`dev-server.js`中，新版`vue-webpack-template`已经删除`dev-server.js`，改用`webpack.dev.conf.js`代替，所以 配置本地访问在`webpack.dev.conf.js`里配置即可。

### 配置 webpack.dev.conf.js 文件
    该文件在cli构建出的项目的 build 目录下。
    在 const portfinder = require('portfinder') 添加内容
    ```javascript
        const express = require('express')
        const app = express()
        const appData = require('../data/data.json')    // JSON数据是假数据
        const ratings = appData
        const apiRouter = express.Router()
        app.use('/api',apiRouter)
    ```
>如图：
<img src="/Dom/imgs/2018_03_12/m1.png" width="100%"/>

    再在 webpack.dev.conf.js 文件下的 devServer 对象里面添加路由
    ```javascript
        before(app) {
            app.post('/api/post/ratings',(req, res) => {
                // res.send(ratings.seller);
                res.json({
                errno: 0,
                data: ratings.seller
                });
                console.log(res);
            }),
            app.get('/api/ratings',(req, res) => {
                res.json({
                errno: 0,
                data: ratings.goods
                });
                console.log(req.query);
            })
        },
    ```
>如图：
<img src="/Dom/imgs/2018_03_12/m2.png" width="100%"/>
    在自己写的代码中加入ajax请求，请求你想要的接口  这样就可以了
>如图1:
<img src="/Dom/imgs/2018_03_12/m3.png" width="100%"/>
>如图2:
<img src="/Dom/imgs/2018_03_12/m4.png" width="100%"/>
>如图3:
<img src="/Dom/imgs/2018_03_12/m5.png" width="100%"/>
>如图4:
<img src="/Dom/imgs/2018_03_12/m6.png" width="100%"/>
>如图5:
<img src="/Dom/imgs/2018_03_12/m7.png" width="100%"/>

