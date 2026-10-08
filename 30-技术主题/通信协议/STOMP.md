# STOMP协议


STOMP即Simple (or Streaming) Text Orientated Messaging Protocol，简单(流)文本定向消息协议，它提供了一个可互操作的连接格式，允许STOMP客户端与任意STOMP消息代理（Broker）进行交互.



# STOMP帧


STOMP帧由命令，一个或多个头信息、一个空行及负载（文本或字节）所组成；



其中可用的COMMAND 包括：



> CONNECT、SEND、SUBSCRIBE、UNSUBSCRIBE、BEGIN、COMMIT、ABORT、ACK、NACK、DISCONNECT；
>



例如：  
发送消息



> SEND  
destination:/queue/trade  
content-type:application/json  
content-length:44  
{“action”:”BUY”,”ticker”:”MMM”,”shares”,44}^@
>



订阅消息



> SUBSCRIBE  
id:sub-1  
destination:/topic/price.stock.*  
^@
>



服务器进行广播消息



> MESSAGE  
message-id:nxahklf6-1  
subscription:sub-1  
destination:/topic/price.stock.MMM  
{“ticker”:”MMM”,”price”:129.45}^@
>



# STOMP 客户端 API


## 引入stomp.js sockjs.js (自行下载)


```html
  <script src="js/sockjs.min.js"></script>
  <script src="js/stomp.min.js"></script>
```



## 发起连接


客户端可以通过使用Stomp.js和sockjs-client连接



```javascript
// 建立连接对象（还未发起连接）后端WebSocket服务的连接地址
var socket=new SockJS("http://localhost:8080/privateServer");

// 获取 STOMP 子协议的客户端对象
var client = Stomp.over(socket);

// 向服务器发起websocket连接并发送CONNECT帧
client.connect(
    {},
    // 连接成功时（服务器响应 CONNECTED 帧）的回调方法
    function connectCallback (frame) {
        console.log('已连接【' + frame + '】');
        client.subscribe('/topic/getResponse', function (response) {
            showResponse(response.body);
        });
    },
    // 连接失败时（服务器响应 ERROR 帧）的回调方法
	function errorCallBack (error) {
        console.log('连接失败【' + error + '】');
    }
);
```



client.connect()方法签名：



```javascript
client.connect(headers, connectCallback, errorCallback);
```



其中 **headers** 表示客户端的认证信息  如：



```javascript
var headers = {
  login: 'mylogin',
  passcode: 'mypasscode',
  // additional header
  'client-id': 'my-client-id'
};
```



若无需认证，直接使用空对象 “{}” 即可；



**connectCallback** 表示连接成功时（服务器响应 CONNECTED 帧）的回调方法；  
**errorCallback** 表示连接失败时（服务器响应 ERROR 帧）的回调方法，非必须；



## 断开连接


若要从客户端主动断开连接，可调用 disconnect() 方法



```javascript
client.disconnect(function () {
   alert("See you next time!");
};
```



该方法为异步进行，因此包含了回调参数，操作完成时自动回调；



关闭浏览器时自动断开连接



```javascript
window.onunload = function () {
    client.disconnect(function () {
        alert("See you next time!");
    };
}
```



## 心跳机制


若使用STOMP 1.1 版本，默认开启了心跳检测机制，可通过client对象的 **heartbeat** 字段进行配置（默认值都是10000 ms）：



```javascript
client.heartbeat.outgoing = 20000;  // client will send heartbeats every 20000ms
client.heartbeat.incoming = 0;      // client does not want to receive heartbeats from the server
// The heart-beating is using window.setInterval() to regularly send heart-beats and/or check server heart-be
```



## 发送信息


连接成功后，客户端可使用 send() 方法向服务器发送信息：



```javascript
client.send(destination url, headers, body);
```



其中  
destination url 为服务器 controller中 [@MessageMapping ](/MessageMapping ) 中匹配的URL，字符串，必需参数；   
headers 为发送信息的header，JavaScript 对象，可选参数；  
body 为发送信息的 body，字符串，可选参数；



## 订阅、接收信息


STOMP 客户端要想接收来自服务器推送的消息，必须先订阅相应的URL，即发送一个 SUBSCRIBE 帧，然后才能不断接收来自服务器的推送消息；  
订阅和接收消息通过 subscribe() 方法实现：



```javascript
subscribe(destination url, callback, headers)
```



该方法返回一个包含了id属性的 JavaScript 对象，可作为 unsubscribe() 方法的参数；  
其中  
destination url 为服务器 [@SendTo ](/SendTo ) 匹配的 URL, 字符串,  必需参数；   
callback 为每次收到服务器推送的消息时的回调方法，该方法包含参数 message, 必需参数；  
headers 为附加的headers，JavaScript 对象, 可选参数;



## 取消订阅


```javascript
// 订阅, 获取一个返回值并接收, 此返回值用来取消订阅
var subscription = client.subscribe(...);
subscription.unsubscribe();
```



## JSON 支持


STOMP 帧的 body 必须是 string 类型，若希望接收/发送 json 对象，可通过 JSON.stringify() 和 JSON.parse() 实现；  
例：



```javascript
var quote = {symbol: 'APPL', value: 195.46};
client.send("/topic/stocks", {}, JSON.stringify(quote));

client.subcribe("/topic/stocks", function(message) {
var quote = JSON.parse(message.body);
alert(quote.symbol + " is at " + quote.value);
});
```



## 事务支持


STOMP 客户端支持在发送消息时用事务进行处理：  
举例说明：



```javascript
// 开启事务 该方法返回一个包含了事务ID、commit()、abort() 的JavaScript 对象
var tx = client.begin();

// 发送消息 最关键的在于要在 headers 对象中加入事务ID，若没有添加，则会直接发送消息，不会以事务进行处理
client.send("/queue/test", {transaction: tx.id}, "message in a transaction");
// 提交事务
tx.commit();
// tx.abort(); 中止
```



## Debug 信息


STOMP 客户端默认将传输过程中的所有 debug 信息以 console.log() 形式输出到客户端浏览器中，也可通过以下方式输出到 DOM 中：



```javascript
client.debug = function(str) {
    // str 参数即为 debug 信息
    // 使用 JQuery 将调试日志附加到页面中某处的#debug div
    $("#debug").append(str + "\n");
};
```

