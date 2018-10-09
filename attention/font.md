##### 小程序同网页一样可以使用字体，但网页中使用src:url()引入的字体无论是本地还是远程资源都不行。
##### 解决：将ttf转换成base64，则可以正常使用。
##### [字体在线转换](https://transfonter.org/)，添加字体并开启Base64 encode，点击Convert按钮开始转换。