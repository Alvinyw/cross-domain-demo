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

> 仅高级浏览器 Internet Explorer 8+, chrome，Firefox , Opera 和 Safari 支持这个功能。

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

**原理：&lt; script> 和 &lt; img> 等带有 src 属性的元素可以跨域获取资源。**

[JSONP](https://baike.baidu.com/item/jsonp/493658?fr=aladdin)(JSON with Padding)是 JSON 的一种"使用模式"，可用于解决主流浏览器的跨域数据访问的问题。


- **iframe.html**

```html

```

- **iframe.html**

```html

```

## 了解更多跨域知识

[跨域知识总结-前端]（http://alvinwp.com/seo/1848）


