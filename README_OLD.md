# ![logo](http://m.ireadercity.com/webapp/img/logo.png) 书香云集js与客户端交互文档

## 设计概要
#### 设计目的
> 为了能够使JS与 **书香云集** 客户端的交互更加顺利，重新设计了交互机制。
> 之前交互机制的不足之处：
> * 采用自定义的schema方式只能由js调起客户端的相应功能。
> * window.location = "xxx://xxx" 这种形式容易阻断部分js功能的执行。
> * 缺少操作的回调机制。比如何时分享成功和分享失败。
> 
> 新的交互机制加入了回调的支持，加入了生命周期概念，目的在于可以对整个交互过程进行操作把控。

> Android 4.2 以下的系统有严重安全隐患。可以使用 [Safe Java-JS WebView Bridge](https://github.com/pedant/safe-java-js-webview-bridge)

#### URL约定
> 新的交互机制约定了一套URL规则，用于控制页面加载的方式，例如，是否需要全屏加载，是否允许关闭等。具体形式如：
> `http://www.sxyj.net/book_read/bookid_e809304b4c434b9fbe00a75eb2f7e31c.html?hostsdk=fullscreen,uncloseable`
>
>| 参数名 | 默认生效 | 备注 |
| --- | --- | --- |
| fullscreen | 否 | 全屏加载页面。没有标题栏，没有底部工具条 |
| uncloseable | 否 | 不允许关闭页面（由页面控制关闭） |
| unshareable | 否 | 不显示分享 |
| openpush | 否 | push 方式打开窗口（ios） |
| openpresent | 否 | present 方式打开窗口（ios） |

#### 链接处理
> 客户端应该支持使用target指定的打开方式，如果 `target` 为 `_blank` 则在新窗口打开页面。否则在当前窗口加载新页面
```html
<a href="http://www.baidu.com" target="_blank">链接</a>
```

#### 安全机制
> 为了保证客户端的安全，防止打开第三方不可信任的地址。造成恶意读取用户信息的安全隐患。设计了安全机制。
> 在适当的时候，客户端向服务端请求一个可信任网址列表（客户端自行考虑）。也可以在访问页面之前，向服务端特定的接口验证是否是可信任地址。如果是，则将 `userId` 等敏感参数通过 `onInit` 传递给网页。
> 可信任地址允许使用简单通配符 `*` 例如 `*.ireadercity.com*`

## 客户端回调
事件回调是App客户端主动调用，用于告知/通知 js,以便于js可以在适当时候做特殊处理。分为 **生命周期回调** 和 **操作回调**

#### 生命周期回调 
生命周期回调由客户端主动发起，是有序的调用，并且一定会调用。自动触发。

| 回调方法名信息 | 备注 |
| --- | --- | --- |
| window.host_sdk.onInit() | 客户端 **加载网页完成** 时调用 |
| window.host_sdk.onPause() | 客户端网页 **不可见**、**被遮挡**、**不可操作** 时调用 |
| window.host_sdk.onResume() | 客户端网页 **恢复可见**、**恢复可操作** 时调用 |
| window.host_sdk.onStop() | 客户端 **网页销毁** 时调用 |

#### 操作回调
操作回调由网页发起某些操作时，客户端 **完成操作** 过程后，

| 回调方法名信息 | 备注 | 参数 |
| --- | --- | --- | --- |
| host_sdk.errorCallback(**msg**) | 网页发起操作后，客户端 **执行操作过程产生错误** 时调用 | 错误信息 |
| host_sdk.shareCancelCallback() | 网页发起分享操作后，用户 **放弃分享** 时调用 | 无 |
| host_sdk.shareSuccessCallback(**platform**) | 网页发起分享操作后，客户端 **执行分享成功** 时调用 | 分享成功的平台 |
| host_sdk.getInfoCallback(**infoJson**) | 网页发起获取环境信息操作后，客户端 **提取信息并通知网页** 时调用 | 包含环境信息的json字符串 |
| host_sdk.downloadBookSuccessCallback(**bookId**) | 网页发起下载书籍操作后，客户端 **完成一本书的下载操作** 时调用 | 下载成功的书籍id |
| host_sdk.downloadBookErrorCallback(bookId) | 网页发起下载书籍操作后，客户端 **下载某本书失败** 时调用 | 下载失败的书籍id |
| host_sdk.rechargeSuccessCallback(orderId) | 网页发起充值操作后，客户端 **充值成功** 时调用 | 充值成功的订单号 |
| host_sdk.rechargeErrorCallback(orderId) | 网页发起充值操作后，客户端 **充值失败**、**取消充值** 时调用 | 如果有订单号则是订单号，如果没有则为空 |
| host_sdk.loginSuccessCallback(**userId**) | 网页发起登录操作后，客户端 **执行登录** 时调用 | 用户Id |
| host_sdk.loginCancelCallback() | 网页发起登录操作后，客户端 **用户取消登录** 时调用 | 无 |
| host_sdk.getVipSuccessCallback() | 网页发起开通vip操作后，客户端 **开通VIP成功** 时调用 | 无 |
| host_sdk.getVipCancelCallback() | 网页发起开通vip操作后，客户端 **用户取消开通** 时调用 | 无 |

## 网页发起的操作
由于Android设备和苹果设备的差异，因此网页发起操作的方式也不同。
>### Android
`android_hostsdk.操作名(参数...);`
###### 以分享为例
```javascript
android_hostsdk.share(
	'标题',
	'http://www.baidu.com',
	'描述',
	'http://m.ireadercity.com/webapp/img/logo.png',
	'qzone,wechat'
);
```

>### IOS
`ios_hostsdk.callHandler("操作名",参数);`
###### 以分享为例
```javascript
ios_hostsdk.callHandler(
	"share",
	{
		title: "分享测试标题",
		description: "书香云集，您的掌上图书馆",
		url: "http://www.ireadercity.com",
		icon: "http://m.ireadercity.com/webapp/img/logo.png",
		platforms: "wechat,weibo,wechatcircle"
	}
);
```

#### 操作一览
| 操作 | 备注 |
| --- | --- |
| share | 分享 |
|getInfo|获取环境信息（userId，deviceId，idfa，version，channel...）|
|openBook|打开并阅读一本书|
|openBookList|打开指定的书单|
|searchBook|直接搜索书籍，显示搜索结果|
|downloadBook|下载指定的书籍|
|showBookDetail|显示书籍详情|
|recharge|打开充值界面|
|login|打开登录界面|
|exit|关闭当前页面|
|setCloseable|设置是否可以关闭窗口（可以让js决定何时关闭窗口）|
|getVip|开通vip|
|openUserCategory|打开用户个人书坊配置|

### share 分享
> 分享形式根据参数判断。比如，icon为空的情况下，分享文字内容。有description和icon的情况下，就是图文内容。
>##### 参数选项
| 参数名 | 类型 | 备注 |
|---	|---|---|
| title | String | 分享的标题 |
| url | String |分享的链接 |
| description | String | 分享描述 |
| icon | String | 分享的图片 |
| platforms | String | 要分享的平台，多个用逗号分割:qzone,qq,wechat,wechatcircle,weibo |
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback(platform)`、`host_sdk.cancelCallback()`
> > 分享成功后回调，携带用户选择的平台名

### getInfo 获取环境信息
> 获取信息，包括userId，deviceId，idfa，version，channel等等
>##### 参数选项
无
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback(infoJson)`
> > 获取成功后回调时，传递包含信息的json字符串

### openBook 打开并阅读一本书
>##### 参数选项
| 参数名 | 类型 | 备注 |
|---	|---|---|
| bookId | String | 书籍Id |
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback()`

### openBookList 打开指定的书单
>##### 参数选项
| 参数名 | 类型 | 备注 |
|---	|---|---|
| bookListId | String | 书单Id |
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback()`

### searchBook 直接搜索书籍，显示搜索结果
>##### 参数选项
| 参数名 | 类型 | 备注 |
|---	|---|---|
| keyword | String | 关键字 |
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback()`

### downloadBook 下载指定的书籍
>##### 参数选项
| 参数名 | 类型 | 备注 |
|---	|---|---|
| bookId | String | 书籍id，多个逗号分割 |
##### 触发的回调
`host_sdk.errorCallback(bookId)`、`host_sdk.successCallback(bookId)`
> > 每本书下载失败回调时，携带下载失败的书籍id；每次下载失败回调时，携带书籍id

### showBookDetail 显示书籍详情
>##### 参数选项
| 参数名 | 类型 | 备注 |
|---	|---|---|
| bookId | String | 书单Id |
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback()`

### recharge 打开充值界面
>##### 参数选项
无
##### 触发的回调
`host_sdk.errorCallback(orderId)`、`host_sdk.successCallback(orderId)`、`host_sdk.cancelCallback(orderId)`
> > 充成功后，携带订单号参数、失败或者取消时如果有订单号，也携带上

### login 打开登录界面
>##### 参数选项
无
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback()`、`host_sdk.cancelCallback()`

### exit 关闭当前页面
>##### 参数选项
无
##### 触发的回调
`host_sdk.errorCallback(msg)`

### setCloseable 设置是否可以关闭窗口
>##### 参数选项
| 参数名 | 类型 | 备注 |
|---	|---|---|
| flag | boolean | 是否可以关闭 |
##### 触发的回调
`host_sdk.errorCallback(msg)`

### getVip 开通VIP
>##### 参数选项
无
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback()`、`host_sdk.cancelCallback()`

### openUserCategory 打开用户个人书坊配置
>##### 参数选项
无
##### 触发的回调
`host_sdk.errorCallback(msg)`、`host_sdk.successCallback()`、`host_sdk.cancelCallback()`

## JavaScript SDK
>#### 由于Android和ios平台的差异，需要根据不同平台使用不同的调用方式。过程过于繁琐，影响前端开发效率。因此对本文档实现的交互功能做了更加易于使用的封装。
#### 文档查看：[HostSdk.md](HostSdk.md)
#### JS下载：[HostSdk.js](HostSdk.js)