虽然对C和C++都不怎么熟悉，但是总忍不住想“扒掉Node的衣服，一探究竟”。今天算是有了初步的成功。下面就分享给大家如何来Debug Node本身，注意是Node本身哦，不是Node应用。（对GDB很熟悉的人来说肯定是小菜一碟）

步骤如下：

1. 从[Node官网](http://nodejs.org/download/)，下载源代码
2. 解压代码后，到根目录下运行
    
    ./configure --prefix 安装路径 --gdb （这里要加上--gdb标志）

3. make
4. 编译好以后，要到out目录下，运行 

    gdb ./Release/node （这里很重要，不能跑到Release目录中，gdb ./node，这样会找不到源代码）

5. 如果，你要运行一个外部node文件的话，别忘记通过gdb传递参数给node

    (gdb) set args path/of/node/file.js

就这么简单！