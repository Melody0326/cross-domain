# cross-domain
自己总结的关于js跨域问题的笔记

###产生跨域问题的原因
跨域问题是浏览器同源策略限制，当前域名的js只能读取同域下的窗口属性。

###跨域问题产生的场景
当要在页面中使用js获取其他网站的数据时，就会产生跨域问题，比如在网站中使用ajax请求其他网站的天气、快递或者其他数据接口时以及hybrid app中请求数据，浏览器就会提示以下错误。这种场景下就要解决js的跨域问题。

###哪些情况会产生跨域问题
一个网站的网址组成包括协议名，子域名，主域名，端口号。比如 https://github.com/ ，其中https是协议名，www是子域名，github是主域名，端口号是80，当你在页面中从一个url请求数据时，如果这个url的协议名、子域名、主域名、端口号任意一个有一个不同，就会产生跨域问题。
即使是在 http://localhost:80/ 页面请求 http://127.0.0.1:80/ 也会有跨域问题

###解决跨域问题
解决跨域问题有以下一种方式

*使用jsonp
*服务端代理
*服务端设置Request Header头中Access-Control-Allow-Origin为指定可获取数据的域名

jsonp的解决方式
json≠jsonp

原理

jsonp解决跨域问题的原理是，浏览器的script标签是不受同源策略限制(你可以在你的网页中设置script的src属性问cdn服务器中静态文件的路径)。那么就可以使用script标签从服务器获取数据，请求时添加一个参数为callbakc=?，?号时你要执行的回调方法。

###前端实现
以jQuery2.1.3的ajax方法为例
$.ajax({
    url:"",
    dataType:"jsonp",
    data:{
        params:""
        }
}).done(function(data){
    //dosomething..
})

仅仅是客户端使用jsonp请求数据是不行的，因为jsonp的请求是放在script标签中的，和普通请求不同的地方在于，它请求到的是一段js代码，如果服务端返回了json字符串，那么浏览器就会报错。所以jsonp返回数据需要服务端做一些处理。

###服务端返回数据处理

上面说了jsonp的原理是利用script标签来解决跨域，但是script标签是用来获取js代码的，那么我们怎么获取到请求的数据呢。

这就需要服务端做一些判断，当参数中带有callback属性时，返回的type要为application/javascript,把数据作为callback的参数执行。下面是jsonp返回的数据的格式示例
/**/ typeof jQuery21307270454438403249_1428044213638 === 'function' && jQuery21307270454438403249_1428044213638({"code":1,"msg":"success","data":{"test":"test"}});

###浏览器支持

Access-Control-Allow-Origin是html5新增的一项标准，IE10以下是不支持的，所以如果产品面向的是PC端，就要使用服务端代理或jsonp。
