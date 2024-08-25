---
title: Vue 全局配置 WebSocket
date: 2023-05-09 00:06	
categories:
  - Vue
  - JS
---



> 服务端可以主动向客户端推送数据，浏览器和服务器只需要完成一次握手，两者之间就可以创建持久性的连接，并进行双向数据传输

### 创建文件/xx/global.js，且在main中引用

```javascript
// global.js 文件
export default {
    ws: {},
    setWs: function(newWs) {
        this.ws = newWs
    }
}
```

```javascript
//vue2 main.js 文件
import global from './xx/global.js'
Vue.prototype.global = global //vue2

// vue3 main.js 文件
import global from './xx/global.js'
const app = createApp(App)
app.config.globalProperties.global = global
```

### 在合适地方进行初始化

```javascript
const localSocket=()=>{
  let that = this
  if ("WebSocket" in window) {
      that.ws = new WebSocket("ws://localhost:8081/xxx)
      that.global.setWs(that.ws)
      that.ws.onopen = function () {
          console.log('websocket连接成功')
      };

      that.ws.onclose = function () {
          // 关闭 websocket
          console.log("连接已关闭...")
          //断线重新连接
          // setTimeout(() => {
          //     that.localSocket()
          // }, 2000)
      };
  } else {
      // 浏览器不支持 WebSocket
      console.log("您的浏览器不支持 WebSocket!")
      this.openNotificationWithIcon('error', '浏览器', '您的浏览器不支持显示消息请更换', 1, 1)
  }
}
```

### 在Vue2中使用

```javascript
handdleMsg() {
            let that = this;
            console.log(that.global.ws);
            if (that.global.ws && that.global.ws.readyState == 1) {
                console.log("发送信息", this.data)
                that.global.ws.send(JSON.stringify(xxx))
            }
            that.global.ws.onmessage = function (res) {
                console.log("收到服务器内容", res)
                let obj = JSON.parse(res.data)
              	//操作
            }
        }
```

### 在Vue3中使用

```javascript
import { ref, reactive, getCurrentInstance, computed } from 'vue'
const { appContext: { config: { globalProperties: global } } } = getCurrentInstance()

const handdleMsg = () => {
    let that = global;
    console.log(that.global.ws);
    if (that.global.ws && that.global.ws.readyState == 1) {
        console.log("发送信息", data.arr);
        that.global.ws.send(JSON.stringify({ ...data.arr, t: list.arr[data.arr.t].t }));
        data.arr.content = ''
    }
    that.global.ws.onmessage = function (res) {
        console.log("收到服务器内容", res);
        let obj = JSON.parse(res.data)
        let i = data.arr.t
        console.log(i);
        for (let index = 0; index < list.arr.length; index++) {
            if (list.arr[index].t === obj.t || list.arr[index].t === obj.f) {
                i = index
                break
            }
        }
        chats.arr[i].push(obj)
    };
}
```