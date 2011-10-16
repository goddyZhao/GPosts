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


