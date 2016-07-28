from wuxiaolan91
# 迅速用node搭建一个服务器
## 先来看看我最终的代码:
1. 先来看看html是怎样的.

```html
<html>
<head>
    <meta charset="UTF-8">
    <title>预定</title>
    <!-- 新 Bootstrap 核心 CSS 文件 -->
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.css">
    <link rel="stylesheet" href="css/common.css">
    <link rel="stylesheet" href="css/index.css">
</head>
<body>
  <ul id="teacherList" class="list-group">
  </ul>
  <script id="teacherTmpl" type="text/html">
     <li class="list-group-item">
          <div class="name">
              apple
          </div>
          <div>
              <img src="https://gd4.alicdn.com/bao/uploaded/i4/90189776/TB2EH1SqXXXXXbaXXXXXXXXXXXX_!!90189776.jpg_400x400.jpg_.webp" alt="">
          </div>
          <p class="desc">
              apple老师是一位教学非常热情且耐心的老师, 课堂掉东西能够很好, 教学非常仔细,
              是一位很受学生欢迎的老师
          </p>
          <div class="book">
              <button type="button" class="btn btn-primary">预定</button>
          </div>
      </li>
  </script>
    <script src="http://cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
    <script src="http://cdn.bootcss.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
    <script src="js/lib/template.js"></script>
    <script>
        $.getJSON('json/teacherList.json')
        .done( function(data) {
            
            $('#teacherList').html(template('teacherTmpl', data));
        })

        $("#teacherList").on("click", "li", function () {
            location.assign('teacherDetail.html');
        })
    </script>
</body>
</html>
```

2. node.js文件(在这里创建http服务器)

```javascript
var http = require('http');
var fs = require('fs');
var url = require('url');

http.createServer( function(request, response) {
    debugger;
    console.log('111')
    console.log(request.url);
    // 解析请求, 输出文件名
    var pathname = url.parse(request.url).pathname;
    // from fileSystem read file content
    fs.readFile(pathname.substr(1), function (err, data) {

        if (err) {
            console.log(err);
            response.writeHead(404, {'Content-Type': 'text/html'});
        } else {
            response.writeHead(200, {'Content-type': 'text/html'});

            // response file content
            response.write(data.toString());
        }
        response.end();
    })

}).listen(8081);
console.log('Server running at http://127.0.0.1:8081/');
```

## 2. 好, 接下来我们来具体拆分代码. 
1. 先引入需要用的module

```javascript
var http = require('http'); // 创建web服务器需要引入的moduel
var fs = require('fs'); // 操作文件系统
var url = require('url');
```

2. 创建一个服务器
	http.createServer方法就能在本地创建一个web服务器,
	它的回调是一个function, 这个function里会传入两个参数request, response,
	当然, 你也可以叫别的名字
    
```
http.createServer( function (request, response) {
	
})
```

ok,  创建服务, 然后req就能拿到请求相关, res就能给请求设置返回了.
3. 解析请求, 拿到文件名
	比如你请求一个index.html文件, 如果这个html文件它引用了十个文件, 五个外部文件, 五个内部文件.
那么第2点提到的回调函数就会被执行6遍.<br/>
外部的文件不会导致执行回调函数, 而内部的会. 包括对当前index.html文件自己.
比如

```javascript
	http.createServer( function(reqest, response) {
		console.log('111');
		console.log(req.url);
	});
```

最终控制台打印出来了6个111.
来, 看看输出
```
111
/index.html
111
/css/common.css
111
/css/index.css
111
/js/lib/template.js
111
/json/teacherList.json
111
/json/teacherList.json
```
说实在的, 在看到这里我自己是有点惊讶的,<br/>
 因为本来之前我一直是以为我请求一个html文件, 那createServer的回调函数应该只会执行一遍,拿到的文件地址就是index.html嘛,<br/> 没有想到这里居然是每个本地文件都会执行回调函数, 这里注意, 引用的外部资源是不会进入到回调函数里的.
4. 拿到文件名
```
var pathname = url.parse(request.url).pathname;
```
5. 读取文件
```
fs.readFile(pathname.substr(1), function (err, data) {
	if (err) {
            console.log(err);
            response.writeHead(404, {'Content-Type': 'text/html'});
        } else {
            response.writeHead(200, {'Content-type': 'text/html'});

            // response file content
            response.write(data.toString());
        }})
            response.end();
   })
 ```
 这里可以看到, 对错误进行了兼容判断. 如果是错误, 就返回404,
 否则就返回200.
 然后把之前读取到的文件写入response里.
 ```
 response.write(data.toString())
 ```
 这一点很重要. 如果你没有写入读取到的内容.
 那你打开http服务器自己定义的地址后,看到的页面就是空白的. 没有内容.
 这样也就达不到我们当初想要抛弃apache/nginx这些web服务器,
 而用基于本地页面搭建node服务器的初衷了.
 6. 结束响应
 ```
 response.end();
 ```
 7.  开端口
http.createServer方法后面要跟一个listen方法, 来, 咱们加一下.
 ```
 http.createServer( function(request, response) {
listen(8081);
```
8. 享受胜利的果实.
在当前项目的cli里输入
```
node node.js
```
现在我们输入地址http://localhost:8081/index.html
就会看到酱紫啦
![](http://oawvy5uzs.bkt.clouddn.com/16-7-28/63416377.jpg)
对了. 这里最终我的代码. 我只放了我的html文件,  和node.js文件, 没有放其他的.
所以大家可能和我最终预览的效果长的不一样
这个嘛我的templatejs就是腾讯的js引擎: artTeamplate
还不错, 才6k大小. 比起jQuery的tmpl插件来说, 真的要轻量太多.
好了. 话不多说, 去试试吧.




