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
- **http://localhost/domain.html**

**原理：给通信的两个页面设置相同的 document.domain**

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

## 二、document.domain
- **iframe.html**

```html

```

- **iframe.html**

```html

```

## 二、document.domain
- **iframe.html**

```html

```

- **iframe.html**

```html

```

## 二、document.domain
- **iframe.html**

```html

```

- **iframe.html**

```html

```

## 了解更多跨域知识

[跨域知识总结-前端]（http://alvinwp.com/seo/1848）


