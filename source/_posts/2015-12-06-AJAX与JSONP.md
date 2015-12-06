title: AJAX跨子域请求与JSONP的实现逻辑和方法
date: 2015-12-06
categories:
    - 学习与进步
    - 自用笔记
tags:
	- javascript
	- Ajax
	- JSONP
---

## AJAX同域名下子域跨域请求方法：

### iframe实现子域跨域请求. 首先页面上要有一个iframe

> 1）a.test.com去执行b.test.com的请求

```
document.domain = 'test.com'; //提升域名
document.getElementById('iframe').contentWindow.doAjax(function(data){
     console.log(data) //返回b.test.com的数据
});
```
<!--more-->

> 2）b.test.com内容所做的方法

```
document.domain = 'test.com'; //提升域名
function doAjax(callback){
     $.ajax('/test').done(function(data){
           callback(data)
     })
}
```

****
## JSONP跨域请求实现原理：

### 原理介绍
> 1) JSONP本质并不是ajax，只是执行了跨域的 js
> 2) html中 所有带src属性的标签都可以跨域 script, img , iframe
> 3) JSONP用src可以跨域的这个特性，来去向另外一个域发起一个script的请求，然后加载一段可以执行的脚本这段脚本里执行一段数据的信息，这样就可以让我们来跨域的处理数据。

### 例子：(简单做一下介绍，没有优化代码)。

```
function jsonp(url, data, callback) {
    var script = document.createElement('script'); //创建一个script标签
    document.body.appendChild(script);
    data = data || {};
    data.callback = 'cb' + new Date().getTime(); //随机写一个callback的名称
    var scripturl = url + '?'; //默认url上不带任何参数
    for (var i in data) {
        scripturl += '&' + (i + '=' + data[i]) //遍历参数放到url上以
    }
    script.url = scripturl; //设置url值
    script.onload = function () {
        document.body.removeChild(script) //回调之后 删除页面上的script
    }
}
jsonp('http://b.test.com', {a: 1, b: 2},function(data){
    console.log(data)
});
```
****

### JSONP的不足之处。
> 1) 只能通过get请求发起，因为src去调用其他域名下的地址是通过get的方法获取的。
> 2) 用JSONP要改变后台的返回方法。

****