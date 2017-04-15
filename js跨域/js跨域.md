

## JS跨域

#### jsonp

客户端先注册一个回调函数，然后通过生成一个script标签，src为：请求资源的地址＋获取函数的字段名＋回调函数名称（需要与服务器端约定好），服务器端接收到script请求后，动态生成一段传入参数的回调函数执行代码，然后返回给客户端。客户端获取到script资源后，执行其中的js代码。

客户端示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script type="text/javascript" src="jquery-1.12.3.js"></script>
  <script type="text/javascript">
    var localHandler = function(data){
      alert('我是本地函数，可以被跨域的test.js文件调用，远程js带来的数据是：' + data.result);
    };
  </script>
  <script type="text/javascript" src="http://localhost:8007/test.js"></script>
</head>
<body>
</body>
</html>
```

服务器端动态生成的代码

```js
localHandler({"result":"我是远程js带来的数据"})
```

还可以用jquery来进行jsonp跨域

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script type="text/javascript" src="jquery-1.12.3.js"></script>
  <script type="text/javascript">
    $(function() {
      $.ajax({
        type: "get",
        async: false,
        url: "http://localhost:8007/test.js",
        dataType: "jsonp",
        jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
        jsonpCallback:"localHandler",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
        success: function(data){
          alert('我是本地函数，可以被跨域的test.js文件调用，远程js带来的数据是：' + data.result);
         },
        error: function(){
          alert('fail');
        }
      });      
    })
  </script>
</head>
<body>
</body>
</html>
```



#### document.domain+iframe

如果A页面和B页面具有相同的父域名，那么将两个页面的document.domain设置为相同的父域名，就可以实现不同子域名之间的跨域通讯。

主页面代码:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <script>
    console.log(document.domain);
    document.domain = '123.test.com';
    function onLoad() {
      var iframe =document.getElementById('iframe');
      var iframeWindow = iframe.contentWindow; // 这里可以获取 iframe 里面 window 对象并且能得到方法和属性
      var doc = iframeWindow.document; // 获取到
      console.log('doc', doc);
    }
  </script>
  <iframe id="iframe" src="http://123.test.com:8009/iframe.html" onload="onLoad()"</iframe>
</body>
</html>
```

iframe页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <script>
    document.domain = '123.test.com';
  </script>
</body>
</html>
```

#### location.hash+iframe

url的任何改变都会重新加载一个新页面除了hash，hash的改变不会导致页面刷新。因此可以通过主页面传递参数改变iframe页面的hash值，iframe不断监听自身hash值，一旦发生改变，获取hash值，同时改变主页面的hash值来向主页面传递信息，实现双向的跨域通讯。

主页面代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <p></p>
  <button>发送消息</button>  
  <iframe id="ifr" src="http://123.test.com:8009/iframe.html"></iframe>  
  <script>  
  var target = "http://123.test.com:8009/iframe.html";  
  function sendMsg(msg){  
    var data = {msg:msg};
    var src = target + "#" + JSON.stringify(data);
    document.getElementById('ifr').src = src;
  }
  document.querySelector('button').onclick = function (){  
    sendMsg("时间：" + (new Date()));
  }
  var oldHash = "";  
  checkIndexMessage = function(){  
    var newHash = window.location.hash;
    if(newHash.length > 1){
        newHash = newHash.substring(1, newHash.length);
        if(newHash != oldHash){
          document.querySelector('p').innerHTML = newHash;
        }
      }
  }
  window.setInterval(checkIndexMessage, 1000);
  </script>    
</body>
</html>
```

iframe页面代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <div></div>
  <script>
    var oldHash = "";  
    var target = "http://123.test.com:8007/hash.html";  
    checkMessage = function(){  
      var newHash = window.location.hash;
      if(newHash.length > 1){
          newHash = newHash.substring(1, newHash.length);
          if(newHash != oldHash){
            oldHash = newHash;
            var msgs = JSON.parse(newHash);
            var msg = msgs.msg;
            sendMessage("Hello");
            document.querySelector('div').innerHTML = msg;
          }
        }
    }
    window.setInterval(checkMessage, 1000);  
    function sendMessage(msg){
      parent.location.href = target + "#" + msg;
    }
  </script>
</body>
</html>
```



#### window.name+iframe

#### HTML5 postMessage