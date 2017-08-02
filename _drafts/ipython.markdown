---
type: ipythonNotebook
date: 2017年5月26日16:01:43
version: '1.0'
---

[ipythonNotebook](https://jupyter.readthedocs.io/en/latest/architecture/how_jupyter_ipython_work.html)

***

# ipython jupyter Notebook 怎么一起工作的

# 介绍
我们在介绍ipython jupyter Notebook 怎么一起工作的前提
  <!-- 1. ipython 终端跟 repl很相似 -->
  2. ipython 内核提供了 计算和前端交流的接口就如 notebook
jupyter notebook 有他的灵活性 并继承了 notebook 不仅仅有代码可视化,多媒体,协作

## 终端ipython
 当键入ipython,你就获得了原始的ipython接口,如下:

 ```
 while True:
    code = input(">>> ")
    exec(code)
 ```
 当然,他还是比较复杂,因为他不得不处理多行代码,tab补齐,使用readline,模糊匹配等等.但是模型就像代码例子:
 提示用户输入一些代码,当他输入了,就执行一样的进程.这个模型就叫做REPL,或者是Read-Eval-Print-Loop.
 ## ipython内核

![ ipython内核](https://jupyter.readthedocs.io/en/latest/_images/ipy_kernel_and_terminal.png)

所有其他的接口如notebook/Qt console,ipython console 在终端或者第三方接口,都使用ipython内核.Ipython内核是一个隔离的进程负责运行用户代码,并且计算可能的代码.前端,正如notebook或者Qt Console 他们与Ipython 内核通过json信息交流在zeroMQ socket上.这个协议使用在前端和ipython内核,正如消息层在jupyter.

ipython kernel的 python执行器 与 Terminal Ipython执行器共享python execution的内核.

![内核两种实现](https://jupyter.readthedocs.io/en/latest/_images/other_kernels.png)


内核线程同时可以连接超过一个前端.这样的情况下,不同的前端将有访问权限访问到同样的变量.

这种设计方便开发基于一样内核的不同前端但是很难在同时的前端支持一种新语言.与其用开发一种新内核来脏支持新语言,我我们精致了一个Ipython让其变得可能.

今天,对一种新语言,有两种方法去开发一个内核.
1. 优点
  1. 利用ipython,包装内核重新利用了他的通信机制,仅仅实现核心执行部分.
  2. 为目标语音重新进行原生内核实现执行/通信机制
  3. *包装内核很简单快速的支持一种新语言,像octave_kernel/语言像bash_kernel不切实际的实现交流机制*
1. native kernel
  4. 原生内核像ijulia/iHaskell,容易被社区维护.

## notebooks
![笔记服务器](https://jupyter.readthedocs.io/en/latest/_images/notebook_components.png)
1. notebook 前端做了一些额外的事情.除此之外,在跑你的代码期间,他存储了你的代码和输出,和markdown notebook一样,在一个可以编辑的文档傻姑娘叫做笔记本.当你保存他,就会从浏览器发送到 notebook的服务器,同时会以.ipynb的jsonfile格式被保在硬盘上.
2. 笔记服务器,不是内核,负责保存和加载笔记,所以你可以编辑笔记即使你没有那种语言的kernel,你仅仅将担心他是否能运行你的代码.内核并不关心关于notebook的任何事情,他仅仅将送入的数据执行,当用户点击run.

## 导出笔记本到另外一种格式
![笔记导出](https://jupyter.readthedocs.io/en/latest/_images/nbconvert.png)
在jupyter中笔记转换工具将笔记转换为其他的格式,比如html/latex/重新结构化的文本.这种转换公国一系列的步骤:
1. 预处理在内存中修改笔记.例如:在代码中执行预处理且更新输出.
2. 用导出工具将笔记导出为另一种格式.大部分的导出使用模板.
3. 投递处理器在导出文件之后开始工作.
在笔记浏览网站使用笔记转换采用html 导出.当你给出一个url,他就从这个url中获取一个notebook,并且转换为一个html,且展示那个html给你

## ipython并行
ipython也包括并行计算框架.Ipython.parallel.他允许你控制许多单独的引擎,他们都在ipython的内核描述中.
