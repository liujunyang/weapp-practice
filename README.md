# 微信小程序开发实践

标签（空格分隔）： 微信 小程序 坑 实践

---
[TOC]

## 项目是否适合移植到小程序上？
小程序由于微信提供了一些组件，在微信中的一些体验确实不错，对于开发来说，由数据驱动的开发模式也是挺爽的。

## 概要介绍
其实就是类似于VUE REACT的 MVVM模式，专注于数据和逻辑。
小程序开发框架的目标是通过尽可能简单、高效的方式让开发者可以在微信中开发具有原生 APP 体验的服务。

框架提供了自己的视图层描述语言 WXML 和 WXSS，以及基于 JavaScript 的逻辑层框架，并在视图层与逻辑层间提供了数据传输和事件系统，可以让开发者可以方便的聚焦于数据与逻辑上。

[官网：微信小程序开发教程](https://mp.weixin.qq.com/debug/wxadoc/dev/)
[官方：微信小程序联盟](http://www.wxapp-union.com/)
[视频：快速入门](https://ke.qq.com/course/182359)

模板层级
请求包装
用户会话处理

## 实践得到的经验
### 规则
1.目前打包后的文件不能超过2M，否则不能上传到微信服务器。

### 小程序不支持的
1.不支持sass语法
2.不支持window、document，不能使用相关的库，如jquery、PreventMoveOverScroll。

3.不支持直接使用svg标签开发。image的src放远程svg可以，background-image里也可以。
但是可以使用的canvas标签*（canvas坑：position absolute的层盖不住canvas）*，可以使用的库：[wx-charts](https://github.com/xiaolin3303/wx-charts) *(有坑，放在app.js中然后在page中引用的话，找不到ringChart上面的函数，如ringChart.addEventListener。要直接在page中引库。)*。

4.不支持阻止默认事件，没有`preventDefault`。
5.没有`br`标签。
6.不支持`table`表格。
7.不能使用`&nbsp;`来增大文字间距。
8.小程序的`xxx.wxss`文件font-face的url不接受http地址作为参数,可以接受 ***base64*** ,因此可以先将字体文件下载后,转换为base64，然后再引用。[链接](https://zhuanlan.zhihu.com/p/24697235)。


### 新特性
1.img标签换成了image标签`<image src="http://sfe.ykt.io/o_1bbd2f7j02583ii2rg1p441gvo9.jpg"></image>`。
2.`text`标签认\n换行，不能包裹`<br/>`标签，会直接输出出来，有点类似`textarea`。
3.小程序中 :nth-child(n) 是从0开始的。
4.`switch`标签。但是不能改变大小样式，像老`radio`标签一样讨厌。
5.`picker`标签。但是在开发者工具中选项不会循环，在安卓手机预览中选项会循环。

6.`scroll-view`标签。有滚动条的盒子。要想在进入页面的时候自动滚动到某处，可以使用scroll-view的scroll-into-view属性，不过一定要在scroll-view存在后设置才会生效，尤其是通过template引用的时候，比如同时通过setData设置使用该template和scroll-into-view的值，并不会使滚动生效。

7.关于屏幕下拉露底：Android不会，iPhone会。可以通过配置解决：disableScroll	Boolean	false	设置为 true 则页面整体不能上下滚动；只在page.json中有效，无法在app.json中设置该项。[链接](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/config.html)。

8.[`template`标签](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/template.html)
模板的作用域：模板拥有自己的作用域，只能使用data传入的数据。

9.在功能按钮上使用`catchtap`防止冒泡

10.`hidden`参数，控制蒙版的利器


### 小窍门
1.关于下拉刷新
要在page.json中设置 `enablePullDownRefresh: true`
`onPullDownRefresh`内要手动`stopPullDownRefresh`，否则一直在点点点（loading）。

2.page的onload函数中有参数options可以获取路径的query

3.小程序的支付和微信的支付不是一个appid，[需要后端新开发下单接口](http://www.wxapp-union.com/article-782-1.html)

4.wxml最好在开发者工具编辑（有提示）。js， wxss可以在熟悉的编辑器编辑。

5.小程序中如果赋予的新值是undefined的话，根本不会进行赋值，也不会覆盖之前的值。如：
```
app.setData({
    isPaperCollected: finishedQuizList['id'+quizID] || false
})
```

6.通过多次使用Object.assign({}) 解决data也模块化后data不能深层复制的问题。
Object.assign不是深层复制。

7.在微信web开发者工具中一定要在动作->设置->勾上“不使用任何代理，勾选后直连网络”，否则老是报“
Failed to load resource: net::ERR_NAME_NOT_RESOLVED”的bug：[链接](http://www.wxapp-union.com/forum.php?mod=viewthread&tid=1450&highlight=ERR%5C_NAME%5C_NOT%5C_RESOLVED)

8.每一个小程序页面也可以使用.json文件来对本页面的窗口表现进行配置。 页面的配置比app.json全局配置简单得多，只是设置app.json中的window配置项的内容，页面中配置项会覆盖app.json的window中相同的配置项。页面的.json只能设置window相关的配置项，以决定本页面的窗口表现，所以无需写 `window` 这个键。[链接](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/config.html)。

9.[data](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html)是在page中设置的，不是在app.js中的。不涉及渲染的可以不要放data里面。

10.[Page.prototype.setData()](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/app-service/page.html)变更数据同时更新页面数据。
setData 函数用于将数据从逻辑层发送到视图层，同时改变对应的 this.data 的值。直接修改 this.data 无效，无法改变页面的状态，还会造成数据不一致。单次设置的数据不能超过1024kB，请尽量避免一次设置过多的数据。

11.[wx:if](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/conditional.html) 是惰性的，如果在初始渲染条件为`false`，框架什么也不做，在条件第一次变成真的时候才开始局部渲染。

12.pageScrollTo 的问题:页面滚动后又滚动回顶部。
我也遇到类似的问题，解决了。我的需求是蒙层上有一些按钮，点击某个按钮后关闭蒙层，然后页面滚动到相应位置。
我的解决方法是，关闭蒙层后，setTimeout 延迟滚动。

### 会话管理
微信的网络请求接口 wx.request() 没有携带 Cookies，这让传统基于 Cookies 实现的会话管理不再适用。为了让处理微信小程序的服务能够识别会话，我们会话管理使用[weapp-session-client](https://github.com/CFETeam/weapp-session-client)。这需要服务端的支持。基本原理是包装wx.request并在 Header 上使用特殊的字段跟踪。

使用时遇到的问题：
1.微信开发者工具报错：`Uncaught ReferenceError: regeneratorRuntime is not defined`
原因是 Generator 函数不被支持。
[解决方法：](http://www.wxapp-union.com/thread-4278-1-1.html)
* 微信开发者工具关闭ES6转ES5
* 真正解决办法，提供regeneratorRuntime [链接1](https://mp.weixin.qq.com/s?__biz=MzI0ODU5Mzg0NA==&mid=2247483696&idx=1&sn=b900291668b17b9755af2bcc7025ebbb&chksm=e99f2debdee8a4fdd38107427c4494e8e115cfa2fc48f11a85b571baf1ed1ccbdaa936f2a647#rd) -- [链接2](https://github.com/guyoung/GyWxappCases)

2.题外话：善用 Promise
本项目后端处理会话管理时要求发送请求时不能使用相同的 X-WX-Code发送多次全部header数据，RawData、Signature等，否则报错。
所以使用weapp-session-client登录的时候要等login返回之后再发送其他的请求，
小程序加载后立即顺序执行app.js和page.js里面的配置，但是wx request是异步的，所以有可能page中的请求开始执行时app.js中的login还没结束，就会导致bug。
可以使用个promise解决掉。

### 进阶
1.[扩展微信小程序框架功能](https://segmentfault.com/a/1190000008068632)

2.疑问：微信切换账号会不会销毁小程序

3.检查TLS版本的问题
http://www.dongcoder.com/detail-410653.html
解决方法：微信开发者工具勾选开发时不检查检查TLS版本或后端配置

4.蓝牙、震动的调用 
