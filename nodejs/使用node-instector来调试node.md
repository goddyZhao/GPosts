大部分程序员（比如我），开发的过程中，其实只有20%的时间在写代码，另外80%的时间都在调试代码。完全符合著名的[80/20法则](http://en.wikipedia.org/wiki/80/20)。  
好吧，我承认我在滥用伟大的法则，废话很多。其实，我就想说明**我们大部分时间都在调试（Debug）而不是在写代码**。  

所以，要想尽一切办法提高调试的效率，这样有助于提高开发效率。
对于node的调试，官方wiki中有[在eclipse中调试node](https://github.com/joyent/node/wiki/Using-Eclipse-as-Node-Applications-Debugger)的文章。  

但是，不知为何，我在eclipse中总是不成功（eclipse 3.7 + ubuntu 11.04）,况且，要是不用eclipse那咋办呢？  
于是，我就另谋出路，在github上一番寻觅之后，发现了：[node-inspector](https://github.com/dannycoates/node-inspector)。  
用了一把之后，感觉神清气爽，遂推荐给大家。  

那么如何使用呢？其实官方有很详细的教程，不过这里我还是要赘述下，并且以joyent官方的例子来作为调试代码：  
<pre><code>// dbgtest.js

var sys=require('sys');
var count = 0;

sys.debug("Starting ...");

function timer_tick() {
  count = count+1;
  sys.debug("Tick count: " + count);
  if (count === 10) {
    count += 1000;
    sys.debug("Set break here");
 }
 setTimeout(timer_tick, 1000);
}

timer_tick();</code></pre>

具体调试步骤如下：  

*  **以debug模式来运行上述代码**： node默认运行方式是直接 node filename。node还提供了两种debug模式运行：  
1. node --debug\[=port\] filename （这种方式，其实就是在指定的端口（默认为 5858）监听远程调试连接）  
2. node --debug-brk\[=port\] filename （这种方式在监听的同时，会在代码执行的时候，强制断点在第一行，这样有个好处就是：**可以debug到node内部的是如何运行的**）  
这里采用第二种方式运行： _node --debug-brk dbgtest.js_。  
这个时候终端就会出现这样一个log：  
<pre><code>debugger listening on port 5858</code></pre> 说明已经开始监听了。  

*  **启动node-inspector**： 首 info  - socket.io started
visit http://0.0.0.0:8080/debug?port=5858 to start debugging先，我们要**全局**安装node-inspector模块，如果你真不知道如何安装模块，或者不知到为啥要全局安装，可以参看[这篇文章](http://goddyzhao.tumblr.com/post/9835631010/no-direct-command-for-local-installed-command-line-modul)），
然后，直接输入命令： _node-inspector_ 就运行起来了。起来后，终端可以看到如下log：  
<pre><code> info  - socket.io started
visit http://0.0.0.0:8080/debug?port=5858 to start debugging</code></pre>

通过看log也大致能够猜到了，node-inspector就是利用socket.io来监听5858端口实现的。  

*  **使用浏览器调试工具来调试node（如： chrome developer tool）**： 在chrome中输入： _http://localhost:8080/debug?port=5858_，就会出现如下页面：    
![chrome下调试界面](http://farm7.static.flickr.com/6235/6249843270_42121ed082_z.jpg)  

这里很明显看到，直接代码就停在第一行，那么，接下来如何断点来debug相信身为前端的童鞋就不用我再赘述了吧。  