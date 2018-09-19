1、正式上线接口需要使用https，接口需在后台配置白名单，域名一个月只能修改五次，建议一次性将开发、生产等环境的域名配全，最好上传、下载、请求都配置一样的，避免多次修改带来的麻烦。

2、小程序中使用websocket需要wss://。

3、请求白名单配置有延迟，可能开发工具请求不了手机却可以、等待几分钟后再次尝试即可。

4、必要时可使用内网穿透到本地。

5、上线测试版本需要小程序关联公众号。

6、由于微信修改了getUserInfo机制，所以不要在进入小程序调用`wx.login`之后立刻调用`wx.getuerinfo`，一方面用户很大概率会点击拒绝授权导致小程序功能无法使用，另一方面由于wx.login已经可以用于用户登录，我们应在需要的时候再调用，详细调整点击[这里](https://developers.weixin.qq.com/blogdetail?action=get_post_info&lang=zh_CN&token=&docid=000c2424654c40bd9c960e71e5b009)。

7、微信修改了分享机制，导致开发者拿不到任何分享后的回调（成功、失败、完成）
- 对于想要获取分享结果的今后将`无法实现`;
- 对于想要获取分享信息的，可在分享后点击群内分享消息卡片，在App的onLaunch与onShow(建议在onShow中获取，onLaunch在小程序整个生命周期中只会执行一次)中可获取shareTicket ，用来换取群唯一标识 openGId ，进而显示对应群的相关信息的小程序。

详见[这里](https://developers.weixin.qq.com/blogdetail?action=get_post_info&lang=zh_CN&token=&docid=0006823675c0e82a8307c6db25bc09)。

8、微信小程序对于iOS端虚拟支付的调整
- 什么是虚拟支付？
  - 虚拟支付是指购买非实物商品。比如：VIP会员、充值、课程、音视频等虚拟产品。
  
之后iOS端的虚拟支付，将不再被允许，新提交的代码中若含有虚拟支付代码，`将无法通过审核`
详见[这里](https://developers.weixin.qq.com/community/develop/doc/000464b5b3cb382b9d372b98f5ac08)。
