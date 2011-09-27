首先有必要解释下什么是**命令行（Command Line）类型的模块**。  
npm的模块一共分为三类：  
1.  **绑定型（Binding）**：本地模块，C++书写，如[node-png](https://github.com/pkrumins/node-png)  
2.  **库型（Library）**：JS书写，直接使用require('module')这种方式，如[Socket.IO-node](https://github.com/learnboost/Socket.IO-node)  
3.  **命令行型（Command Line）**: 以命令行形式使用，如[json-command](https://github.com/zpoley/json-command)

显而易见，提供命令行形式调用方式的模块就属于命令行型的。那么如何来申明自己的模块以什么命令调用，调用执行代码又如何制定呢？

这里就要说到npm模块的核心文件：**package.json**，此文件是npm模块的配置信息（npm help json可以获取详细信息）。  

其中有一个key就叫**“bin”**，具体形式为**“{ "bin": {"command-name" : "command-file"}}”**，意思是申明用户可以直接使用**“command-name”**这个命令，而输入该命令后就会去执行**“command-file”**这个文件。  
比如json-command这个模块，安装好后就可以看到它的package.json文件中bin配置项为**“{"bin" : {"json" : "./bin/json.js"}}”**。这就申明了命令行名字为**“json”**，对应地会去执行**“bin目录下的json.js”**这个文件。

其次还要解释下npm中安装模块的两种模式：  
1.  **全局模式**： npm -g install module-name，这种模式模块会被安装在node安装记录的lib所在目录的node_modules文件夹中，全局使用  
2.  **本地模式**： npm install module-name，这种模式模块只会被安装在当前目录的node_modules文件夹中，非全局使用

理解了什么叫**“命令行型的模块”**和**“npm模块的安装模式”**之后，再来看看什么叫**“npm中本地安装命令行类型的模块是不注册Path的”**？  
这句话的意思是说：比如像json-command这样的命令行类型的模块，如果是“全局安装”，安装成功之后，就可以直接使用**“json”**的命令了，而如果是本地模式安装，则没法使用**“json”**命令。只能手动找到执行文件，然后调用./command-file这样调用。很是不爽！

后来仔细想了下，npm这样设计也是有道理的，因为命令行的模块，通过命令行使用，一般也都会认为是全局的，就像压缩文件，合并文件之类的命令，一般是不会随着应用的发布而发布的，都只是开发过程中的中间工具。所以，npm对这种类型的模块只有当全局安装的时候才会可以直接使用命令。

那么，**“npm到底是如何实现全局直接使用，而本地就不能使用的呢？”**

通过查看npm的源码，稍作调试就能发现其中的奥妙了：  

*    通过npm的package.json配置文件就能知道npm的命令行入口是**“bin/npm.js”**  
*    npm.js有会去调用**“模块根目录的npm.js”**，并会调用相关的命令，依据来自如下代码：  
<pre><code>npm = require("../npm")</code></pre>
<pre><code>npm.load(conf, function (er) {
  if (er) return errorHandler(er)
  npm.commands[npm.command](npm.argv, errorHandler)
})</code></pre>  

*   **“npm.commands...(npm.argv,errorHandler)”**是一行典型的command模式代码，npm.js中的commands对象肯定维护了所有command的列表。依据来自如下代码：
<pre><code>var cmd = require(__dirname+"/lib/"+a+".js")</code></pre>  
*  那么install命令就会去调用**“lib/install.js”**的install方法，install方法会去做一些**模块查询**，**模块依赖关系处理**，**模块下载**，**模块安装**等工作，之后会去区分本地模式和全局模式，依据来自如下代码：
<pre><code>var fn = npm.config.get("global") ? installMany : installManyTop</code></pre>  
*  之后会去调用lib/build.js中的build函数，依据来自于如下代码：
<pre><code>npm.commands.build([where], false, true, function (er) {
      return cb_(er, d)
    })</code></pre>  
*  然后，再看看build函数的申明中，其**第二个参数**正是表示模块是否全局安装，依据来自如下代码：
<pre><code>function build (args, global, didPre, didRB, cb) {</code></pre>  
*  最后，global参数被赋予给gtop变量，最后判断是否全局，如果是则注册Path否则直接调用回调函数返回，依据来自如下代码：
<pre><code>if (er || !gtop) return cb(er)
var dest = path.resolve(binRoot, b)
     , src = path.resolve(folder, pkg.bin[b])
     , out = npm.config.get("parseable")
                ? dest + "::" + src + ":BINFILE"
                : dest + " -> " + src</code></pre>  
这里很清楚，如果**“!gtop”**，即本地模式就直接调用cb并返回，否则，就去注册Path，其中**“dest->src”**会发现很眼熟，通常全局安装好一个命令行模块之后，都会显示这行log。这行代码其实就做了如下这件事情：  
**将命令执行文件json.js link到node/bin/json命令中**  
其中node的bin路径命令已经被加入到PATH中了，这样输入json的时候，系统自动去PATH中查找对应的执行文件，这样就能够找到了！

---
**参考文献**  
*  [http://howtonode.org/how-to-module](http://howtonode.org/how-to-module)    
*  [https://github.com/isaacs/npm](https://github.com/isaacs/npm)
