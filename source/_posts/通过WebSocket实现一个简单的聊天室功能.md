---
title: 通过WebSocket实现一个简单的聊天室功能
date: 2017-10-17 22:03
tags: [WebSocket]
---

## WebSocket
WebSocket是一个协议，它是是基于TCP的一种新的网络协议，TCP协议是一种持续性的协议，和HTTP不同的是，它可以在服务器端主动向客户端推送消息。通过这个协议，可以在建立一个nodejs的服务器，然后所有的客户端都可以向服务器端发送消息，然后服务器端把这个消息广播出去，形成了一个类似于聊天室的东西。
<!--more-->
## 客户端：
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>websoket</title>
</head>
<body>
    <h1>chat room</h1>
    <input type="text" id="msg" />
    <button id="send">发送</button>
    <script type="text/javascript">
        var websocket = new WebSocket("ws://localhost:6666/");

        function showMsg(str){
            var div = document.createElement('div');
            div.innerHTML = str;
            document.body.appendChild(div)
        }

        websocket.onopen=function(){
            console.log("open");
            document.getElementById('send').onclick = function() {
                var txt = document.getElementById('msg').value;
                if (txt) {
                    websocket.send(txt);
                }
            }
        }
        websocket.onclose = function() {
            console.log("close");
        }
        websocket.onmessage = function(e) {
            console.log(e.data);
            showMsg(e.data);
        }
    </script>
</body>
</html>
```
从我的服务器localhost:6666实例化一个新的websocket，然后打开他监听发送按钮，点击发送就把txt发送到服务器端，然后监听得到的消息，通过showMsg渲染到界面上去

## 服务器端(node.js)：
```
var ws = require("nodejs-websocket")

var port = 6666;

var clientCount = 0;

var server = ws.createServer(function (conn) {
    console.log("New connection")
    clientCount++
    conn.nickname = "user" + clientCount
    broadcast("******* "+conn.nickname + " comes in *******");


    conn.on("text", function (str) {
        console.log("Received "+str)
        broadcast(conn.nickname + " say: " + str)
    })


    conn.on("close", function (code, reason) {
        broadcast("******* " + conn.nickname + " left *******");
    })
    conn.on("error", function(err) {
        console.log("error: "+err);
    })
}).listen(port)

console.log("websocket server listening on " + port);

function broadcast (str) {
    server.connections.forEach(function (connection) {
        connection.sendText(str)
    })
}
```
 之前要加载一下nodejs-websocket模块，来一个人就把计数器加1，然后给他设置名字，监听收到的消息text，有消息就执行broadcast，broadcast就是向所有的客户端广播新的消息

## 举个例子
这里是我服务器上的栗子，大家可以看看

http://www.xiedashuaige.cn/websocket.html