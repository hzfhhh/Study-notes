

## JS跨域

注：参考文章：[【原创】说说JSON和JSONP，也许你会豁然开朗，含jQuery用例](http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)、[前端跨域的整理](https://qiutc.me/post/cross-domain-collections.html)、[试试跨域通信 - 利用iframe](http://liuwanlin.info/shi-shi-kua-yu-tong-xin-iframe/)

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

window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个 `window.name` ，每个页面对 `window.name` 都有读写的权限，`window.name` 是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。因此，我们可以在一个页面中创建一个iframe页面，通过这个iframe去获取指定地址的数据保存到`window.name`中，当iframe获取到指定数据后，根据同源策略，将iframe的src设置成跟主页面在同一个域后，主页面就可以通过`window.name`获取到指定的数据。

主页面代码：

```html
// a.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script>
	function getData() {
		var iframe =document.getElementById('iframe');
		iframe.onload = function() {
			var data = iframe.contentWindow.name; // 得到
		}
		iframe.src = 'b.html';  // 这里b和a同源
	}
  </script>
</head>
<body>
  <iframe src="https://www.qiutc.com/data.html" style="display:none" onload="getData()"</iframe>
</body>
</html>
```

iframe页面代码：

```html
<script>
window.name = '我是被期望得到的数据';
</script>
```



#### HTML5 postMessage

`window.postMessage(message, targetOrigin)` 方法是html5新引进的特性，可以使用它来向其它的window对象发送消息，无论这个window对象是属于同源或不同源。兼容性：
![cross-domain-postmessage](https://qiutc.me/img/section-cross-domain-postmessage.png)

`window.postMessage(message, targetOrigin)`接收两个参数

`message`要传递的数据，html5规范中提到该参数可以是JavaScript的任意基本类型或可复制的对象，然而并不是所有浏览器都做到了这点儿，部分浏览器只能处理字符串参数，所以我们在传递参数的时候需要使用JSON.stringify()方法对对象参数序列化，在低版本IE中引用json2.js可以实现类似效果。

`targetOrigin`用来限定接收消息的那个window对象所在的域，如果不想限定域，可以使用通配符 * 。

主页面代码：

```html
// 主页面  blog.qiutc.com
<script>
function onLoad() {
	var iframe =document.getElementById('iframe');
	var iframeWindow = iframe.contentWindow;
	iframeWindow.postMessage("I'm message from main page.");
}
</script>
<iframe src="https://www.qiutc.me/b.html" onload="onLoad()"</iframe>
```

iframe页面代码：

```html
// b 页面
<script>
var onmessage = function(e) {
	e = e || event;
	console.log(e.data);
}
window.addEventListener('message', onmessage, false)
</script>
```

