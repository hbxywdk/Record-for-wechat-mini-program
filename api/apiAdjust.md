#### 1、获取用户信息机制修改。
由于微信修改了getUserInfo机制，所以不要在进入小程序调用`wx.login`之后立刻调用`wx.getuerinfo`，一方面用户很大概率会点击拒绝授权导致小程序功能无法使用，另一方面由于wx.login已经可以用于用户登录，我们应在需要的时候再调用，详细调整点击[这里](https://developers.weixin.qq.com/blogdetail?action=get_post_info&lang=zh_CN&token=&docid=000c2424654c40bd9c960e71e5b009)。

#### 2、分享机制修改。
微信本次修改了分享机制，之后开发者拿不到任何分享后的回调（成功、失败、完成）
- 对于想要获取分享结果的今后将`无法实现`;
- 对于想要获取分享信息的，可在分享后点击群内分享消息卡片，在App的onLaunch与onShow(建议在onShow中获取，onLaunch在小程序整个生命周期中只会执行一次)中可获取shareTicket ，用来换取群唯一标识 openGId ，进而显示对应群的相关信息的小程序。

详见[这里](https://developers.weixin.qq.com/blogdetail?action=get_post_info&lang=zh_CN&token=&docid=0006823675c0e82a8307c6db25bc09)。

#### 3、微信小程序对于iOS端虚拟支付的调整
- 什么是虚拟支付？
  - 虚拟支付是指购买非实物商品。比如：VIP会员、充值、课程、音视频等虚拟产品。
  
之后iOS端的虚拟支付，将不再被允许，新提交的代码中若含有虚拟支付代码，`将无法通过审核`
详见[这里](https://developers.weixin.qq.com/community/develop/doc/000464b5b3cb382b9d372b98f5ac08)。

#### 4、对于wx.openSetting的调整  
- 早先wx.openSetting开发者是可以无限调用的，导致API被滥用，后调整为只允许button方式调用，由用户主动触发打开setting界面:
```
<button open-type="openSetting">打开授权设置页</button>
```
##### 2018年9月12日该API再次调整为以下几种方式打开：
- 方法1：使用 button 组件来使用此功能：
```
<button open-type="openSetting" bindopensetting="callback">打开设置页</button>
```
- 方法2:由点击行为触发wx.openSetting接口的调用：
```
<button bindtap="openSetting">打开设置页</button> 
// js 
openSetting() {  wx.openSetting()}
```
方法2已在最新版开发者工具中支持（基础库切到2.2.4及以上）

实测还存在第三种打开的方法
- 方法3：使用wx.showModal，在sucess回调中打开setting页
```
wx.showModal({
    title: '授权',
    content: '是否授权',
    success: res => {
        if (res.confirm) {
            wx.openSetting({})
        } else {
            console.log('取消')
        }
    }
})
```
注意：方法3，若在封装中使用了promise则无法调起setting页面，将所有的promise方式改成callback回调的方式就能正常的调起（promise是异步的，“点击行为允许调用”这个机制要求是同步的”）。 

#### 5、微信调整小程序跳转小程序功能（2018-11-14）
> 使用跳转其他小程序功能，则需要在代码配置中声明将要跳转的小程序名单，限定不超过10个。该名单可在发布新版时更新，不支持动态修改。
具体配置为在app.json中添加navigateToMiniProgramAppIdList（String、Array）一项：
```
{
  "pages": ["pages/index/index", "pages/logs/index"],
  "window": {
    "navigationBarTitleText": "Demo"
  },
  "tabBar": {
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页"
      },
      {
        "pagePath": "pages/logs/logs",
        "text": "日志"
      }
    ]
  },
  "networkTimeout": {
    "request": 10000,
    "downloadFile": 10000
  },
  "debug": true,
  "navigateToMiniProgramAppIdList": ["wxe5f52902cf4de896", ..., ...]
}
```
该调整导致所有游戏盒子类小程序凉凉。
详见[这里](https://mp.weixin.qq.com/cgi-bin/announce?action=getannouncement&announce_id=11541056526eufNY&version=&lang=zh_CN)。

##### 6、对于自定义导航栏样式的支持。
详见[这里](https://developers.weixin.qq.com/community/develop/doc/000408d3d34178b682d7bdee259401)。

##### 7、获取用户位置信息时需填写用途说明（2018-12-26）
> 根据 iOS 系统对用户隐私保护的要求，同时我们也为了让用户可以更好的判断是否要将地理位置信息提供给开发者，故调整为当小程序/小游戏获取用户地理位置信息时，开发者需要填写获取用户地理位置的用途说明。
具体配置：

在 app.json 里面增加 permission 属性配置（小游戏需在game.json中配置）
```
"permission": {
    "scope.userLocation": {
        "desc": "位置信息用途说明"
    }
}
```
##### 但目前该方法并不支持多语言。