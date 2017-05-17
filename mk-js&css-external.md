## 使用外联的Javascript和CSS
我们应该使用外联拓展Javascript和Css还是在页面中内联他们?我们将会发现，使用拓展文件通常会更好。

#### 内联VS外联
让我们开始来权衡下内联和外联这两种方式。

#### 在原始条件下，内联比较快
我制作了两个例子来说明内联JS和CSS比外联引用他们有更快的响应。

内联JS和CSS http://stevesouders.com/hpws/inlined.php
外联JS和CSS http://stevesouders.com/hpws/external.php

内联例子包含一个87K大的HTML文档，所有的JS和CSS都在页面里面。外联的例子包含一个7K的HTML文档，
一个59K的样式文件，和三个JS(1K,11K和9K)，总共87K,虽然总大小是一样的，但是内联要去外联的例子
快30%-50%。主要原因是因为外联的例子有多个HTTP请求的开销，即便部件是平行下载的，但还是要比内联
来的慢。

尽管是这个结构，但在真实的世界中，使用外联文件通常产出更快的页面访问。这个得益于外联文件不需要获取，
JS和CSS有机会被浏览器缓存。HTML文档至少包含动态内容，通常不配置为可缓存。内联JS和CSS每一次HTML文件
请求时，都会去下载。另一边，如果JS和CSS被浏览器缓存，HTML文档的尺寸减少了，因为减少了好几个HTTP请求。

关键因素是外联JS和CSS与HTML文档相关缓存的频繁度，这个比较难评估，可以使用下面的测量标准。

#### 页面访问数量
每个用户的页面浏览量越少，就是该使用内联JS和CSS的有力论据。想象一下，一个典型的用户访问你的网站每月
一次，在每次访问中间，任何的JS和CSS缓存都可能已经失效，即便部件有一个很长的expire头。

另一方面，如果一个典型用户有很多的页面访问量，浏览器更可能有外联部件的缓存。使用JS和CSS外联文件的益处
是随着每月每个用户额访问量或者每次session每个用户的页面访问量而水涨船高的。

#### 空缓存VS命中缓存
比较内联和外联时，知道缓存外联文件的潜能非常重要。我们在Yahoo上测量，发现单个用户一天至少命中缓存一次的人数
在40%-60%之间。相同的研究表明页面访问命中缓存的数量在75%-85%之间。注意第一个统计的是“单个用户”而第二个统计
的是“页面访问"。用户可能一天中碰到一次空缓存访问，但是更多的后来页面访问都来自命中缓存。

这些度量非常依赖网站的类型。知道这些数据帮助评估使用拓展文件相对内联潜在的好处。如果你的网站对你的用户有一个
较高的命中缓存，使用外联文件更好。否则，内联会使一种更好的选择。

#### 部件重用
部件在不同网页中重用虑比较大，使用外联；如果重用虑低，使用内联

#### 多因数考虑典型的结果
权衡内联和外联，关键是访问多个HTML请求相关的被缓存的外联JS和CSS的频率。在上面部分说的，我描述了三个指标(页面
访问量，空缓存VS命中缓存，和组件重用)能够帮助你做出最好的决定。

很多网站都在这三个指标之中，他们每个用户每月访问5-15个页面，每个用户每次session访问2-5个页面。40%-60%用户每天
有一次命中缓存，每天页面访问的75%-85%命中缓存。

考虑这些指标，最好的解决办法是发布JS和CSS为外联文件。

#### 首页
唯一的例外是内联使用在首页上。

#### 两者结合的最佳实践
两个技术在这里介绍，让你同时获得内联和外联两者的益处。

######提前下载
首页是网页访问的开始，对于首页我们想要内联js和css，但是对于所有的二级页面使用外联文件。
这个完全可以在首页加载完成后，动态地下载二级页面的外联文件。使得外联文件驻留在浏览器缓存
中，为用户继续访问其他页面而使用。

预下载例子：
http://stevesouders.com/hpws/post-onload.php

```
<script type="text/javascript">
function doOnload( ) {
 setTimeout("downloadComponents( )", 1000);
}
window.onload = doOnload;
// Download external components dynamically using JavaScript.
function downloadComponents( ) {
 downloadJS("http://stevesouders.com/hpws/testsma.js");
 downloadCSS("http://stevesouders.com/hpws/testsm.css");
}
// Download a script dynamically.
function downloadJS(url) {
 var elem = document.createElement("script");
 elem.src = url;
 document.body.appendChild(elem);
}
// Download a stylesheet dynamically.
function downloadCSS(url) {
 var elem = document.createElement("link");
 elem.rel = "stylesheet";
 elem.type = "text/css";
 elem.href = url;
 document.body.appendChild(elem);
}
</script>
```

###### 动态内联
如果一个首页的服务端知道一个部件是否在浏览器缓存中，它能够做出决定是否使用内联
还是外联文件。虽然没有办法让服务端知道浏览器缓存中有什么，但cookie似乎是有用的。
返回服务端一个基于session的cookie，服务端基于cookie值出现使用外联，不出现使用
内联；

动态内联的例子:
http://stevesouders/hpws/dynamic-inlinling.php

```
<?php
if ( $_COOKIE["CA"] ) {
 // If the cookie is present, it's likely the component is cached.
 // Use external files since they'll just be read from disk.
 echo <<<OUTPUT
<link rel="stylesheet" href="testsm.css" type="text/css">
<script src="testsma.js" type="text/javascript"></script>
OUTPUT;
}
else {
 // If the cookie is NOT present, it's likely the component is NOT cached.
 // Inline all the components and trigger a post-onload download of the files.
 echo "<style>\n" . file_get_contents("testsm.css") . "</style>\n";
 echo "<script type=\"text/javascript\">\n" . file_get_contents("testsma.js") .
"</script>\n";
 // Output the Post-Onload Download JavaScript code here.
 echo <<<ONLOAD
<script type="text/javascript">
function doOnload( ) {
 setTimeout("downloadComponents( )", 1000);
}
window.onload = doOnload;
// Download external components dynamically using JavaScript.
function downloadComponents( ) {
 document.cookie = "CA=1";
[snip...]
ONLOAD;
}
?>
```