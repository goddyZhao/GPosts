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

而事实上，从JavaScript内部工作原理去看，在[变量对象](http://goddyzhao.tumblr.com/post/11141710441/variable-object)中讲过，
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
  
**解释：**  
这个问题，第一反应可能会是将 function a打印出来。因为明明就看到了function a了。看似，也顺其自然。  

但是，事实并非如此。还是和此前一个问题一样。从两个阶段来分析：  

*  **进入执行上下文**： 这个时候要注意了， _b = function a(){}_，这里的 _function a_并非函数申明，因为整个这个句话属于**赋值语句（assignment statement）**，所以，这里的 _function a_会被看作是函数表达式。
函数表达式是不会对VO造成影响的。所以，这个时候VO中其实只有 **a和x（函数形参）**：  
<pre><code>VO(global) = {
	a: undefined,
	b: undefined
}</code></pre>

*  **执行代码**： 这个时候a的值修改为1：  
<pre><code>VO(global) = {
	x: undefined,
	a: 1
}</code></pre>

所以，最后alert(a)的结果是1。  


问题 #3
--------
* * *
<pre><code>function a(x) {
    return x * 2;
}
var a;
alert(a);</code></pre>

*  **正确答案：** 函数a  

**解释：**  
这个问题，很多人可能会以为是： undefined。理由可能是，明明看到了 _var a_定义在了function a的后面，感觉应该会覆盖之前a的申明。  

事实又是怎样的呢？ 老套路，从两个阶段来分析：  

*  **进入执行上下文**： 这里出现了名字一样的情况，一个是函数申明，一个是变量申明。那么，根据[变量对象](http://goddyzhao.tumblr.com/post/11141710441/variable-object)
介绍的，填充VO的顺序是:  函数的形参 -> 函数申明 -> 变量申明。  
上述例子中，变量a在函数a后面，那么，变量a遇到函数a怎么办呢？还是根据 _变量对象_中介绍的，当变量申明遇到VO中已经有同名的时候，不会影响已经存在的属性。因此，VO如下所示：  
<pre><code>VO(global) = {
	a: 引用了函数申明“x”
}</code></pre>

*  **执行代码**：啥也没变化  

所以，最终的结果是：函数a。  


问题 #4
--------
* * *
<pre><code>function b(x, y, a) {
    arguments[2] = 10;
    alert(a);
}
b(1, 2, 3);</code></pre>

*  **正确答案：** 10  

**解释：**  
个人感觉这个问题其实不是很复杂。这里也不需要从两个阶段去分析了。根据 _变量对象_中介绍的，**arguments对象的properties-indexes和实际传递的参数是共享的**
也就是说，通过arguments\[2\]修改的参数，也会影响到a，所以，这里的值是10。但是，要注意的是和**实际传递的值**，所以，如果把上述问题改成如下形式：  
<pre><code>function b(x, y, a) {
    arguments[2] = 10;
    alert(a);
}
b(1, 2);</code></pre>

结果就会是： **undefined**。因为，并没有传递a的值。  


问题 #5
--------
* * *
<pre><code>function a() {
    alert(this);
}
a.call(null);</code></pre>

*  **正确答案：** 全局对象（window）  

**解释：**  
这个问题，可能会比较困惑。因为懂call的童鞋都会觉得，call的时候把null传递为了当前的上下文了。里面的this应该是null才对啊。  

事实却是： 前面都没错，this会是null。但是，[this](http://goddyzhao.tumblr.com/post/11218727474/this)中介绍过，null是没有任何意义的，因此，最终会变成全局对象。
所以，这里结果就变成了全局对象。  这里还有ECMAScript-262-3标准文档中的一句话作为证据：  
_“If thisArg is null or undefined, the called function is passed the global object as the this value. Otherwise, the called function is passed ToObject(thisArg) as the this value.”_  



总结
--------
* * *
上面这5个问题其实也只是牵涉到了JavaScript内部原理中的部分知识点，要想了解更多，还是建议读完[JavaScript内部原理系列](http://goddyzhao.tumblr.com/JavaScript-Internal)
以及去看[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/)的文章。
