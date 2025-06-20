---
layout: page
title: socket.io连接
tags: 服务器
categories: socket.io
---

## Socket.IO
Socket.IO 是一个库，可以在客户端和服务器之间实现 低延迟, 双向 和 基于事件的 通信。

*Socket.IO 不是 WebSocket实现。*
WebSocket 客户端将无法成功连接到 Socket.IO 服务器，而 Socket.IO 客户端也将无法连接到普通 WebSocket 服务器

### 服务端使用node.js

```ts
/**
 * websocket服务
 * 使用socket.io
 */
const app = express();
const SocketServer = http.createServer(app);
const io = new Server(SocketServer, {
    cors: {
        origin: '*',//跨域问题
    },
    connectionStateRecovery: {
        // 会话和报文的备份时间
        maxDisconnectionDuration: 60 * 1000,
        // 恢复成功后是否跳过中间件
        skipMiddlewares: true,
    },
    // 发送新的ping packet（30000）之前有多少ms
    pingInterval: 60000,
    // 有多少ms没有传递消息则考虑连接close（5000）
    pingTimeout: 10000,
});
```

#### 连接事件
```ts
io.on('connection', async (socket) => {
    consol.log('连接成功')
})
```
#### 监听接收事件
可以在连接成功的时候监听对应事件
```ts
// socket为上述连接成功后的实例
 socket.on('eventName',(value:any,callback:()=>{})=>{
    console.log('接收名为eventName事件的传输')
 })
```
#### 发送事件
也可以连接成功时发送消息到客户端
```ts
/**
 * 发送名为eventName事件给客户端，传输数据为value
 */
// socket为上述连接成功后的实例

//这是发送给所有人，包括自己
 socket.emit('eventName',(value:any));

 //这是发送给除了自己以外的其他连接客户端
 socket.broadcast.emit('onlineCount', value:any)
 
```

### 客户端使用(react为例)

#### redux + socket.io.client
```js
import store from "../../redux/store/configureStore";
import { saveSocket } from 'redux/actions/socket'

import io from 'socket.io-client';
/**
 * @Description: 建立socket连接
 * @name: createConnect
 * @param {*} path 传入格式模板'localhost:8080',连接地址，当传入为空时会取域名
 * @return {*}
 */
export function createConnect(path) {
    const state = store.getState()
    if (!state.socket.socket) {
        const userInfo = sessionStorage.getItem('userLogin') ? JSON.parse(sessionStorage.getItem('userLogin')) : {}
        /* 判断redux中是否已经存在socket对象，存在证明已经连接直接返回socket对象，避免重复连接 */

        let origin = 'portal.ly-sky.com:6203'//后端socket服务地址
        let socket = io('wss://' + origin);/* 建立连接 */

        // let origin = 'localhost:3011'//后端socket服务地址
        // let socket = io('ws://' + origin);/* 建立连接 */

        const userItem = {
            userName: userInfo.userName,
            userId: userInfo.userId,
            header: userInfo.headPortrait
        }
        socket.emit('userOnline', userItem)//发送用户数据
        store.dispatch(saveSocket(socket));/* 把返回的对象存入redux中 */
    }

    return state.socket.socket/* 返回redux中保存的socket对象 */
}
```
#### 组件中创建连接

```js
class Index extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            userInfo: {},
            pageNum: 1,
            pageSize: 20,
            open: false,
            onlineUsers: [],
            onLineUserOpen: false,
            value: '',
            list: []
            // list: Array.from({ length: 100 }).fill({ userName: '测试', value: '测试数据', date: '2024-10-09 15:55' })
        }
        // 建立socket连接
        this.sockets = createConnect();
    }
}
```

#### 事件监听（api与服务端一致）
```js
//接收
sockets.on('connect', () => {
            //监听连接是否成功
            console.log('socket连接成功');
        });
//发送
socket.emit('sendMessage', userItem)
```
