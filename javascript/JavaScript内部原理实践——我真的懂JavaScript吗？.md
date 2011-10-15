通过翻译了[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/)的关于ECMAScript-262-3 [JavaScript内部原理](http://goddyzhao.tumblr.com/JavaScript-Internal)的文章，
从理论角度对JavaScript中部分特性的内部工作机制有了一定的了解。  
但是，邓爷爷说过：**“实践才是检验真理的唯一标准”**。  
所以，我打算通过**从内部原理来解释一些经常在笔试或者面试中遇到的关于JavaScript语言层面的题目**来进一步学习和掌握JavaScript内部工作原理。   

那么，首先就是要去找那些题目，google了一圈终于找到了来自**Dmitry Baranovskiy**的非常著名的[5个问题](http://dmitry.baranovskiy.com/post/91403200)，
这5个问题，NCZ给出了[非常清楚的解释](http://www.nczonline.net/blog/2010/01/26/answering-baranovskiys-javascript-quiz/)。
不过，我还是想尝试下从low-level——JavaScript内部工作机制的角度去解释下这些问题。  

好吧，我承认我废话很多，那就开始吧。

问题 #1
--------
* * *
<pre><code>if (!("a" in window)) {
    var a = 1;
}
alert(a);</code></pre>
 
*  **正确答案：** undefined  
  
  
**解释：**  
这个问题，初一看感觉答案很自然是1。因为从上到下执行下去，_if_语句中的条件应该为 _true_，因为"a"的确是没有定义啊。
随后，顺理成章地进入 _var a = 1;_，最后，alert出来就应该是1。  

而实施上，从JavaScript内部工作原理去看，在[变量对象](http://goddyzhao.tumblr.com/post/11141710441/variable-object)中讲过，
JavaScript处理上下文分为两个阶段：  

*  进入执行上下文  
*  执行代码  

可以理解为，第一个阶段是静态处理阶段，第二个阶段为动态处理阶段。  

而在静态处理阶段，就会创建 _变量对象（variable object）_，并且将变量申明作为属性进行填充。  
到了执行阶段，才会根据执行情况，来对变量对象中属性（就是申明的变量）的值进行更新。  

针对这个问题，其实际过程如下：  

*  **进入执行上下文**：  创建VO，并填充变量申明 _a_，VO如下所示：    
<pre><code>VO(global) = {
	a: undefined
}</code></pre>
所以，这个时候，a其实已经存在了。

*  **执行代码**：  进入 _if_语句，发现条件判断 _"a" in window_ 为**true**。于是就不会进入if代码块，直接执行alert语句，因此，最终为**undefined**。  


问题 #2
--------
* * *
<pre><code>var a = 1,
    b = function a(x) {
        x && a(--x);
    };
alert(a);</code></pre>
  
*  **正确答案：** 1    
  


