#  微信小程序之wafer2-node方案

开始先简述建立小程序和使用腾讯云wafer2的基本步骤

1.在[微信公众平台](https://mp.weixin.qq.com/)注册开公众平台开发者帐号（这里有几点注意事项，此帐号和微信开放平台不同，且注册的邮箱必须没有在微信其他平台下注册过的，一个账号只能开发一个小程序，要再开发新的只能再申请一个，但是可以在帐号内绑定多个不同的开发者，不过小程序拥有者帐号留有最高权限，一些重要关键的操作需要权限验证）

2.注册后在帐号内新建小程序并完善信息，在开发设置里获得AppID ，并在开发者工具里点击腾讯云，按照指引开通腾讯云的wafer2开发环境，这部分隶属腾讯云小程序解决方案，相关文档指引里面的问答已经比较详细了

![img](https://pic2.zhimg.com/v2-29bbc7c09744a0717f8d8f8bd2378a85_b.jpg)

3.wafer2解决方案，分开发环境和生产环境，不同的就是工程内的相关参数不同，开发环境可以直接在微信[微信开发者工具](https://mp.weixin.qq.com/debug/wxadoc/dev/devtools/devtools.html)内直接上传部署起作用，而生产环境则需要上传代码，然后到腾讯云小程序解决方案处进行手动部署。

4.[微信开发者工具](https://mp.weixin.qq.com/debug/wxadoc/dev/devtools/devtools.html)的wafer2-node相关使用方法请移步[开通及使用指引](https://cloud.tencent.com/document/product/619/11447)再仔细浏览一遍。

首先是wafer2的[GitHub页面](https://github.com/tencentyun/wafer2-quickstart)，相比于wafer1，wafer2目前只提供php和node的方案，在这里建议两种初始化项目的方式，第一种就是[开通及使用指引](https://cloud.tencent.com/document/product/619/11447)里描述的在微信开发者工具里使用建立腾讯云node.js启动模板，第二种是下载[GitHub的node.js项目模板](https://github.com/tencentyun/wafer2-quickstart-nodejs)，从这里开始建立自己的项目，不建议从零开始构建项目。

为什么使用wafer2-node？

首先是因为，小程序的实现中缺少cookie和session的部分，所以在会话的实现并没有网页端实现方便，通过粗略的浏览源码，该解决方案通过[客户端SDK](https://github.com/tencentyun/wafer-client-sdk)和[服务端SDK](https://github.com/tencentyun/wafer2-node-sdk)配合，将session以小程序端的localstorage存储实现，自建tunnel服务模拟websocket的实现，整个方案其实也就关注这两部分的功能，客服消息和文件上传功能不是主要，方案内的数据库mysql的连接采用的是[knex.js](http://knexjs.org/)实现，并没有上升到orm，数据库的管理可以在腾讯云小程序解决方案控制台内通过phpMyAdmin登陆并进行相关更改和配置，总的来说，使用wafer2-node主要是解决会话的管理和websocket实现的问题。

![img](https://pic4.zhimg.com/v2-a2ed4d9b2bfa88af17400bc857c03382_b.jpg)

sdk内数据库实例的获取？

```
const { mysql } = require('../qcloud.js');
```

qcloud.js是wafer-node-sdk配置完以后的实例

```
const qcloud = require('wafer-node-sdk')
```

从qcloud中获取knex实例，获取knex实例mysql以后，可以如下使用

```
mysql('cAuth').table('cSessionInfo').select('user_info', 'totalscore')
```

相关使用方法请参见[knex.js](http://knexjs.org/)，比较简单

wafer2-node的构成

wafer2-node是基于koa2，内集成了koa-router和koa-bodyparser中间件，熟悉koa2这里有一个[入门教程](https://chenshenhai.github.io/koa2-note/)，简单学习一下，熟悉基本，方便之后使用。

实际上开发过程中可以发现，在koa2前面还有一层nginx做反向代理，但这一层似乎并不让开发者接触到，所以也就忽略了。

下面来做一个基本解析

```
const router = require('koa-router')({
    prefix: '/weapp'
})
const controllers = require('../controllers')
// 从 sdk 中取出中间件
// 这里展示如何使用 Koa 中间件完成登录态的颁发与验证
const { auth: { authorizationMiddleware, validationMiddleware } } = require('../qcloud')
// --- 登录与授权 Demo --- //
// 登录接口
router.get('/login', authorizationMiddleware, controllers.login)
// 用户信息接口（可以用来验证登录态）
router.get('/user', validationMiddleware, controllers.user)
// --- 图片上传 Demo --- //
// 图片上传接口，小程序端可以直接将 url 填入 wx.uploadFile 中
router.post('/upload', controllers.upload)
// --- 信道服务接口 Demo --- //
// GET  用来响应请求信道地址的
router.get('/tunnel', controllers.tunnel.get)
// POST 用来处理信道传递过来的消息
router.post('/tunnel', controllers.tunnel.post)
// --- 客服消息接口 Demo --- //
// GET  用来响应小程序后台配置时发送的验证请求
router.get('/message', controllers.message.get)
// POST 用来处理微信转发过来的客服消息
router.post('/message', controllers.message.post)

module.exports = router
```

以上是demo里的routes.js里的内容，小程序及服务端算比较严格的前后端分离的形式，这里的router主要是为了实现api的功能，从qcloud里获取的authorizationMiddleware和validationMiddleware中间件，简单的说就是用户的登陆注册和后续的请求认证，在/login进行authorizationMiddleware，然后后续只要需要验证用户信息的借口都调用validationMiddleware进行用户信息的验证，识别特定用户。这里用法也很简单，相应的接口地址对应特定的处理的controllers的处理函数，一一对应还算比较清晰。

```
const _ = require('lodash')
const fs = require('fs')
const path = require('path')
/**
 * 映射 d 文件夹下的文件为模块
 */
const mapDir = d => {
    const tree = {}
    // 获得当前文件夹下的所有的文件夹和文件
    const [dirs, files] = _(fs.readdirSync(d)).partition(p => fs.statSync(path.join(d, p)).isDirectory())
    // 映射文件夹
    dirs.forEach(dir => {
        tree[dir] = mapDir(path.join(d, dir))
    })
    // 映射文件
    files.forEach(file => {
        if (path.extname(file) === '.js') {
            tree[path.basename(file, '.js')] = require(path.join(d, file))
        }
    })
    return tree
}
// 默认导出当前文件夹下的映射
module.exports = mapDir(path.join(__dirname))
```

以上是controllers的index.js实现为将文件下的文件映射到controllers上，比如文件夹下的login.js映射为controllers.login，其他的类似，如果想配置自己的接口，有如下一个例子

```
//routes/index.js
router.get('/youapi', validationMiddleware, controllers.youapi);

//controllers/youapi.js
module.exports = async (ctx, next) => {
    if (ctx.state.$wxInfo.loginState === 1) {
        // loginState 为 1，登录态校验成功
        ctx.state.data = ctx.state.$wxInfo.userinfo
    } else {
        ctx.state.code = -1
    }
}
```

只要在youapi.js里写你的接口处理逻辑就行了，ctx.state.$wxInfo是携带的相关的登录信息，里面的内容大家可以自己console出来看，服务端的控制台，使用[微信开发者工具](https://mp.weixin.qq.com/debug/wxadoc/dev/devtools/devtools.html)里腾讯云选下的启动单步调试选项，就可以进行服务端的代码调试了，也很方便。

今天先写这部分，稍后还会继续完善，有任何意见建议，请留言交流，谢谢大家。

这里稍微做个更新，因为近期没有开发小程序相关的东西，所以用户授权和用户信息获取部分好像有变，这里提醒一下，等有时间更新一下。