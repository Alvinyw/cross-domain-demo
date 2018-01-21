# 解决浏览器跨域问题的 Demo

## 一、无跨域问题的页面之间的通信
- **http://localhost/iframe.html**

```html
<div id="a">我是 parent 的内容</div>
<div id="b">点击获取 iframe 页面的内容</div>

<iframe id="iframe" src="http://localhost/iframe2.html"></iframe>
<script type="text/javascript">
   document.getElementById('b').onclick = function(){
	var iframeWin = document.getElementById('iframe').contentWindow;
	document.getElementById('b').innerHTML = iframeWin.document.getElementById('a').innerHTML;        
    }
</script>
```

- **http://localhost/iframe2.html**

```html
<div id="a">我是 iframe 的内容</div>
<div id="b">点击获取 parent 页面的内容</div>

<script>
document.getElementById('b').onclick = function(){
	document.getElementById('b').innerHTML = window.parent.document.getElementById('a').innerHTML;        
}
</script>
```

代码如上的两个页面间可以正常通信。

## 二、document.domain

**原理：给通信的两个页面设置相同的 document.domain**

- **http://localhost/domain.html**

```html
<div id="c">我是 parent 页面里的内容</div>
<div id="b">点击这里获取 iframe 页面的内容</div>
<iframe id="iframe" src="http://localhost/:86/domain.html"></iframe>
<script type="text/javascript">
	document.domain = 'http://localhost';

	var iframe = document.getElementById('iframe');
	var win = iframe.contentWindow;

	document.getElementById('b').onclick = function(){
		document.getElementById('b').innerHTML = win.document.getElementById('a').innerHTML
	}
</script>
```

- **http://localhost/:86/domain.html**

```html
<div id="a">我是 iframe 页面里的内容</div>
<div id="d">点击这里获取 parent 的内容</div>
<script type="text/javascript">
    document.domain = 'http://localhost';
	
	function aa(){
		alert('aa');
	}
	document.getElementById('d').onclick = function(){
		document.getElementById('d').innerHTML = window.parent.document.getElementById('c').innerHTML;
	}
</script>
```
> document.domain 的设置是有限制的，我们只能把 document.domain 设置成自身或更高一级的父域，且主域必须相同; 即修改 document.domain 的方法只适用于**主域相同，子域不同或是端口号不同**的框架之间的交互。

## 三、location.hash

**原理：parent 页面能改变 iframe 页面的 url，iframe 页面也能改变 parent 的 url；而 url 的改变不会引起页面刷新和 http 请求，但是能够产生浏览器缓存**

- **http://localhost/hash.html**

```html
<ul>
    <li>Click to send the first message</li>
    <li>Click to send the second message</li>
    <li>Click to send the thrid message</li>
</ul>
<iframe id="myIFrame" src="https://alvinyw.github.io/Blog/crossDomain/hash.html"></iframe>
<div id="b">接受 iframe 页面发送的消息</div>

<script>
//点击发送消息
$("li").each(function(index, element) {
	var _this = $(this);
    _this.click(function(){
		document.getElementById('myIFrame').src = 'https://alvinyw.github.io/Blog/crossDomain/hash.html' + '#' + _this.html();
	});
});

//监听 hash 变化
window.onhashchange = checkMessage;
function checkMessage() {
  var message = window.location.hash;
  $("#b").html(message);
}
</script>
```

- **https://alvinyw.github.io/Blog/crossDomain/hash.html**

```html
<div id="a">接受 parent 页面发送的消息</div>

<script>
//监听 hash 变化
window.onhashchange = checkMessage;

function checkMessage() {
  var message = window.location.hash;
  document.getElementById('a').innerHTML = message;
  
  switch(message)
	{
	case '#Click to send the first message':
	  parent.location.href = 'http://localhost/hash.html#received-first';
	  break;
	case '#Click to send the second message':
	  parent.location.href = 'http://localhost/hash.html#received-second';
	  break;
	case '#Click to send the thrid message':
	  parent.location.href = 'http://localhost/hash.html#received-thrid';
	  break;
	default:
	  parent.location.href = 'http://localhost/hash.html#received';
	}  
}
</script>
```
> 缺点：数据暴露在了 url 里；不适合发送大数据量的通信。

## 四、HTML5 的 postMessage

**说明：HTML5 为了解决跨域问题，引入了一个全新的 API：跨文档通信 API（Cross-document messaging）。这个 API 为 window 对象新增了一个 window.postMessage 方法，允许跨窗口通信，不论这两个窗口是否同源。**

- **http://localhost/postMessage.html**

```html
<div id="a">我是主页面</div>
<div id="c">iframe 内嵌页面返回的信息会显示在这里</div>
<iframe id="childPage" src="https://alvinyw.github.io/Blog/crossDomain/postMessage.html"></iframe>
<input id="btnSubmit" onclick="sendData()" type="button" value="给 iframe 页面发送文本：hello world!"/>

<script>
//发消息
function sendData(){
    var ifr = document.getElementById('childPage');  
    var targetOrigin = "https://alvinyw.github.io";  
    ifr.contentWindow.postMessage('hello world!', targetOrigin);  
};

//收消息
var onmessage = function (event) {  
  var data = event.data;//消息内容  
  var origin = event.origin;//消息来源地址  
  var source = event.source;//源 Window 对象
  if(origin=="https://alvinyw.github.io"){  
	document.getElementById("c").innerHTML = data;
  }  
};  

if (typeof window.addEventListener != 'undefined') { 

  window.addEventListener('message', onmessage, false);  
  
} else if (typeof window.attachEvent != 'undefined') { 
 
  //for ie 
  window.attachEvent('onmessage', onmessage);  
  
}  
</script>
```

- **https://alvinyw.github.io/Blog/crossDomain/postMessage.html**

```html
<div>我是 iframe 内嵌页面</div>
<div id="b">主页面发过来的信息会显示在这里</div>

<script>
var onmessage = function (event) {  
  var data = event.data;//消息  
  var origin = event.origin;//消息来源地址  
  var source = event.source;//源 Window 对象  
  if(origin=="http://localhost/"){  
	document.getElementById("b").innerHTML = data;
	
	//发消息 
    source.postMessage('我收到信息啦！', origin);  
  }  
};  

if (typeof window.addEventListener != 'undefined') {  
  window.addEventListener('message', onmessage, false);  
} else if (typeof window.attachEvent != 'undefined') {  
  //for ie  
  window.attachEvent('onmessage', onmessage);  
}  
</script>
```
### postMessage 的参数：
1. **otherWindow:** 指目标窗口，也就是给哪个 window 发消息，是 window.frames 属性的成员或者由 window.open 方法创建的窗口;
2. **message:** 是要发送的消息，类型为 String、Object (IE8、9 不支持);
3. **targetOrigin:** 是限定消息接收范围，不限制请使用 “*”。

> 缺点：仅高级浏览器 Internet Explorer 8+, chrome，Firefox , Opera 和 Safari 支持这个功能。

## 五、window.name

**原理：在一个窗口 (window) 的生命周期内,窗口载入的所有的页面都是共享一个 window.name 的，每个页面对 window.name 都有读写的权限，window.name 是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。**

- **http://localhost/name.html**

```html
<a class="link1" href="https://alvinyw.github.io/Blog/crossDomain/name.html">https://alvinyw.github.io/Blog/crossDomain/name.html</a>

<a class="link2" target="_blank" href="https://alvinyw.github.io/Blog/crossDomain/name.html">https://alvinyw.github.io/Blog/crossDomain/name.html</a>

<script>
	window.name = "Information from the parent window";
</script>
```

- **https://alvinyw.github.io/Blog/crossDomain/name.html**

```html
<script>
	alert("window.name is: "+ window.name);
</script>
```
> 说明：只要跟 http://localhost/name.html 页面在同一个 Tab 窗口打开的页面，无论是否跨域都会共享同一个 window.name。

## 六、jsonp

**原理：&lt;script> 和 &lt;img> 等带有 src 属性的元素可以跨域获取资源。**

[JSONP](https://baike.baidu.com/item/jsonp/493658?fr=aladdin)(JSON with Padding)是 JSON 的一种"使用模式"，可用于解决主流浏览器的跨域数据访问的问题。

网页通过添加一个 &lt;script> 元素，向服务器请求 [JSON](https://baike.baidu.com/item/JSON/2462549?fr=aladdin) 数据，这种做法不受同源政策限制；利用 &lt;script> 元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的能执行的 JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。

上面说的这几种都是双向通信的，即两个 iframe，页面与 iframe 或是页面与页面之间的，下面说几种单项跨域的（一般用来获取数据），因为通过 script 标签引入的 js 是不受同源策略的限制的。所以我们可以通过 script 标签引入一个 js 或者是一个其他后缀形式（如 php，jsp 等）的文件，**此文件返回一个 js 函数的调用**。

- **http://localhost/JSONP.html**

```html
<ul>
    <li>Name: <span id="a"></span></li>
    <li>Age: <span id="b"></span></li>
    <li>Weight: <span id="c"></span></li>
</ul>
<script type="text/javascript">
    function dosomething(jsondata){
    	document.getElementById('a').innerHTML = jsondata.Nmae;
		document.getElementById('b').innerHTML = jsondata.Age;
		document.getElementById('c').innerHTML = jsondata.Weight;
    }
</script> 
<script src="https://alvinyw.github.io/Blog/crossDomain/jsonp.json?callback=dosomething"></script>
```

- **https://alvinyw.github.io/Blog/crossDomain/jsonp.json**

```javascript
fun 
dosomething(
{
  "Nmae": "Alvin",
  "Age": "xx",
  "Weight": "8.8.8.8"
}
);
```

- **https://alvinyw.github.io/Blog/crossDomain/jsonp.php**

```php
<?php

$callback = $_GET['callback'];//得到回调函数名

$data = '{
  "Nmae": "Alvin",
  "Age": "xx",
  "Weight": "8.8.8.8"
}';//要返回的数据

echo $callback.'('.json_encode($data).')';//输出

?>
```
### JSONP 的优缺点: 
1. JSONP 的优点是：它不像 XMLHttpRequest 对象实现的 Ajax 请求那样受到同源策略的限制；它的兼容性更好，在更加古老的浏览器中都可以运行，不需要 XMLHttpRequest 或 ActiveX 的支持；并且在请求完毕后可以通过调用 callback 的方式回传结果。
2. JSONP 的缺点则是：它只支持 GET 请求而不支持 POST 等其它类型的 HTTP 请求；它只支持跨域 HTTP 请求这种情况，不能解决不同域的两个页面之间如何进行 JavaScript 调用的问题。

## 七、CORS

CORS（Cross-Origin Resource Sharing）跨域资源共享，定义了必须在访问跨域资源时，浏览器与服务器应该如何沟通。**CORS 背后的基本思想就是使用自定义的 HTTP 头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功还是失败**。目前，所有浏览器都支持该功能，IE 浏览器不能低于 IE10。整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。

对于开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信。

需要设置的几个 HTTP 头信息如下：

```html
Access-Control-Allow-Origin: http://alvinwp.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
```

服务器的设置可参考如下代码：
```html
location / {
     if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
     }
 
     if ($request_method = 'POST') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
     }
 
     if ($request_method = 'GET') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
     }
}
```
### CORS 和 JSONP 对比:
1. JSONP 只能实现 GET 请求，而 CORS 支持所有类型的 HTTP 请求。
2. 使用 CORS，开发者可以使用普通的 XMLHttpRequest 发起请求和获得数据，比起 JSONP 有更好的错误处理。
3. JSONP 主要被老的浏览器支持，它们往往不支持 CORS，而绝大多数现代浏览器都已经支持了 CORS）。

## 八、了解更多跨域知识

[跨域知识总结-前端](http://alvinwp.com/seo/1848)


