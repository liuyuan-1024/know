# WebSocket


WebSocket是基于TCP协议的**全双工的、持久的通信协议**。



全双工：通信允许数据在两个方向上同时传输.



## 一、nodejs搭建WebSocket服务器


1.  安装ws环境：npm install ws 
2.  安装nodejs-websocket环境：npm install nodejs-websocket 
3.  搭建WebSocket服务器：创建index.js，并编写以下代码 

```javascript
// 导入nodejs-websocket
const ws = require('nodejs-websocket')

// 服务器的端口
const PORT = 3000

// 创建服务器，每次用户连接，都会创建一个connect对象
const server = ws.createServer(function (connect) {
    console.log('用户连接了服务');

    // 监听用户发送数据事件
    connect.on("text", function (data) {
        console.log("用户发送的信息：" + data);

        connect.send(data);
    });

    // 监听关闭连接事件
    connect.on("close", function (code, reason) {
        console.log("连接关闭")
    });

    // 监听error事件，连接断开也会报error事件，error会导致服务不再执行
    connect.on('error', function () {
        console.log('出错了!!!');
    });
});

// 监听服务器端口
server.listen(PORT, () => {
    console.log('websocket服务启动成功!!!监听了端口' + PORT);
})
```

 

4.  开启WebSocket服务：打开index.js所在终端，并键入“**node index.js**+TAB+回车” 



## 二、客户端建立连接


```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>WebSocket</title>
    <style>
        div {
            width: 300px;
            height: 300px;
            border: 1px solid #111;
        }
    </style>
</head>

<body>
    <input type="text" placeholder="请输入内容">
    <button>发送</button><br>
    <div></div>

    <script>
        var input = document.querySelector('input')
        var button = document.querySelector('button')
        var div = document.querySelector('div')

        // 连接服务器
        var socket = new WebSocket('ws://localhost:3000')

        // 监听连接成功事件
        socket.addEventListener('open', function () {
            div.innerHTML = '连接服务成功'
        })

        // 给服务器发送数据
        button.addEventListener('click', function () {
            socket.send(input.value)
        });

        // 接收服务器发送的数据
        socket.addEventListener('message', function (msg) {
            div.innerHTML = msg.data;
        });
    </script>
</body>

</html>
```



## 三、socket.io创建WebSocket服务器和客户端


1. 下载socket.io：npm install socket.io express
2. 下载socket.io客户端：npm install socket.io-client



在**io.on('connection', (socket) => {});**中socket有两个方法：



1. socket.emit('事件名', 待发送的事件数据);
2. socket.on('事件名', 事件处理函数);



### 1、socket.io创建WebSocket服务器


```javascript
// 导入express
const express = require('express');
const app = express();

// 导入http
const http = require('http');

// http使用express创建了一个服务器
const server = http.createServer(app);

// 监听3000端口
server.listen(3000, () => {
    console.log('服务器在3000端口启动成功');
});

// socket.io服务可以作为独立服务，或添加到一个已存在服务上(HTTP,Express,Koa,Nest)
const io = require("socket.io")(server);

// express处理静态资源：将public目录下的资源设置为静态资源
app.use(express.static('public'));

app.get('/', (req, res) => {
    res.redirect('/index.html');
});

io.on('connection', (socket) => {
    console.log('有用户连接了服务器');
});

// 广播通知所有用户
io.emit()
```



### 2、socket.io创建WebSocket客户端


```html
<!-- 引入socket.io.js -->
<script src="/socket.io/socket.io.js"></script>
```



## 四、jQuery-emoji


[jQuery-emoji官网](http://eshengsky.github.io/jQuery-emoji/)



安装jQuery-emoji：npm install --save jQuery-emoji



jQuery-emoji的使用：



	首先在页面上引用css文件和js文件，css文件一般在中添加，js文件一般在之前添加。注意要先引用jquery和jquery.mCustomScrollbar的js、css，再引用该js。



```html
jquery.mCustomScrollbar.min.css
jquery.mCustomScrollbar.min.js

jquery.emoji.css
jquery.emoji.js
```



初始化emoji：



```javascript
按钮选择器.on('click', funcation(){	//表情选择面板按钮注册点击事件，当点击按钮时初始化emoji
    //初始emoji
    选择器.emoji({
    button: 选择器,	  //表情选择面板绑定到该按钮，若未指定，则自动创建一个按钮
    showTab: false,		//当只有一组表情时，是否仍然显示Tab导航栏
    animation: 'slide',	 //表情选择面板的动画效果
    position: 'topLeft', //表情面板相对按钮的位置，可能的值：'bottomRight'，'bottomLeft'，'topRight'，'topLeft'
    icons: [{			//表情包组
        name: "QQ表情",
        path: "img/qq/",
        maxNum: 91,
        excludeNums: [41, 45, 54],
        file: ".gif",
        placeholder: "#qq_{alias}#"
    },{}]
})
})
```



发送表情时，**textarea**是无法显示表情的，而是显示表情的唯一标识字符串，当我们想要在输入时就显示表情，可以用**div**代替*textarea作为文本输入区域，但是div需要添加**属性：contenteditable**，使得div内容可编辑。

