[Fiddler](http://www.fiddler2.com/fiddler2/) 相信大家，尤其是前端工程师们都知道。
用它的文件替换功能，将线上的静态资源文件（JS、CSS、图片）替换为本地相应的文件，来调试线上（代码都被压缩过）UI的问题。的确是一神器。（相比，它的HTTP请求的inspector功能因为各大主流浏览器都内置有这功能，反而现在用的不多）。

但是，Fiddler最大的问题就是只支持Windows，这对于Mac党和Linux党来说，有些遗憾。

以往，总是得开个虚拟机来用Fiddler。后来也有了跨平台的类似Fiddler的工具，如：[Charles](http://www.charlesproxy.com/)、[Rythem](http://www.alloyteam.com/2012/05/web-front-end-tool-rythem-1/)以及[Tinyproxy](https://banu.com/tinyproxy/)。

尽管这些各有优势，但是，都没有办法满足我的需求：

* 支持Mac、Linux以及Windows
* 支持HTTP和HTTPS（很重要）
* 支持单文件替换
* 支持combo文件替换（即多个文件合并为一个文件的替换）
* 支持目录替换

下面这张图显示了，根据我的需求，罗列出的各工具的支持情况：

     +---+---------+---------+---------+-----------+
     | R | Fiddler | Charles | Rythem  | TinyProxy |
     |---------------------------------------------|
     | A |  Win    |    Y    | Mac&Win | Mac&Linux |
     |---------------------------------------------|
     | B |   Y     |    Y    |  Y/N    |    Y      |
     |---------------------------------------------|
     | C |   Y     |    Y    |   Y     |    N      |
     |---------------------------------------------|
     | D |   N     |    N    |  Y/N    |    N      |
     |---------------------------------------------|
     | E |  Y/N    |    N    |   Y     |    N      |
     +---------------------------------------------+

     A: 支持Mac、Linux以及Windows
     B: 支持HTTP和HTTPS
     C: 支持单文件替换
     D: 支持combo文件替换（即多个文件合并为一个文件的替换）
     E: 支持目录替换
     Y: 支持
     N: 不支持
     Y/N: 不完全支持

这就是为什么会有[NProxy](http://goddyzhao.me)，它满足所有上述我的需求。这里并不表示Nproxy就比其他这4个工具优秀，只是NProxy在文件替换上更胜一筹。
它不提供HTTP Inspector功能，只专注在文件替换功能上。

目前，NProxy发布了1.3.0， 据我所知，除了我自己所在的公司——[SuccessFactors(An SAP Company)](http://www.successfactors.com/homepage.html)在使用之外，部分[天猫](http://tmall.com)的前端也在使用。如果你也在用，麻烦请告诉[我](http://weibo.com/goddyzhao)。

因此，各位平时用Mac和Linux的朋友（当然windows也完全可以），可以使用NProxy！具体的安装和使用非常简单，可以参考[官网介绍](http://goddyzhao.me)