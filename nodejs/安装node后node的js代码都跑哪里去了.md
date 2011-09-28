[Node](http://nodejs.org)的[源代码](https://github.com/joyent/node)有两部分组成：  
1.  C/C++代码（包括Google的V8引擎代码）： 位于src目录下  
2.  JavaScript代码：node.js位于src目录下，其余全部位于lib目录下，**而且数量很多**  

但是，奇怪的是，按照node wiki上的[安装文档](https://github.com/joyent/node/wiki/Installation)安装好Node之后，却发现在安装目录中并没有任何JavaScript文件，于是我就很想知道为什么（我犯贱）  

因为node运行的时候这些js代码都是需要的，而且基本是node的核心代码，  
所以**大致猜测**：JS文件在make的过程中可能被翻译然后合并到C代码中一同编译成了二进制代码  
通过粗略地看了下node的几个和安装相关的脚本文件后，大概知道了真正的原因，如下：  

**首先，回忆下node的安装过程**：  
1.  ./configure --prefex=node_install_path: 配置node安装，指定安装目录  
2.  make：编译代码  
3.  make install: 安装  

那么，按照这个顺序首先去看下**configure**文件，发现其代码非常少：
<pre><code>if [ ! -z "`echo $CC | grep ccache`" ]; then
  echo "Error: V8 doesn't like ccache. Please set your CC env var to 'gcc'"
  echo "  (ba)sh: export CC=gcc"
  exit 1
fi

CUR_DIR=$PWD

#possible relative path
WORKINGDIR=`dirname $0`
cd "$WORKINGDIR"
#abs path
WORKINGDIR=`pwd`
cd "$CUR_DIR"

"${WORKINGDIR}/tools/waf-light" --jobs=1 configure $*

exit $?</code></pre>

没啥发现，继续去看make执行的**makefile**，makefile里面一堆代码，没怎么看懂，感觉没啥发现  

这个时候，心想这个思路不好，不一定能够找到原因，于是决定换个思路：  
既然JavaScript文件都是在lib目录下，那直接搜索lib看看哪些文件用到了，这个方法果然奏效，搜索发现根目录下的**node.gyp**文件非常可疑，  
其文件内容就感觉是个build文件，其中有如下这段代码：
<pre><code>'library_files': [
      'src/node.js',
      'lib/_debugger.js',
      'lib/_linklist.js',
      'lib/assert.js',
      'lib/buffer.js',
      ...</code></pre>

这里就是把所有JavaScript文件都罗列出来了，这时已经看到光明了，在此文件中找**library_files**这个变量，又发现如下代码：
<pre><code>'actions': [
        {
          'action_name': 'node_js2c',

          'inputs': [
            './tools/js2c.py',
            '&lt;@(library_files)',
          ],

          'outputs': [
            '&lt;(SHARED_INTERMEDIATE_DIR)/node_natives.h',
          ],
          ...</code></pre>

猜测就是一个build任务，其输入源就是这些JS文件，输出是一个C的头文件，这里有个文件极其可疑**./tools/js2c.py**，继续追看该文件  
终于找到真凶了：
<pre><code># This is a utility for converting JavaScript source code into C-style
# char arrays. It is used for embedded JavaScript code in the V8
# library.</code></pre>

这三行注释就大致等于告诉我们了，node在安装过程中会把JavaScript代码翻译成C语言风格的字符数组，在V8引擎中运行，这下终于豁然开朗了。

**说明：**以上过程纯属自己YY，仅供大家一起YY。
