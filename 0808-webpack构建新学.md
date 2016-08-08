# 0808学习命令行和webpack.md
## 新学命令行
*  今天发现touch filename 居然在windows下也可以创建新文件. 
*  vim filename也是可以进入编辑模式的.
只不过vim这个我编辑了后不知道怎么及时保存退出.
## webpack. 
重构wecash项目的构建工具dw打算从gulp换成webpack了.
所以我又重新看了下
1. webpack里的export.modules堪称是其最基础最重要的方法.
因为这个对象决定了你是构建代码是什么, 输出到哪里去
```
module.exports = {
	entry: '构建文件/目录', // 这个可以通过js动态取得
	filename: '构建完后的输出' // xx[hash].js // 便可以根据源文件动态输出文件名
}
```

2.  webpack命令的修饰命令
```
webpack --display-module
```
 
因为默认是不显示构建的时候不属于你自己的模块的. 但是这个命令却可以显示出来
```
webpack --watch
```

这个很有用, 如果你的源文件改变了, 它自动帮你编译.不过注意, *你还是需要手动更新浏览器的*,  因为它并没有替你把浏览器也给刷新了. 尽管它可以做到.
3. webpack的laoder的每一个test原来都要以$为结尾.
今天我看例子的时候别人是这么写的
```
module.exports = {
 entry: './src',
 output: {
  path: './dest',
  filename: 'bundle.js'
 },
  module: {
   loaders: [
     {
       test: /\.js$/,
       loader: 'babel'
     }
   ]
  }
}
```
这里看到.我是编译我src目录下的js文件, loader用的是babel, 我的src目录下只有一个文件, 就是index.js
内容是酱紫滴
```
import $ from 'jquery';
$('body').html('现在是1949时刻')
```
我试着运行webpack, 但是一直报错, 我左思右想, 觉得不对呀. 看不出来哪里错.
好, 咱不纠结, 请同事来帮忙看了下,
原来是
test需要以$结尾.
改了一下, 终于好了<br>
呜呜~  心痛.
其实仔细看官网的例子是能看出来跟我的不同的
```
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```

可以看到, 官网的例子里css后跟了个$符号, 意思是以.css文件结尾的文件才应用这个loader.
而我呢. 因为一艘到, 看到别人讲的例子. 就毫不怀疑是例子本身就错了.
明明也看了官网, 却并没有仔细辨明区别.
一看到官网的loader是针对css的, 就没有仔细看.
所以才导致这个问题在我这里解决不了.

总结:  不要!  完全!  相信!  例子.!
遇到问题还是仔细看官网吧.
官网才是最可靠的人呀.
[贴一下错误的例子的地址](https://cnodejs.org/topic/57528759adc77ac170409e79)
