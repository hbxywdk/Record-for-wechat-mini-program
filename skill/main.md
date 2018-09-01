1、小程序调用子组件方法
```
<countdown id="countdown"></countdown >
onReady() {
    this.countdown = this.selectComponent('#countdown')
    this.countdown.start()
}
``` 

2、直接改动上一个页面的data。
```
const pages = getCurrentPages(); // 当前页面
const prevPage = pages[pages.length - 2]; // 上一页
prevPage.setData({ // 直接给上一页面赋值
  selectCity: info,
});
```  

3、需求：微信群只显示首字母  
分享到群的信息只能拿到群ID
<open-data type="groupName" open-gid="{{openGId}}" />获取到的又是完整群名
只显示首字母有两种方法：
- 足够大的字体与足够大的字间距，只保留第一个字，将其余的字挤下去，父元素溢出隐藏。
该方法只支持纯中文或纯英文，中英混杂会出现一个中文和一个英文同时出现的情况。
```
<view class='big-qun-name'>
  <open-data type="groupName" open-gid="{{ item.open_gid }}"></open-data>
</view>
```
```
.test open-data{
   width: 100rpx;
   height: 100rpx;
   line-height: 100rpx;
   font-size: 51rpx;
   letter-spacing:1px;
   display: block;
   color: #fff;
}
```
- 设置字体为0，使用css3给首字母设置字体大小。
该方法使用起来时好时坏，不太建议使用。
```
<view class="nc-img__text" wx:else>
  <open-data openGid="{{groupid}}" type="groupName" ></open-data>
</view>
```
```
.text {
    width: 50px;
    height: 50px;
    border-radius: 50px;
    background-color: #b8b8b8;
    color: #fff;
    font-size: 60rpx;
    display: flex;
    justify-content: center;
    align-items: center;
    position: absolute;
    left: -1px;
    top: -1px;
}

.text open-data {
    width: 50px;
    height: 50px;
    line-height: 48px;
    display: block;
    text-align: center;
    font-size: 0;
}

.text open-data::first-letter {
    font-size: 60rpx;
}
```
###### 可以考虑使用群成员的头像来表示该群。  

4、去除小程序<button>的默认样式。

```
button.remove-btn-style{
  outline:none;
  border:none;
  list-style: none;
  border-radius:0;
}
button.remove-btn-style:after{
  outline:none;
  border:none;
  list-style: none;
}
```

5、 直接退出小程序
虽然官方没有给出关闭小程序的方法，但其实还是有办法关闭的。
- 如果是在首页
```
onLoad: function () {
    wx.navigateBack({
        delta: -1
    })
}
```