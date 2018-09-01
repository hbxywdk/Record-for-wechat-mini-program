# websocket-in-wechat-mini-program
微信小程序中使用websocket的一种思路

###### App中连接socket，并接收消息，Page中订阅消息，App收到消息后向订阅的Page下发消息，Page销毁时，删除该Page的处理函数。
```
App({
  onLaunch: function (options) {
    // new 聊天实例
    this.$chat = new Chat(this)
    this.$chat.connectSocket() // 连接
  },
  $chat: null
})
```
###### Chat.js
```
/**
 * Chat
 * 
 */
import baseConfig from '../config/urlConfig'
import { wxLogin } from './login'

export default class Chat {

  constructor (app) {
    this.chat_id = null // chat_id
    this.connectStatus = 0 // websocket 连接状态 0：未连接，1：已连接
    this.heartListen = null // 心跳
    this.watcherList = [] // 订阅者
    this.app = app // 方便在Chat内部操作app
  }

  /* 初始化连接 */
  connectSocket () {
    const token = wx.getStorageSync('token')
    const url = `${baseConfig.socketUrl}/websocket/socketServer.do?token=${token}`
    this.chat_id = new Date().getTime()
    console.log('开始连接')
    // websocket连接
    wx.connectSocket({
      url: url,
      header: {
        'content-type': 'application/json'
      },
      method: 'post',
      success: res => {
        console.log('连接成功', res)
        // 设置连接状态
        this.connectStatus = 1
        // 心跳
        clearInterval(this.heartListen)
        this.heartListen = setInterval(() => {
          if (this.connectStatus === 0) {
            console.log('监听到没心跳了，抢救一下')
            clearInterval(this.heartListen)
            this.reconnect()
          } else {
            // console.log('我还活着')
          }
        }, 3000)
      },
      fail: err => {
        console.error('连接失败')
      }
    })
    // 监听webSocket错误
    wx.onSocketError(res => {
      console.log('监听到 WebSocket 打开错误，请检查！')
      // 修改连接状态
      this.connectStatus = 0
    })
    // 监听WebSocket关闭
    wx.onSocketClose(res => {
      console.log('监听到 WebSocket 已关闭！')
      this.connectStatus = 0
    })
    // websocket打开
    wx.onSocketOpen(res => {
      console.log('监听到 WebSocket 连接已打开！')
    })
    // 收到websocket消息
    wx.onSocketMessage(res => {
      this.getSocketMsg(JSON.parse(res.data))  // 收到的消息为字符串，需处理一下
    })
  
  }

  /* 重连 */
  reconnect() {
    console.log('尝试重连')
    wx.closeSocket() // 重连之前手动关闭一次
    this.connectSocket()
  }

  /* 关闭websocket */
  closeSocket(removeChat) {
    wx.closeSocket({
      success: res => {
        // code
      }
    })
  }

  /* 添加watcher */
  addWatcher (fn) {
    this.watcherList.push(fn)
    return this.watcherList.length - 1 // 返回添加位置的下标，Page unload的时候方便删除List成员
  }

  /* 删除watcher */
  delWatcher (index) {
    this.watcherList.splice(index, 1)
    // console.log('销毁watcher', this.watcherList)
  }

  /* 收到消息 */
  getSocketMsg (data) {
    console.log('收到消息', data)
    if (data.code === 5100) { // 处理登录过期
      wxLogin()
        .then(res => {
          console.log('登录成功')
          // 重新登录成功，发起重连
          this.reconnect()
        })
        .catch(err => {
          console.error('登录失败', err)
        })
    // 正确状态
    } else if (data.code === 0) {
      // 给每个订阅者发消息
      const list = this.watcherList
      for (let i = 0; i < list.length; i++) {
        list[i](data)
      }
    // 其他返回类型
    } else {
      // balabalabala
    }
  }

  // 这里可以写一些方法，如发送消息等
  // code

}

```
###### PageA内处理
```
// PageA
const app = getApp()
Page({
  data:{
    watcherIndex: ''
  },
  // 处理收到websocket消息
  dealFn (data) {
    console.log('这是在 PageA 中收到的消息', data)
  },
  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    const watcherIndex = app.$chat.addWatcher(this.dealFn)
    this.setData({ watcherIndex })
    console.log('添加至 watcherList', app.$chat.watcherList)
  },
  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {
    // 销毁
    app.$chat.delWatcher(this.data.watcherIndex)
  }
})
```
###### PageB内处理
```
// PageB
const app = getApp()
Page({
  data:{
    watcherIndex: ''
  },
  // 处理收到websocket消息
  dealFn (data) {
    console.log('这是在 PageB 中收到的消息', data)
  },
  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    const watcherIndex = app.$chat.addWatcher(this.dealFn)
    this.setData({ watcherIndex })
    console.log('添加至 watcherList', app.$chat.watcherList)
  },
  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {
    // 销毁
    app.$chat.delWatcher(this.data.watcherIndex)
  }
})
```
