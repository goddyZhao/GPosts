今天某个同事遇到一个诡异的问题，问题描述如下：

**需求**：通过脚本动态修改script标签的src来载入一段外部脚本并执行  
**实现方式(#1)**：

	<script type="text/javascript" id="external-script">
	</script>
	<script type="text/javascript">
		document.getElementById('external-script').src='url2';
	</script>

**url2的内容如下**：

	alert('I am dynamic');

**结果**：

- Chrome: 什么事都没发生（没有请求url2）
- Firefox: 什么事都没发生（没有请求url2）
- IE9：什么事都没发生（请求url2但不执行url2的脚本）
- IE(6,7,8): I am dynamic(请求并执行了url2的脚本)

注意实现方式中，第一段的script标签中间是有内容的（空格、换行符以及回车符）。

如何来解释这个问题呢？要解释这个问题，我们来看两个变种的例子，第一个例子（明确内联内容)，如下所示（#2）：

	<script type="text/javascript" id="external-script">
	alert('I am inline');
	</script>
	<script type="text/javascript">
		document.getElementById('external-script').src='url2';
	</script>

结果如下：

- Chrome: I am inline（没有请求url2）
- Firefox: I am inline（没有请求url2）
- IE9：I am inline（请求url2但不执行url2的脚本）
- IE(6,7,8): I am inline I am dynamic(请求并执行了url2的脚本)

再来看看第二个变种的例子(#3)：

	<script type="text/javascript" id="external-script" src="url1">
	alert('I am inline script');
	</script>
	<script type="text/javascript">
		document.getElementById('external-script').src='url2';
	</script>

其中url1的内容如下：

	alert('I am url1');

结果如下：

- Chrome: I am url1（没有请求url2）
- Firefox: I am url1（没有请求url2）
- IE9：I am url1（请求url2但不执行url2的脚本）
- IE(6,7,8): I am url1 I am dynamic(请求并执行了url2的脚本)

首先这里肯定的是src属性是修改成功的，可以通过看dom的变化看到src已经设置进去了。这个时候我们比对这三个例子，思考几十秒。分析下这三个例子，其实#2和#1是一样的，这里用#2是为了说明#1中的空格、换行符以及回车符会被浏览器认为是内联的脚本。通过比对#2和#3，是不是会让你想到什么？没错，我们第一个会想到的就是：_当script标签既有src属性又有内联脚本的时候浏览器该如何处理？_ ， 先来解释这个问题。

一谈到浏览器应该怎样处理，就不得不翻出各种宝典，这次不再是葵花宝典了，而是九阴真经([W3C的HTML4标准](http://www.w3.org/TR/1999/REC-html401-19991224/interact/scripts.html#h-18.2.4))，标准中关于script标签的src部分有如下一段话：  
> If the src attribute is not set, user agents must interpret the contents of the element as the script. If the src has a URI value, user agents must ignore the element's contents and retrieve the script via the URI

上面这段话的意思就是说：_如果src没有设置，那么就执行内联脚本，如果src设置了浏览器就必须忽略内敛脚本而要去请求src指定的url的内容_

这解释了为什么#3中标准浏览器（甚至IE6，7，8）都没有执行内联脚本（因为src设置了url1）。

搞清楚了这个基础问题之后，接下来问题就定位到了_动态修改script的src属性的时候浏览器如何处理？_ ，从结果来看，标准的浏览器都没有去请求url2（更改src无效），这回IE6，7，8终于犯傻了。当然了，咱们也不能随随便便说人家犯傻，要拿出证据，这个时候继续拿出九阴真经[W3C的HTML5标准](http://www.w3.org/TR/2012/WD-html5-20120329/the-script-element.html#the-script-element)，其中有这样一句话：  
> Changing the src, type, charset, async, and defer attributes dynamically has no direct effect; these attribute are only used at specific times described below.

意思就是说：_修改src是没用的，对src的处理只会在特定的时候进行（个人猜测就是第一次看到这个属性的时候浏览器会去做相应处理，之后就无视它了）。_

好了，这下真相大白了：这解释了为啥#3和#1中除了IE6，7，8之外其他浏览器都没有去请求url2（IE9请求了，但没执行），而且实验发现IE6，7，8对动态修改src都会做请求执行处理。

最后，这个故事至少告诉我们：写script标签的时候千万别手贱打回车。



参考文档：

- [HTML5标准文档](http://www.w3.org/TR/2012/WD-html5-20120329/the-script-element.html#the-script-element)
- [HTML4标准文档](http://www.w3.org/TR/1999/REC-html401-19991224/interact/scripts.html#h-18.2.4)







