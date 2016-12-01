
# WafBook CN  
Thomas Nagy

建立了一个QQ群, 欢迎大家来交流 : WAF Build System讨论交流 54669543

### 引言
----
Copyright © 2010-2016 Thomas Nagy
Copies of this book may be redistributed, verbatim, and for non-commercial purposes. The license for this book is by-nc-nd license.

#### 关于生成系统
随着软件越来越复杂，功能越来越丰富，创建软件所需的过程也越来越多。复杂性的增加自然来自项目的基本需求。  
1. 可用的软件(程序)是机器可读的但通常不是人类可读的。  
2. 因此，被称为源代码的人类可读的输入数据被编译器或归档器转换为可分发的软件。  
3. 源代码通常被分解为互相依赖的单元例如文件，模块，类和函数。  
4. 这些单元随着时间的版本演变是通过版本控制软件管理的。  
5. 其他程序由源代码生成附加数据如测试结果及静态分析结果。  
6. 源代码的处理过程耗时而且容易出错所以通过其他软件帮助其自动化。  

虽然有时编译器有时会提供完全自动的生成，但他们通常仅限于非常特别的功能上。例如，Java编译器（javac）能够一次生成整个源码树，但另一种编译器需要生成归档文件（jar）。文本编辑器和集成开发环境（IDE如Xcode或Eclipse）能够提供自动生成功能，但是他们这样的用户界面最适合的工作是编写软件而不是运行生成。版本控制系统例如Git最适合管理终端用户的文件，但通常不适合调用编译器或者运行脚本。虽然解决方案集成系统（Jenkins, Teamcity）有时被理解为生成系统，但他们通常无法独立生成软件。他们需要生成脚本以及运行在他们生成代理（build agents Maven, Make, 等等）上的生成软件。  

因此我们认为生成软件是软件开发过程中一个独特的活动，需要独特的工具。术语“生成系统”通常用于表示这样的软件，并且我认为这里有两种定义应该被区分：  
A.帮助软件项目自动化处理的软件专注于源代码的处理。  
B.处理特定软件项目所需的工具集合：编译器，生成脚本，编配软件，版本控制系统，等等。  

本篇文档其余部分Waf使用第一个定义。

#### Waf框架  
生成系统通常会绑定到他们所属的特定框架上。例如，Visual Studio项目一般需要MSBuild而Angular.js项目一般需要Npm。这些解决方案通常专注于特殊的特性而且通常在处理其他语言或不同项目时会受到限制。例如，Ant比Make更加适合Java项目，但是在生成简单的c项目时Ant比Make更加复杂。由于编程语言和解决方案的不断发展，创造一个适合所有的理想生成系统是不可能的，所以要在框架的专业化和通用化之间权衡。  

这里还有一些生成系统旨在解决的常见问题：
* 区分编译器和脚本的处理流程
* 仅在记录“有改变”的情况下进行处理
* 并行处理提高效率
* 使软件测试更加方便如配置测试
* 为典型的编译器和工具提供支持

Waf提供的功能满足了插件式的随时使用，并且提供了一个必要时扩展其功能的框架。与其他框架相比主要区别在于他的设计：  
1. Waf只需要Python支持，没有对其他软件或库的依赖。  
2. Waf没有定义一种新的语言，它完全使用可重用的Python模块。  
3. Waf不需要依赖代码生成器(Makefiles)实现高效率和可扩展的生成。  
4. Waf的目标为定义一个关注于从运行命令过程中分离出定义目标过程的对象。  

#### 本书的目的
本书主要面向Waf生成系统的新用户和进阶用户，意图通过实际实例展示Waf生成系统的使用，描述Waf扩展系统，并且提供Waf系统的内部概貌。  

我们同样了解生成系统在一天天的被重新改变，所以我们也希望生成系统作者可以从这篇文档中获得启发以此重用现有的技术。  

这些章节依据难度排序，从基础的使用Waf和Python开始，逐步深入最复杂的主题。因此建议按照章节顺序阅读，也可以在阅读之前从Waf源码配套的样例开始。  


### 2.下载和安装
----

#### 2.1获取Waf文件
##### 2.1.1如何下载Waf  
release释出文件可以从主站点下载，源文件可以从Github上获取。下载或大多数项目的提交都需要使用项目公钥（public key）登陆。
* Waf可执行文件中包含一个可以被如下脚本验证的签名：  

        $ wget https://waf.io/waf-1.9.5
        $ ./waf/utils/verify-sig.py waf-1.9.5

* 分发的源文件为其归档文件提供了签名文件：  

        $ wget https://waf.io/waf-1.9.5.tar.bz2
        $ wget https://waf.io/waf-1.9.5.tar.bz2.asc
        $ gpg --verify waf-1.9.5.tar.bz2.asc  

* 大多数项目提交都使用相同的公钥签名：  

        $ git clone https://github.com/waf-project/waf.git
        $ cd waf/
        $ git show --show-signature
        commit b73ccba03cd5f34b40a36e1d60b6a082a04cd563
        gpg: Signature made sam. 16 jul. 2016 17:31:19 CEST
        gpg:                using RSA key 0x49B4C67C05277AAA
        ...

##### 2.1.2如何运行Waf
可执行文件可以直接通过Python解释器运行，如cPython2.5至3.5，Pypy或JPython>=2.5。它提供了它自己压缩成二进制流的库文件，执行后这个库将解压到文件目录中的隐藏文件夹中（删除时会重新创建）。这个方案允许在相同文件夹和各个Python解释器版本下执行Waf的不同版本：  

      $ python waf-1.9.5 --help
      $ ls -ld .waf*
      .waf-1.9.5-2c924e3f453eb715218b9cc852291170

不需要安装，但Python解释器必须有一个bzip2解压缩模块；如果缺少此模块，那么可能需要使用源码生成Waf（请参阅下一节）。  

此外，Waf文件所在的文件夹必须是可以写入的；如果无法做到，一个选择就是将WAFDIR环境变量指向一个其中包含名为waflib目录的文件夹。

另一种方法是在可见文件夹中提供waf文件，然后可以将它保存在例如Git这样的版本控制解决方案之中。例如，官方waf-light脚本并没有包含生成的waf库文件，但如果waflib文件夹存在，就可以像waf一样使用。  

下图便表示查找waflib文件夹的过程：  
![waflib](https://github.com/kklook/wafbook_cn/raw/master/book_image/waflib.png)

##### 2.1.3 权限和别名

由于Waf文件是一个Python脚本，通常通过调用Python执行：  

      $ python waf

在Unix-like系统上，通常可以通过设置执行权限，来避免每次调用Python的麻烦：  

      $ chmod 755 waf
      $ ./waf --version
      waf 1.9.5 (54dc13ba5f51bfe2ae277451ec5ac1d0a91c7aaf)

如果命令行解释器支持别名，建议设置别名而非重复输入命令：  

      $ alias waf=$PWD/waf
      $ waf --version
      waf 1.9.5 (54dc13ba5f51bfe2ae277451ec5ac1d0a91c7aaf)

另外，也可以修改执行环境的路径来指向waf的位置：  

      $ export PATH=$PWD:$PATH
      $ waf --version
      waf 1.9.5 (54dc13ba5f51bfe2ae277451ec5ac1d0a91c7aaf)

在Windows系统上为了方便起见，提供了一个waf.bat文件检测Python应用程序是否存在。它假设Python应用程序处于Waf文件相同的文件夹中。  

#### 2.2 定制与再分发

##### 2.2.1 如何生成Waf可执行文件
生成Waf需要Python解释器版本在2.6-3.5之间。源码被处理过以支持Python 2.5。  

      $ wget https://waf.io/waf-1.9.5.tar.bz2
      $ tar xjvf waf-1.9.5.tar.bz2
      $ cd waf-1.9.5
      $ ./waf-light
      Configuring the project
      Setting top to                           : /home/user/waf
      Setting out to                           : /home/user/waf/build
      Checking for program 'python'            : /usr/bin/python
      Waf: Entering directory `/waf-1.9.5/build'
      [1/1] Creating waf
      Waf: Leaving directory `/waf-1.9.5/build'
      'build' finished successfully (0.726s)

对于较老版本的Python解释器，可以使用gzip压缩替代bzip2压缩来创建waf文件：  

      $ python waf-light --zip-type=gz

可以添加附加的扩展，并重新分配为waf文件的一部分。例如，源发布文件在waflib/extras文件夹下包含几个测试阶段的扩展。通过在--tools选项添加相对路径将会引入相应的文件，而添加绝对路径可以引用在文件系统上的任何文件，特别是非Python文件（他们最后将会被放置于本地的 waflib/extras/ 文件夹）：  

      $ python waf-light --tools=swig,msvs

##### 2.2.2 如何提供自定义的初始化模块
提供一个初始化扩展，也可以用于在常规函数执行前执行的自定义功能。假设在当前目录中存在一个名为aba.py的文件：  

      def foo():
              from waflib.Context import WAFVERSION
              print("This is Waf %s" % WAFVERSION)

如下命令将创建一个自定义的waf文件，它将在调用waf库之前导入并执行函数foo。  

      $ python waf-light --make-waf --tools=msvs,$PWD/aba.py
         --prelude=$'\tfrom waflib.extras import aba\n\taba.foo()'
      $ ./waf --help
      This is Waf 1.9.5
      [...]

也可以添加return语句；请参阅waf-light的内容以了解更多信息，或研究build system kit中说明如何创建派生自waf的生成系统的样例。  

##### 2.2.3 开源协议与再分发说明
waf文件中包含的文件（waf-light和waflib下的所有文件）以如下所示的BSD协议发布：  

      Redistribution and use in source and binary forms, with or without
      modification, are permitted provided that the following conditions
      are met:

      1. Redistributions of source code must retain the above copyright
         notice, this list of conditions and the following disclaimer.

      2. Redistributions in binary form must reproduce the above copyright
         notice, this list of conditions and the following disclaimer in the
         documentation and/or other materials provided with the distribution.

      3. The name of the author may not be used to endorse or promote products
         derived from this software without specific prior written permission.

      THIS SOFTWARE IS PROVIDED BY THE AUTHOR "AS IS" AND ANY EXPRESS OR
      IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
      WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
      DISCLAIMED. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT,
      INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
      (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
      SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
      HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
      STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING
      IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
      POSSIBILITY OF SUCH DAMAGE.

虽然此许可被认为十分自由，但版权声明的副本必须被包含在衍生品之中。为了消除关于此的任何疑问，这样的声明副本已经被默认添加到waf文件：使用文件编辑器打开waf文件并阅读第一行。

### 3.项目和命令
----

#### 3.1 Waf命令及用法

由于Waf是用于生成项目通用实用程序，因此具体项目的细节信息最好和源码保存在一起进行版本控制。这种文件是使用Python编写的模块，并且命名为wscript。虽然他们可以包含任何Python代码，但Waf使用其中定义的特定函数与类。接下来的部分我们将探讨一个特别有用的概念，被叫做函数命令。  

##### 3.1.1 命令行概述

Waf通常在被称为终端或shell的命令行解释器中运行，有三种方式传递数据至Waf进程以告诉它做某事：  

      $ CFLAGS=-O3 ❶ waf distclean configure ❷ -j1 --help ❸


❶ CFLAGS参数是一个环境变量; 他以一种未经检查的方式提供任意数据至进程  
❷	Waf被指示以特定顺序运行两个命令distclean 和 configure。命令在waf文件调用后给出，不包含-或=字符  
❸	-j1 与 --help 是命令行选项；它们是可选的，并且它们在参数列表中的位置或顺序并不重要。  

##### 3.1.2 Waf命令与Python函数之间的映射

使用Waf命令假定在项目中的wscript文件定义了相应的命令函数，wscript文件通常处于当前文件夹下。他们采用单个上下文（context）变量作为参数，并且无需返回任何特定值，如以下示例所示：  

      #! /usr/bin/env python
      # encoding: utf-8

      def hello(ctx):
          print('hello world')

使用命令来指示waf调用函数hello：  

      $ waf hello
      hello world
      'hello' finished successfully (0.001s)

上下文对象允许夸脚本进行数据共享，其用法将在下面部分描述。  

##### 3.1.3 Waf命令链

如前所述，命令按照命令行上的输入顺序执行。因此，一个wscript文件可以提供任意数量的命令在相同的wscript文件中：  

      def ping(ctx):
          print(' ping! %d' % id(ctx))

      def pong(ctx):
          print(' pong! %d' % id(ctx))

而且这样的命令可以通过在命令行上的重复被多次调用：  

      $ waf ping pong ping ping
       ping! 140704847272272
      'ping' finished successfully (0.001s)
       pong! 140704847271376
      'pong' finished successfully (0.001s)
       ping! 140704847272336
      'ping' finished successfully (0.001s)
       ping! 140704847272528
      'ping' finished successfully (0.001s)

当错误发生时，命令执行将被中断，并且不会进一步执行后续的命令。  
<table class="table table-bordered table-striped table-condensed"><tbody><tr>
<td width = "57px">
<img src="https://github.com/kklook/wafbook_cn/raw/master/book_image/note.png" alt="Note">
</td>
<td>
命令行函数被调用时会传递一个新的上下文对象；该对象的类对应于特定命令；ConfigureContext类用于configure命令，BuildContext类用于build命令，OptionContext类用于option命令，Context类用于其他的命令。</td>
</tr></tbody></table>

##### 3.1.4 基本的项目结构

虽然waf工程必须包含一个顶级的wscript文件，但它的内容可以划分成几个子项目文件。我们将在一个小的项目上说明这个概念：  

      $ tree
      |-- src
      |   `-- wscript
      `-- wscript

在顶层的wscript命令将通过调用名为recurse的上下文对象方法从子项目中调用相同的命令：  

```
def ping(ctx):
        print('→ ping from ' + ctx.path.abspath())
        ctx.recurse('src')
```

这里是src/wscript的内容  

```
def ping(ctx):
        print('→ ping from ' + ctx.path.abspath())
```

执行后，结果将会是：  

```
$ cd /tmp/execution_recurse

$ waf ping
→ ping from /tmp/execution_recurse
→ ping from /tmp/execution_recurse/src
'ping' finished successfully (0.002s)

$ cd src

$ waf ping
→ ping from /tmp/execution_recurse/src
'ping' finished successfully (0.001s)
```

<table class="table table-bordered table-striped table-condensed"><tbody><tr>
<td width = "57px">
<img src="https://github.com/kklook/wafbook_cn/raw/master/book_image/note.png" alt="Note">
</td>
<td>
该方法递归执行，并且path属性在所有waf上下文类中可用，以便所有waf命令可以使用他们。
</td>
</tr></tbody></table>  


#### 3.2 基本的Waf命令

以下部分提供了经常使用或在项目文件中重新实现的Waf命令的详细信息。  

##### 3.2.1 配置项目（configure命令）

尽管可以从包含wscript文件的任何文件夹调用Waf，但在特定项目文件中必须定义入口。这种做法提升了概括性并且提供了一个项目所有sub-wscript相同输入对应函数的重新定义。以下概念有助于构建一个Waf项目：  
1. 项目目录：这个目录包含将会打包并重新分发给其他开发人员或最终用户的源文件  
2. 生成目录：包含项目生成的文件的目录（配置集，生成文件，日志等）  
3. 系统文件：不属于项目的文件或文件夹（如操作系统文件... ...）  

名为configure的预定义命令用来收集和储存有关这些文件夹的信息。现在我们使用下面这个位于顶层文件夹的wscript文件来扩展上一节的示例：  

      top = '.' ❶
      out = 'build_directory' ❷

      def configure(ctx): ❸
              print('→ configuring the project in ' + ctx.path.abspath())

      def ping(ctx):
              print('→ ping from ' + ctx.path.abspath())
              ctx.recurse('src')

❶表示项目所在目录的字符串。一般来说。top设置为 . ，除非wscript不能加到项目的顶级目录中，top可以设置成 ../.. 或者形如 /checkout/perforce/project 的路径  
❷表示生成目录的字符串。一般设置为 build ，特殊情况下生成目录也可以设置成绝对路径，如 /tmp/build/ 。重要的是能够安全的删除生成目录。所以它不应该被设置成 . 或 ..  
❸configure函数由configure命令调用  
