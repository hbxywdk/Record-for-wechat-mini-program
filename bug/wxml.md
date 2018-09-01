1、textarea、input等元素层级最高的Bug、由于textarea等使用的是原生组件，它的层级最高，会产生很多诡异的Bug，具体可以看[这里](https://developers.weixin.qq.com/blogdetail?action=get_post_info&docid=000264ad214998b4b1f608ae05b800&highline=textarea%E5%B1%82%E7%BA%A7)和[这里](https://developers.weixin.qq.com/miniprogram/dev/component/textarea.html)的末尾几行。  

2、 `<scroll-view>`必须设置高度，否则无法监听相关事件。

3、<scroll-view>不能通过绝对定位，上下左右0，来设置全屏铺满，需调用`wx.getSystemInfo`方法获得屏高，设置`style=“height：{{ heightNumber }}px；”`
补充：可以在css中为其设置高度为 `vh （如：100vh）`无需js获取高度也可正常使用。

4、wxml的 {{ }} 中不能使用函数与方法，如：{{ useList[index].labIndex.indexOf(lab) > -1 ? 'lab_active' : '' }} ，可使用基本+-*/运算与三目运算等。

5、input获得焦点，软键盘弹出，光标与键盘默认距离为0，键盘会遮挡住部分元素，可设置cursor-spacing（int）来设置键盘与光标的距离，以显示完整元素

6、两个canvas叠加在一切，之后进行的一切绘制，dom层级最后的canvas所绘制的东西永远会在最前端。  
原因：canvas等元素使用的是原生组件，层级最高，故后面的后显示再最见面。
解决方法：将要放在下层的canvas的wxml标签写在前面。