---
title: 通过web实现IM功能
date: 2017-12-19 15:29:51
tags:
- IM
- Ajax
- WebSocket
---
### web 的通信方式
　　浏览器和我们桌面应用的工作方式是不同的，桌面应用通过socket和远程主机上另外一端的一个进程建立TCP连接，从而达到全双工的即时通信。浏览器是客户端请求服务器，服务器返回结果的模式。浏览器要想实现两个客户端的通信，必然要通过服务器进行信息的转发。例如A要和B通信，则应该是A先把信息发送给IM应用服务器，服务器根据A信息中携带的接收者将它再转发给B，同样B到A也是这种模式，如下所示：
![](/images/191409262329047.png)

### web 下 IM 特点
- 双全工通信：浏览器拉取（pull）服务器数据，服务器推送（push）数据到浏览器；
- 低延迟：浏览器A发送给B的信息经过服务器要快速转发给B，同理B的信息也要快速交给A；
- 支持跨域：通常客户端浏览器和服务器都是处于网络的不同位置，浏览器本身不允许通过脚本直接访问不同域名下的服务器。

### 解决方案
#### Ajax 短轮询服务器(polling)
　　这是最简单的一种解决方案。
　　原理：在客户端通过Ajax的方式的方式每隔一小段时间就发送一个请求到服务器，服务器返回最新数据，然后客户端根据获得的数据来更新界面。
　　***优点是简单，缺点是对服务器压力较大，浪费带宽流量（通常情况下数据都是没有发生改变的）。***

创建一个XHR对象，每2秒就请求服务器一次获取服务器时间并打印出来,客户端Javascript代码：
```javascript
function createXHR(){
        if(typeof XMLHttpRequest !='undefined'){
            return new XMLHttpRequest();
        }else if(typeof ActiveXObject !='undefined' ){
            if(typeof arguments.callee.activeXString!="string"){
            var versions=["MSXML2.XMLHttp.6.0","MSXML2.XMLHttp.3.0", "MSXML2.XMLHttp"],i,len;
            for(i=0,len=versions.length;i<len;i++){
                try{
                    new ActiveXObject(versions[i]);
                    arguments.callee.activeXString=versions[i];
                    break;
                }catch(ex) {

                }
            }
        }
        return new ActiveXObject(arguments.callee.activeXString);
       }else{
            throw new Error("no xhr object available");
        }
    }
    function polling(url,method,data){
       method=method ||'get';
       data=data || null;
       var xhr=createXHR();
        xhr.onreadystatechange=function(){
            if(xhr.readyState==4){
                if(xhr.status>=200&&xhr.status<300||xhr.status==304){
                    console.log(xhr.responseText);
                }else{
                    console.log("fail");
                }
            }
        };
        xhr.open(method,url,true);
        xhr.send(data);
    }
    setInterval(function(){
        polling('http://localhost:8088/time','get');
    },2000);
```

服务器NodeJS代码：
```javascript
var http=require('http');
var server=http.createServer(function(req,res){
if(req.url=='/time'){
    //res.writeHead(200, {'Content-Type': 'text/plain','Access-Control-Allow-Origin':'http://localhost'});
    res.end(new Date().toLocaleString());
};
}).listen(8088,'localhost');
server.on('connection',function(socket){
    console.log("客户端连接已经建立");
});
server.on('close',function(){
    console.log('服务器被关闭');
});
```
结果如下：
![](/images/191446187483346.png)
#### Ajax 长轮询服务器（long-polling）
　　`Ajax长轮询服务器`（long-polling），解决 `Ajax短轮询服务器` 浪费带宽流量的问题。
　　原理：客户端发送一个请求到服务器，服务器查看客户端请求的数据是否发生了变化（是否有最新数据），如果发生变化则立即响应返回，否则保持这个连接并定期检查最新数据，直到发生了数据更新或连接超时。同时客户端连接一旦断开，则再次发出请求，这样在相同时间内大大减少了客户端请求服务器的次数。
　　***优点是节省了带宽流量，缺点是客户端断开服务端还在执行。因为采用的是服务端阻塞方式（死循环，保持响应不返回）。***

客户端Javascript代码：
```js
function createXHR(){
        if(typeof XMLHttpRequest !='undefined'){
            return new XMLHttpRequest();
        }else if(typeof ActiveXObject !='undefined' ){
            if(typeof arguments.callee.activeXString!="string"){
                var versions=["MSXML2.XMLHttp.6.0","MSXML2.XMLHttp.3.0",
                            "MSXML2.XMLHttp"],
                        i,len;
                for(i=0,len=versions.length;i<len;i++){
                    try{
                        new ActiveXObject(versions[i]);
                        arguments.callee.activeXString=versions[i];
                        break;
                    }catch(ex) {

                    }
                }
            }
            return new ActiveXObject(arguments.callee.activeXString);
        }else{
            throw new Error("no xhr object available");
        }
    }
    function longPolling(url,method,data){
        method=method ||'get';
        data=data || null;
        var xhr=createXHR();
        xhr.onreadystatechange=function(){
            if(xhr.readyState==4){
                if(xhr.status>=200&&xhr.status<300||xhr.status==304){
                    console.log(xhr.responseText);
                }else{
                    console.log("fail");
                }
                longPolling(url,method,data);
            }
        };
        xhr.open(method,url,true);
        xhr.send(data);
    }
    longPolling('http://localhost:8088/time','get');
```

服务端NodeJS代码：
```js 
var http=require('http');
var server=http.createServer(function(req,res){
    if(req.url=='/time'){
        setInterval(function(){
            sendData(res);
        },20000);
    };
}).listen(8088,'localhost');
//用随机数模拟数据是否变化
function sendData(res){
    var randomNum=Math.floor(10*Math.random());
    console.log(randomNum);
    if(randomNum>=0&&randomNum<=5){
        res.end(new Date().toLocaleString());
    }
}
```
　　在服务端通过生成一个在1到9之间的随机数来模拟判断数据是否发生了变化，当随机数在0到5之间表示数据发生了变化，直接返回，否则保持连接，每隔2秒再检测。可以看到返回的时间是没有规律的，并且单位时间内返回的响应数相比polling方式较少。
结果如下：
![](/images/191504150763288.png)

　　`Ajax短轮询服务器`和`Ajax长轮询服务器`都是一次性传输数据，在传输数据较小时没有问题，在数据量较大时可采用`http-stream方式`。

#### http-stream方式
　　原理：客户端在一次请求中保持和服务端连接不断开，然后服务端源源不断传送数据给客户端，就好比数据流一样，并不是一次性将数据全部发给客户端。它与polling方式的区别在于整个通信过程客户端只发送一次请求，然后服务端保持与客户端的长连接，并利用这个连接在回送数据给客户端。

##### 基于ajax的http-stream方式
　　原理：构造一个XHR对象，通过监听它的onreadystatechange事件，当它的readyState为3的时候，获取它的responseText然后进行处理，readyState为3表示数据传送中，整个通信过程还没有结束，所以它还在不断获取服务端发送过来的数据，直到readyState为4的时候才表示数据发送完毕，一次通信过程结束。
　　***缺点是：不兼容IE***
客户端JavaScript代码：
``` javascript
function createStreamClient(url,progress,done){
        //received为接收到数据的计数器
        var xhr=new XMLHttpRequest(),received=0;
        xhr.open("get",url,true);
        xhr.onreadystatechange=function(){
            var result;
            if(xhr.readyState==3){
                //console.log(xhr.responseText);
                result=xhr.responseText.substring(received);
                received+=result.length;
                progress(result);
            }else if(xhr.readyState==4){
                done(xhr.responseText);
            }
        };
        xhr.send(null);
        return xhr;
    }
    var client=createStreamClient("http://localhost:8088/stream",function(data){
        console.log("Received:"+data);
    },function(data){
        console.log("Done,the last data is:"+data);
    })
```
　　这里由于客户端收到的数据是分段发过来的，所以最好定义一个游标received，来获取最新数据而舍弃之前已经接收到的数据，通过这个游标每次将接收到的最新数据打印出来，并且在通信结束后打印出整个responseText。

服务端NodeJS代码：
```javascript
var http = require('http');
var count = 0;
var server = http.createServer(function(req, res){
    if(req.url == '/stream'){
        res.setHeader('content-type', 'multipart/octet-stream');
        var timer=setInterval(function(){
            sendRandomData(timer, res);
        }, 2000);
    };
}).listen(8088, 'localhost');
function sendRandomData(timer, res){
    var randomNum = Math.floor(10000 * Math.random());
    console.log(randomNum);
    if(count++ == 10){
        clearInterval(timer);
        res.end(randomNum.toString());
    }
        res.write(randomNum.toString());
}
```

服务端通过计数器count将数据分十次发送，每次生成一个小于10000的随机数发送给客户端让它进行处理。
![](/images/191541195298304.png)

##### 基于iframe的http-stream方式
　　原理：由于低版本的IE不允许在XHR的readyState为3的时候获取其responseText属性，为了达到在IE上使用这个技术，又出现了基于iframe的数据流通信方式。具体来讲，就是在浏览器中动态载入一个iframe,让它的src属性指向请求的服务器的URL，实际上就是向服务器发送了一个http请求，然后在浏览器端创建一个处理数据的函数，在服务端通过iframe与浏览器的长连接定时输出数据给客户端，但是这个返回的数据并不是一般的数据，而是一个类似于<script type=\"text/javascript\">parent.process('"+randomNum.toString()+"')</script>脚本执行的方式，浏览器接收到这个数据就会将它解析成js代码并找到页面上指定的函数去执行，实际上是服务端间接使用自己的数据间接调用了客户端的代码，达到实时更新客户端的目的。
　　***缺点是：使用iframe请求服务端，服务端保持通信连接没有全部返回之前，浏览器title一直处于加载状态，并且底部也显示正在加载，用户体验不好。***

客户端Javascript代码：
```javascript
function process(data){
        console.log(data);
    }
var dataStream = function (url) {
    var ifr = document.createElement("iframe"),timer;
    ifr.src = url;
    document.body.appendChild(ifr);
};
    dataStream('http://localhost:8088/htmlfile');
```
服务端NodeJS代码：
```javascript
var http = require('http');
var count = 0;
var server = http.createServer(function(req, res){
    if(req.url == '/htmlfile'){
        res.setHeader('content-type', 'text/html');
        var timer = setInterval(function(){
            sendRandomData(timer, res);
        },2000);
    };
}).listen(8088, 'localhost');
function sendRandomData(timer, res){
    var randomNum = Math.floor(10000 * Math.random());
    console.log(randomNum.toString());
    if(count++ == 10){
        clearInterval(timer);
        res.end("<script type=\"text/javascript\">parent.process('"+randomNum.toString()+"')</script>");
    }
    res.write("<script type=\"text/javascript\">parent.process('"+randomNum.toString()+"')</script>");
}
```
客户端简单的把数据打印出来，服务端定时发送随机数给客户端，并调用客户端process函数。
在IE5中测试结果如下：
![](/images/191558011238845.png)

##### 基于htmlfile的http-stream方式
　　iframe的http-stream方式的升级版，解决浏览器title一直处于加载状态，并且底部也显示正在加载的问题。
　　原理：在IE中，动态生成一个htmlfile对象，这个对象ActiveX形式的com组件，它实际上就是一个在内存中实现的HTML文档，通过将生成的iframe添加到这个内存中的HTMLfile中，并利用iframe的数据流通信方式达到上面的效果。同时由于HTMLfile对象并不是直接添加到页面上的，所以并没有造成浏览器显示正在加载的现象。
客户端JavaScript代码：

```javascript
function connect_htmlfile(url, callback) {
            var transferDoc = new ActiveXObject("htmlfile");
            transferDoc.open();
            transferDoc.write(
                "<!DOCTYPE html><html><body><script  type=\"text/javascript\">"+
                "document.domain='" + document.domain + "';" +
                "<\/script><\/body><\/html>");
            transferDoc.close();
            var ifrDiv = transferDoc.createElement("div");
            transferDoc.body.appendChild(ifrDiv);
            ifrDiv.innerHTML = "<iframe src='" + url + "'><\/iframe>";
            transferDoc.callback=callback;
            setInterval( function () {}, 10000);
        }
        function prograss(data) {
            alert(data);
        }
        connect_htmlfile('http://localhost:8088/htmlfile',prograss);
```
服务端传送给iframe的是这样子
```
<script type=\"text/javascript\">callback.process('"+randomNum.toString()+"')</script>
```
这样就在iframe流的原有方式下避免了浏览器的加载状态。

### 服务端推送事件
　　HTML5提供了一种新的技术叫做服务器推送事件SSE，它能够实现客户端请求服务端，然后服务端利用与客户端建立的这条通信连接push数据给客户端，客户端接收数据并处理的目的。从独立的角度看，SSE技术提供的是从服务器单向推送数据给浏览器的功能，但是配合浏览器主动请求，实际上就实现了客户端和服务器的双向通信。它的原理是在客户端构造一个eventSource对象，该对象具有readySate属性，分别表示如下：
- 0：正在连接到服务器；
- 1：打开了连接；
- 2：关闭了连接。

　　同时eventSource对象会保持与服务器的长连接，断开后会自动重连，如果要强制连接可以调用它的close方法。可以它的监听onmessage事件，服务端遵循SSE数据传输的格式给客户端，客户端在onmessage事件触发时就能够接收到数据，从而进行某种处理。

客户端Javascript代码：
```javascript
var source=new EventSource('http://localhost:8088/evt');
    source.addEventListener('message', function(e) {
        console.log(e.data);
    }, false);
    source.onopen=function(){
        console.log('connected');
    }
    source.onerror=function(err){
        console.log(err);
    }
```

服务端：
```javascript
var http = require('http');
var fs = require("fs");
var count = 0;
var server = http.createServer(function(req, res){
    if(req.url == '/evt'){
        //res.setHeader('content-type', 'multipart/octet-stream');
        res.writeHead(200, { "Content-Type":"tex" +
            "t/event-stream", "Cache-Control" : "no-cache",
            'Access-Control-Allow-Origin' : '*',
            "Connection" : "keep-alive"});
        var timer = setInterval(function(){
            if(++count == 10){
                clearInterval(timer);
                res.end();
            }else{
                res.write('id: ' + count + '\n');
                res.write("data: " + new Date().toLocaleString() + '\n\n');
            }
        },2000);
    };
}).listen(8088,'localhost');
```
　　注意这里服务端发送的数据要遵循一定的格式，通常是id:（空格）数据（换行符）data：（空格）数据（两个换行符），如果不遵循这种格式，实际上客户端是会触发error事件的。这里的id是用来标识每次发送的数据的id,是强制要加的。
结果如下：
![](/images/191631319359663.png)

### 跨域的问题
#### cors实现跨域
在服务器请求头添加:
```
`Access-Control-Allow-Origin':'*'` 
```
为了安全需要，浏览器的跨域的XHR请求存在限制：
- 客户端不能使用setRequestHeader设置自定义头部；
- 不能发送和接收cookie；
- 调用getAllResponseHeaders()方法总会返回空字符串。

以上这些措施都是为了安全考虑，防止常见的跨站点脚本攻击（XSS）和跨站点请求伪造（CSRF）。

***浏览器兼容性：适用于ff,safari,opera,chrome等非IE浏览器***

客户端JavaScript代码：
```javascript
var polling = function(){
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function(){
        if(xhr.readyState == 4){
            if(xhr.status == 200){
                console.log(xhr.responseText);
            }
        }
    }
    xhr.open('get','http://localhost:8088/cors');
    xhr.send(null);
};
setInterval(function(){
    polling();
},1000);
```

服务器NodeJS代码：
```javascript
var http = require('http');
var fs = require("fs");
var server = http.createServer(function(req, res){
    if(req.url == '/cors'){
            res.writeHead(200, {'Content-Type': 'text/plain','Access-Control-Allow-Origin':'http://localhost'});
            res.end(new Date().toString());
    }
    if(req.url == '/jsonp'){

    }
}).listen(8088, 'localhost');
server.on('connection',function(socket){
    console.log("客户端连接已经建立");
});
server.on('close',function(){
    console.log('服务器被关闭');
});
```

#### IE浏览器XDR跨域
对于IE8-10，它是不支持使用原生的XHR对象请求跨域服务器的，它自己实现了一个XDomainRequest对象，类似于XHR对象，能够发送跨域请求，它主要有以下限制：
- cookie不会随请求发送，也不会随响应返回；
- 只能设置请求头部信息中的Content-Type字段；
- 不能访问响应头部信息；
- 只支持Get和Post请求；
- 只支持IE8-IE10；

客户端JavaScript代码：
```javascript
var polling = function(){
    var xdr = new XDomainRequest();
    xdr.onload=function(){
        console.log(xdr.responseText);
    };
    xdr.onerror=function(){
        console.log('failed');
    };
    xdr.open('get','http://localhost:8088/cors');
    xdr.send(null);
};
setInterval(function(){
    polling();
},1000);
```

服务器NodeJS代码同上( `CORS实现跨域` )。

#### Jsonp的跨域
　　原理：利用HTML页面上script标签对跨域没有限制的特点，让它的src属性指向服务端请求的地址，其实是通过script标签发送了一个http请求，服务器接收到这个请求之后，返回的数据是自己的数据加上对客户端JS函数的调用。

客户端JavaScript代码：
```javascript
function callback(data){
    console.log("获得的跨域数据为:"+data);
}
function sendJsonp(url){
    var oScript=document.createElement("script");
    oScript.src=url;
    oScript.setAttribute('type',"text/javascript");
    document.getElementsByTagName('head')[0].appendChild(oScript);
}
setInterval(function(){
    sendJsonp('http://localhost:8088/jsonp?cb=callback');
},1000);
```

服务器NodeJS代码：
```javascript
var http = require('http');
var url = require('url');
var server = http.createServer(function(req, res){
    if(/\/jsonp/.test(req.url)){
        var urlData = url.parse(req.url, true);
        var methodName = urlData.query.cb;
        res.writeHead(200,{'Content-Type':'application/javascript'});
        //res.end("<script type=\"text/javascript\">"+methodName+"("+new Date().getTime()+");</script>");
        res.end(methodName+"("+new Date().getTime()+");");
        //res.end(new Date().toString());
    }
}).listen(8088,'localhost');
server.on('connection',function(socket){
    console.log("客户端连接已经建立");
});
server.on('close',function(){
    console.log('服务器被关闭');
});
```

### websocket实现
　　以上方案通常称为hack技术。
　　在HTML5中，浏览器为我们提供了websocket技术，支持H5浏览器的首选方案。
websocket整个工作过程：
　　首先是客户端new 一个websocket对象，该对象会发送一个http请求到服务端，服务端发现这是个webscoket请求，会同意协议转换，发送回客户端一个101状态码的response，以上过程称之为一次握手，经过这次握手之后，客户端就和服务端建立了一条TCP连接，应用层使用wss协议,在该连接上，服务端和客户端就可以进行双向通信了。

　　wss协议数据格式：
![](/images/191724033739549.png)
　　`FIN字段`，它占用1位，表示这是一个数据帧的结束标志，同时也下一个数据帧的开始标志。
　　`opcode字段`，它占用4位，当为1时，表示传递的是text帧，2表示二进制数据帧，8表示需要结束此次通信（就是客户端或者服务端哪个发送给对方这个字段，就表示对方要关闭连接了）。9表示发送的是一个ping数据。
　　`mask字段`占用1位，为1表示masking-key字段可用，
　　`masking-key字段`是用来对客户端发送来的数据做unmask操作的。它占用0到4个字节。
　　`Payload字段`表示实际发送的数据，可以是字符数据也可以是二进制数据。

客户端JavaScript代码：
```javascript
window.onload = function(){
    var ws = new WebSocket("ws://127.0.0.1:8088");
    var oText = document.getElementById('message');
    var oSend = document.getElementById('send');
    var oClose = document.getElementById('close');
    var oUl = document.getElementsByTagName('ul')[0];
    ws.onopen = function(){
        oSend.onclick = function(){
            if(!/^\s*$/.test(oText.value)){
                ws.send(oText.value);
            }
        };
    };
    ws.onmessage = function(msg){
      var str = "<li>"+msg.data+"</li>";
      oUl.innerHTML += str;
    };
    ws.onclose=function(e){
        console.log("已断开与服务器的连接");
        ws.close();
    }
}
```
　　客户端创建一个websocket对象，在onopen时间触发之后（握手成功后），给页面上的button指定一个事件，用来发送页面input当中的信息，服务端接收到信息打印出来，并组装成帧返回给日客户端，客户端再append到页面上。


服务端NodeJS代码：

```javascript
//握手成功之后就可以发送数据了
var crypto = require('crypto');
var WS = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
var server = require('net').createServer(function (socket) {
    var key;
    socket.on('data', function (msg) {
        if (!key) {
            // 第一次握手，响应头发送101状态码并更新协议
            //获取发送过来的Sec-WebSocket-key首部
            key = msg.toString().match(/Sec-WebSocket-Key: (.+)/)[1];
            key = crypto.createHash('sha1').update(key + WS).digest('base64');
            socket.write('HTTP/1.1 101 Switching Protocols\r\n');
            socket.write('Upgrade: WebSocket\r\n');
            socket.write('Connection: Upgrade\r\n');
            //将确认后的key发送回去
            socket.write('Sec-WebSocket-Accept: ' + key + '\r\n');
            //输出空行，结束Http头
            socket.write('\r\n');
        } else {
            // 按wss协议解码数据
            var msg = decodeData(msg);
            console.log(msg);
            //如果客户端发送的操作码为8,表示断开连接,关闭TCP连接并退出应用程序
            if(msg.Opcode == 8){
                socket.end();
                server.unref();
            }else{
            	// 按wss协议编码数据
                socket.write(encodeData({FIN:1,
                    Opcode:1,
                    PayloadData:"接受到的数据为"+msg.PayloadData}));
            }
        }
    });
});
    server.listen(8000,'localhost');
//按照websocket数据帧格式提取数据
function decodeData(e){
    var i = 0,j,s,frame = {
        //解析前两个字节的基本数据
        FIN:e[i] >> 7,
        Opcode : e[i++] & 15,
        Mask : e[i] >> 7,
        PayloadLength : e[i++]&0x7F
    };
    //处理特殊长度126和127
    if(frame.PayloadLength == 126)
        frame.length = (e[i++] << 8) + e[i++];
    if(frame.PayloadLength == 127)
        i+=4, //长度一般用四字节的整型，前四个字节通常为长整形留空的
            frame.length=(e[i++] << 24) + (e[i++] << 16) + (e[i++] << 8) + e[i++];
    //判断是否使用掩码
    if(frame.Mask){
        //获取掩码实体
        frame.MaskingKey = [e[i++], e[i++], e[i++], e[i++]];
        //对数据和掩码做异或运算
        for( j=0, s=[]; j<frame.PayloadLength; j++ )
            s.push(e[i+j]^frame.MaskingKey[j%4]);
    }else s = e.slice(i,frame.PayloadLength); //否则直接使用数据
    //数组转换成缓冲区来使用
    s = new Buffer(s);
    //如果有必要则把缓冲区转换成字符串来使用
    if(frame.Opcode == 1)s = s.toString();
    //设置上数据部分
    frame.PayloadData = s;
    //返回数据帧
    return frame;
}
//对发送数据进行编码
function encodeData(e){
    var s = [],o = new Buffer(e.PayloadData),l = o.length;
    //输入第一个字节
    s.push((e.FIN << 7) + e.Opcode);
    //输入第二个字节，判断它的长度并放入相应的后续长度消息
    //永远不使用掩码
    if(l < 126)  s.push(l);
    else if(l < 0x10000) s.push(126,(l&0xFF00) >> 2,l & 0xFF);
    else s.push(
            127, 0,0,0,0, //8字节数据，前4字节一般没用留空
                (l & 0xFF000000) >> 6,(l & 0xFF0000) >> 4,( l & 0xFF00) >> 2,l & 0xFF
        );
    //返回头部分和数据部分的合并缓冲区
    return Buffer.concat([new Buffer(s),o]);
}
```
结果如下：
浏览器:
![](/images/191754118102961.png)
服务器:
![](/images/191754557634399.png)

### 总结
　　在实际开发中，推荐选用成熟的实时通讯库（比如socket.io,sockjs,swoole,workerman），他们的原理就是将上面（还有一些其他的如基于Flash的push）的一些技术进行了在客户端和服务端的封装，然后给开发者一个统一调用的接口。在支持websocket的环境下使用websocket，在不支持它的时候启用上面所讲的一些hack技术。