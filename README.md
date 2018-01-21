# 解决浏览器跨域问题的 Demo

## 一、无跨域问题的页面之间的通信
- **iframe.html**

```html
<div id="a">我是 parent 的内容</div>
<div id="b">点击获取 iframe 页面的内容</div>

<iframe id="iframe" src="iframe2.html"></iframe>
<script type="text/javascript">
   document.getElementById('b').onclick = function(){
        var iframeWin = document.getElementById('iframe').contentWindow;
		document.getElementById('b').innerHTML = iframeWin.document.getElementById('a').innerHTML;        
    }
</script>
```

- **iframe2.html**

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

## 二、document.domain
- **iframe.html**

```html

```

- **iframe.html**

```html

```

## 了解更多跨域知识

[跨域知识总结-前端]（http://alvinwp.com/seo/1848）


