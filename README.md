# cross-domain
## 关于跨域问题的学习笔记

在以前，前端和后端混杂在一起， 比如JavaScript直接调用同系统里面的一个Httphandler，就不存在跨域的问题，但是随着现代的这种多种客户端的流行，比如一个应用通常会有Web端，App端，以及WebApp端，各种客户端通常会使用同一套的后台处理逻辑，即API， 前后端分离的开发策略流行起来，前端只关注展现，通常使用JavaScript，后端处理逻辑和数据通常使用WebService来提供json数据。一般的前端页面和后端的WebService API通常部署在不同的服务器或者域名上。这样，通过ajax请求WebService的时候，就会出现同源策略的问题。

### 同源策略
如果两个页面拥有相同的协议（protocol），端口（如果指定），和主机，那么这两个页面就属于同一个源（origin）。

### 产生跨域问题的原因
跨域问题是浏览器同源策略限制，当前域名的js只能读取同域下的窗口属性。

### 跨域问题产生的场景
当要在在页面中使用js获取其他网站的数据时，就会产生跨域问题，比如在网站中使用ajax请求其他网站的天气、快递或者其他数据接口时以及hybrid app中请求数据，浏览器就会提示以下错误。这种场景下就要解决js的跨域问题。

### 哪些情况会产生跨域问题
一个网站的网址组成包括协议名，子域名，主域名，端口号。比如 https://github.com/ ，其中https是协议名，www是子域名，github是主域名，端口号是80，当你在页面中从一个url请求数据时，如果这个url的协议名、子域名、主域名、端口号任意一个有一个不同，就会产生跨域问题。
即使是在 http://localhost:80/ 页面请求 http://127.0.0.1:80/ 也会有跨域问题。

### 解决跨域问题

解决跨域问题有以下几种方式，这里只简单阐述几种常用的技术

* 使用jsonp

* 通过iframe

* 通过XHR2

* 服务端代理

* 服务端设置Request Header头中Access-Control-Allow-Origin为指定可获取数据的域名

* 其他简单的跨域技术,例如图像Ping, flash, CORS


#### 1.JSONP

原理

jsonp由两部分组成：回调函数和数据。回调函数就是当请求返回时在页面中调用的函数。数据就是传入回调函数中的json数据。

jsonp解决跨域问题的原理是，浏览器的script标签是不受同源策略限制(你可以在你的网页中设置script的src属性问cdn服务器中静态文件的路径)。那么就可以使用script标签从服务器获取数据，请求时添加一个参数为callbakc=?，?号时你要执行的回调方法。
简单来说就是jsonp在前端声明数据，利用script传递给后端，后端调用数据，把传过来的参数当作实参并构造函数调用。
jsonp相对简单，但只支持GET方式调用。

##### 实现方法

jsonp.html

```
var url = "http://localhost:1335/ajax/jsonp.php?name=太阳&sex=女&callbackname=jsonp_callback"; 

//访问localhost:1335下的jsonp.php

var scriptTag = document.createElement("script"); //创建一个script标签

scriptTag.setAttribute("src",url); //设置script的src属性

document.body.appendChild(scriptTag); //将script标签添加到body中

//回调函数

var jsonp_callback = function(resultObj){

document.getElementById("box").innerHTML = resultObj.name+":"+resultObj.sex;

}
```

这样通过动态创建script标签就可以加载其它域的js文件，然后通过本页面就可以调用加载后js文件的函数，这样做的缺陷就是不能加载其它域的文档，只能是js文件，jsonp便是通过这种方式实现的，jsonp通过向其它域传入一个callback参数，通过其他域的后台将callback参数值和json串包装成javascript函数返回，因为是通过script标签发出的请求，浏览器会将返回来的字符串按照javascript进行解析执行，实现了域与域之间的数据传输。

jsonp.php

```
$name = $_GET["name"];

$sex = $_GET["sex"];

$callbackname = $_GET["callbackname"]; //回调函数名称

echo "$callbackname({name:'$name',sex:'$sex'})";
```


jquery中对jsonp的支持也是基于此方案。 

以jQuery2.1.3的ajax方法为例

```
$.ajax({

    url:"",
    
    dataType:"jsonp",
    
    data:{
    
        params:""
        
        }
        
}).done(function(data){

    //dosomething..
    
})
```

仅仅是客户端使用jsonp请求数据是不行的，因为jsonp的请求是放在script标签中的，和普通请求不同的地方在于，它请求到的是一段js代码，如果服务端返回了json字符串，那么浏览器就会报错。所以jsonp返回数据需要服务端做一些处理。

##### 服务端返回数据处理

上面说了jsonp的原理是利用script标签来解决跨域，但是script标签是用来获取js代码的，那么我们怎么获取到请求的数据呢。

这就需要服务端做一些判断，当参数中带有callback属性时，返回的type要为application/javascript，把数据作为callback的参数执行。下面是jsonp返回的数据的格式示例

```
/**/ typeof jQuery21307270454438403249_1428044213638 === 'function' && jQuery21307270454438403249_1428044213638({"code":1,"msg":"success","data":{"test":"test"}});
```


#### 2.iframe

基于iframe实现的跨域要求两个域具有aa.xx.com,bb.xx.com这种特点，也就是两个页面必须属于一个基础域（例如都是xxx.com，或是xxx.com.cn），使用同一协议（例如都是 http）和同一端口（例如都是80），这样在两个页面中同时添加document.domain，就可以实现父页面调用子页面的函数,实现js跨域访问。

iframe会存在以下的问题：

1、安全性，当一个站点被攻击后，另一个站点会引起安全漏洞。

2、如果一个页面中引入多个iframe，要想能够操作所有iframe，必须都得设置相同domain。


#### 3.XHR2
高版本浏览器支持html5的话，还可以使用XHR2支持跨域通信的新特性。如果你是移动端开发，可以选择使用XHR2。

这需要在远程服务器端添加如下代码：

```
header('Access-Control-Allow-Origin:*'); //*代表可访问的地址，可以设置指定域名

header('Access-Control-Allow-Methods:POST,GET');
```

这样在客户端使用常规的AJAX代码即可。


#### 4.代理
这种方式可以解决所有跨域问题，也就是将后台作为代理，每次对其它域的请求转交给本域的后台，本域的后台通过模拟http请求去访问其它域，再将返回的结果返回给前台，这样做的好处是，无论访问的是文档，还是js文件都可以实现跨域。
代理实现最麻烦，但使用最广泛，任何支持AJAX的浏览器都可以使用这种方式。

