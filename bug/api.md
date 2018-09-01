1、tab页中使用‘wx.getSystemInfo’来获取屏幕高度，真机获取的高度比开发工具少48，这48为tabBar的高度。  

2、小程序canvas无法直接绘制网络图（开发工具有效，真机无效）  
需要使用 downloadFile API 下载图片后绘制，且图片地址必须配置在后台的‘downloadFile合法域名中’
```
// 无效
const cardBg = 'https://image.shanglishi.com/share-BJ.png'
ctx.drawImage(cardBg, 0, 0, 250, 200) 
ctx.draw()

// 有效
wx.downloadFile({
   url: cardBg ,
   success: function (res) {
     ctx.drawImage(res.tempFilePath, 0, 0, 250, 200)
     ctx.draw()
   }
})
```