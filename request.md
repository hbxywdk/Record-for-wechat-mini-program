##### 场景：每次请求必须为已登录状态
###### 使用:
```
    app.$http({
      url: '/api/test',
      method: 'post',
      params: {
        page: 1
      },
      successFn: res => {
        console.log('success', res)
      },
      failFn: err => {
        console.error('fail', err)
      },
      completeFn: res => {
        console.log('complete', res)
      }
    })
```
###### request.js
```
import { wxLogin } from '../utils/login'
import { requestUrl } from '../config/urlConfig'
import { needHttpLog } from '../config/index'

let onLogin = false // 是否在登录中
let overdueToken = false // token是否过期
let requestList = [] // 请求队列

function wxRequest(options) {
  let defaultOptions = { // 默认参数
    params: {},
    method: 'post',
    header: { 'content-type': 'application/json' },
    needToken: true,
    successFn(res) {},
    failFn(err) {},
    completeFn(res) {}
  }
  // 合并参数
  if (options) {
    for (let item in defaultOptions) {
      if (options[item] === undefined) { options[item] = defaultOptions[item] }
    }
  }
  options = options || defaultOptions
  // 需要uid的请求
  if (options.uidParam) {
    options.params[options.uidParam] = wx.getStorageSync('uid')
  }
  // 需要token 的请求
  if (options.needToken) {
    let token
    try { token = wx.getStorageSync('token') || '' } catch (err) { token = '' }
    options.header.token = token
  }
  // 需要token但没有token
  if (options.needToken && options.header.token === '') {
    console.log('token为空')
    if (!onLogin) {
      // 未登录的情况下，并行多个请求，只许发起一个登陆请求，其他请求先保存起来，成功登录后顺序执行
      onLogin = true
      wxLogin() // 登录
        .then(res => {
          console.log('登录成功')
          wxRequest(options) // 重新登录成功再次发起请求
        })
        .catch(err => {
          console.error('登录失败', err)
        })
    } else {
      requestList.push(options) // 保存其他请求
    }
  } else {
    // 非token过期的情况下，执行并清空请求
    if (requestList.length && !overdueToken) {
      clearRequestList()
    }
    if (options.url && typeof options.url === 'string' && options.url.length > 0) {
      // 如果需要向本地发送请求
      if (options.local) {
        options.url = options.ip + options.url
      } else {
        // 解决重新登陆后发起请求
        const urlReg = new RegExp(requestUrl, 'ig') // 解决url重复拼接问题
        options.url = urlReg.test(options.url) ? options.url : requestUrl + options.url
      }
      wx.request({
        url: options.url,
        method: options.method,
        data: options.params,
        header: options.header,
        success: function (res) {
          needHttpLog && requestLog(options.url, options.params, options.header, res)
          // 正常状态
          if (res.statusCode === 200) {
            options.successFn(res)
          } else {
            // 登录失效，重新登录
            if (res.data.code === 10025 || res.data.code === 10026) {
              if (!onLogin && !overdueToken) {
                onLogin = true
                overdueToken = true
                wxLogin()
                  .then(res => {
                    console.log('登录成功')
                    // 重新登录成功再次发起请求
                    wxRequest(options)
                    clearRequestList()
                    overdueToken = false
                  })
                  .catch(err => {
                    console.error('登录失败', err)
                  })
              } else {
                requestList.push(options)
              }

            }
          }
        },
        fail: function (err) {
          needHttpLog && requestLog(options.url, options.params, options.header, err)
          options.failFn(err)
        },
        complete: function (res) {
          options.completeFn(res)
        }
      })
    }

  }
}

// 打请求log
function requestLog(url, params, header, res) {
    console.warn('当前请求地址：', url, '当前请求参数：', params, '请求header：', header, '本次请求UID', wx.getStorageSync('uid'), '请求返回结果：', res)
  }

// 执行等待请求队列，并清空
function clearRequestList () {
  requestList.forEach(e => {
    setTimeout(() => { wxRequest(e) }, 0)
  })
  onLogin = false
  requestList = []
}

module.exports = {
  wxRequest
}
```
###### 其他说明:
```
/*
    1.调试本地接口
    传入 local: true 与 ip: 'xxxxxx'，其他不变
    app.$http({
      local: true, 
      ip: 'http://192.168.1.45'
      // code
    })
    2.需要uid的情景
    如过用户信息为空，但请求又用到了uid
    wx.getStorageSync('uid')
    此时user_id为空，wxRequest会重新登录，之后再用原请求参数data再次发起请求，就会出现虽然登录了，参数user_id却为空。
    应将请求写为如下形式
    app.$http({
      uidParam: 'user_id', // 添加uidParam一项，值为接口参数名
    })
 */
```
