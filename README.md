# 解决浏览器跨域问题的 Demo

## 无跨域问题的页面之间的通信
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

- *iframe2.html*

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



## 了解更多跨域知识

[跨域知识总结-前端]（http://alvinwp.com/seo/1848）


