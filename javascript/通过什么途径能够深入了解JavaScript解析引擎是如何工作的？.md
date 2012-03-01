昨天收到一封来自深圳的一位前端童鞋的邮件，邮件内容如下（很抱歉，未经过他的允许，公开邮件内容，不过我相信其他人肯定也有同样的问题，所以，直接把问题原文抛出来）：

“读了你的几篇关于JS（变量对象、作用域、上下文、执行代码）的文章，我个人觉得有点抽象，难以深刻理解。我想请教下通过什么途径能够深入点的了解javascript解析引擎在执行代码前后是怎么工作的，ecma英文版实在看不下去呵呵。”

其实这个问题个人觉得太笼统了，直接回答很难回答，所以，我打算先把他的问题拆解成如下几个子问题，并对其表达个人的观点，希望对有同样困惑的童鞋能够有所帮助。

## 1. 什么是JavaScript解析引擎？

简单地说，JavaScript解析引擎就是能够“读懂”JavaScript代码，并准确地给出代码运行结果的一段程序。比方说，当你写了 *var a = 1 + 1;* 这样一段代码，JavaScript引擎做的事情就是看懂（解析）你这段代码，并且将a的值变为2。

学过编译原理的人都知道，对于静态语言来说（如Java、C++、C），处理上述这些事情的叫**编译器（Compiler）**，相应地对于JavaScript这样的动态语言则叫**解释器（Interpreter）**。这两者的区别用一句话来概括就是：**编译器是将源代码编译为另外一种代码（比如机器码，或者字节码），而解释器是直接解析并将代码运行结果输出**。 比方说，firebug的console就是一个JavaScript的解释器。

但是，现在很难去界定说，JavaScript引擎它到底算是个解释器还是个编译器，因为，比如像V8（Chrome的JS引擎），它其实为了提高JS的运行性能，在运行之前会先将JS编译为本地的机器码（native machine code），然后再去执行机器码（这样速度就快很多），相信大家对**JIT（Just In Time Compilation）**一定不陌生吧。

我个人认为，不需要过分去强调JavaScript解析引擎到底是什么，了解它究竟做了什么事情我个人认为就可以了。对于编译器或者解释器究竟是如何看懂代码的，翻出大学编译课的教材就可以了。

这里还要强调的就是，JavaScript引擎本身也是程序，代码编写而成。比如V8就是用C/C++写的。

## 2. JavaScript解析引擎与ECMAScript是什么关系？

JavaScript引擎是一段程序，我们写的JavaScript代码也是程序，如何让程序去读懂程序呢？这就需要定义规则。比如，之前提到的*var a = 1 + 1;*，它表示：

- 左边var代表了这是申明（declaration）,它申明了a这个变量
- 右边的+表示要将1和1做加法
- 中间的等号表示了这是个赋值语句
- 最后的分号表示这句语句结束了

上述这些就是规则，有了它就等于有了衡量的标准，JavaScript引擎就可以根据这个标准去解析JavaScript代码了。那么这里的ECMAScript就是定义了这些规则。其中ECMAScript 262这份文档，就是对JavaScript这门语言定义了一整套完整的标准。其中包括：

- var，if，else，break，continue等是JavaScript的关键词
- abstract，int，long等是JavaScript保留词
- 怎么样算是数字、怎么样算是字符串等等
- 定义了操作符（+，-，>，<等）
- 定义了JavaScript的语法
- 定义了对表达式，语句等标准的处理算法，比如遇到**==**该如何处理
- ⋯⋯

**标准**的JavaScript引擎就会根据这套文档去实现，注意这里强调了**标准**，因为也有不按照标准来实现的，比如IE的JS引擎。这也是为什么JavaScript会有兼容性的问题。至于为什么IE的JS引擎不按照标准来实现，就要说到浏览器大战了，这里就不赘述了，自行Google之。

所以，简单的说，ECMAScript定义了语言的标准，JavaScript引擎根据它来实现，这就是两者的关系。

## 3. JavaScript解析引擎与浏览器又是什么关系？

简单地说，JavaScript引擎是浏览器的组成部分之一。因为浏览器还要做很多别的事情，比如解析页面、渲染页面、Cookie管理、历史记录等等。那么，既然是组成部分，因此一般情况下JavaScript引擎都是浏览器开发商自行开发的。比如：IE9的Chakra、Firefox的TraceMonkey、Chrome的V8等等。

从而也看出，不同浏览器都采用了不同的JavaScript引擎。因此，**我们只能说要深入了解哪个JavaScript引擎**。

## 4. 深入了解其内部原理的途径有哪些？

搞清楚了前面三个问题，那这个问题就好回答了。个人认为，主要途径有如下几种（依次由浅入深）：

- 看讲JavaScript引擎工作原理的书  
  这种方式最方便，不过我个人了解到的这样的书几乎没有，但是[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/)博客上的文章真的是非常的赞，建议直接看英文，实在英文看起来吃力的，可以看我翻译的[译本](http://blog.goddyzhao.me/JavaScript-Internal)
- 看ECMAScript的标准文档  
  这种方式相对直接，原汁原味，因为引擎就是根据标准来实现的。目前来说，可以看[第五版](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf)和[第三版](http://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%203rd%20edition,%20December%201999.pdf)，不过要看懂也是不容易的。
- 看JS引擎源代码  
  这种方式最直接，当然也最难了。因为还牵涉到了如何实现词法分析器，语法分析器等等更加底层的东西了，而且并非所有的引擎代码都是开源的。

## 5. 以上几种方式中第一种都很难看明白怎么办？
  
其实第一种方式中的文章，作者已经将文档中内容提炼出来，用通俗易懂的方式阐述出来了。如果，看起来还觉得吃力，那说明还缺少两块的东西：

- 对JavaScript本身还理解的不够深入  
  如果你刚刚接触JavaScript，或者说以前甚至都没有接触过。那一下子就想要去理解内部工作原理，的确是很吃力的。首先应该多看看书，多实践实践，从知识和实践的方式来了解JavaScript预言特性。这种情况下，你只需要了解现象。比方说，*(function(){})()* 这样可以直接调用该匿名函数、用闭包可以解决循环中的延迟操作的变量值获取问题等等。要了解这些，都是需要多汲取和实践的。实践这里就不多说了，而知识汲取方面可以多看看书和博客。这个层面的书就相对比较多了，[《Professional JavaScript for Web Developers》](http://www.amazon.com/Professional-JavaScript-Developers-Nicholas-Zakas/dp/1118026691/)就是本很好的书（中文版请自行寻找）。
- 缺乏相应的领域知识  
  当JavaScript也达到一定深度了，但是，还是看不大明白，或者没法很深入到内部去一探究竟。那就意味着缺少对应的领域知识。这里明显的就是编译原理相关的知识。不过，其实对这块了解个大概基本看起来就没问题了。要再继续深入，那需要对编译原理了解的很深入，比如说词法分析采用什么算法，一般怎么处理。会有什么问题，如何解决，AST生成算法一般有哪几种等等。那要看编译原理方面的书，也有基本经典的书，比如[《Compilers: Principles, Techniques, and Tools》](http://www.amazon.com/Compilers-Principles-Techniques-Tools-2nd/dp/0321486811/)这本也是传说中的龙书，还有非常著名的[《SICP》](http://mitpress.mit.edu/sicp/full-text/book/book.html)和[《PLAI》](http://www.cs.brown.edu/~sk/Publications/Books/ProgLangs/)。不过其实根据个人经验，对于Dmitry的文章，要看懂它，只要你对JavaScript有一定深度的了解，同时你大学计算机的课程都能大致掌握了（尤其是操作系统），也就是说基础不错，理解起来应该没问题。因为这些文章基本没有涉及底层编译相关的，只是在解释文档的内容，并且其中很多东西都是相通的，比如：context的切换与CPU的进程切换、函数相关的的局部变量的栈存储、函数退出的操作等等都是一致的。
  
以上就是个人对这个问题的看法，除此之外，我觉得，学习任何技术都不能操之过急，要把基础打扎实了，这样学什么都会很快。
