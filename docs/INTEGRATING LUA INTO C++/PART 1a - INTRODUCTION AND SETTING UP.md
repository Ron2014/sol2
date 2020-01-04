# INTEGRATING LUA INTO C++ (PART 1A: INTRODUCTION AND SETTING UP)

集成 Lua 到 C++（1A：介绍和安装）

This is Part 1a of my Integrating Lua into C++ Tutorial series.

- [Part 1a: Introduction and Setting Up (Current)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: The Basics](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: Interacting with Classes](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: Adding Templates to ScriptObject Class](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

There are plenty of scripting languages to work with when binding with C++. Everyone have their personal favorite and mine is Lua.

There are plenty of great tutorials out there that teaches on how to integrate C++ with Lua but I feel that those tutorials are kind of outdated. I feel that there are better ways to do things in 2018. As such, this tutorial counts on utilizing more updated libraries compared to most tutorials out there.

本篇是我的《集成Lua到C++系列教程》的1a部分。

- [Part 1a: 介绍和安装(本篇)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: 基础](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: 类的交互](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: 为脚本对象添加模板](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

如今已经有了很多可以绑定到C++工作的脚本语言。每个人都会有他的个人倾向，我呢就是Lua。

如今也已经有了很多牛逼的教程教大家如何集成 C++ 和 Lua，但是我感觉这些教程都有点过时了。到了2018年，我感觉有了更好的方法来做这样的工作。像这样，本系列教程区别域其他教程的地方，就在于依赖于使用更新版本的库。

## 1. Why use a scripting language?

Source codes are usually compiled at compile-time while scripting languages are mostly compiled at run-time. As a game gets bigger, one might notice that compiling the entire code-base would take a long time, which results to a huge productivity hit.

As such, being able to tweak game logic without the need to recompile the code-base would result in a significant productivity boost. There are always people out there complaining about the time it takes to compile a code-base.

Some scripting languages, such as Lua, is also easy to learn. This means that non-programmers would be able to provide gameplay behaviors without having access to the main source codes.

With having an environment for the non-programmers to code (e.g. your scripting language), the programmers can effectively constraint the working environment of the non-programmers, for better or for worse. This ensures that the non-programmers are not able to overstep some boundaries, and write game breaking code out of the context that they were supposed to work in.

Scripting languages also makes it super easy to add modding support to your game. You can easily allow your players to modify gameplay features by giving them limited access to your scripting platform. For example, look at [Binding of Isaac](https://bindingofisaacrebirth.gamepedia.com/Modding_Tutorials).

Finally, being able to refresh your code at a press of a button during run-time is really really neat.

## 1. 为啥要使用脚本语言？

源码通常在编译期编译，而脚本语言在运行时编译。当一个游戏工程的体谅越来越大，有人或许会注意到编译这整个代码库会占用很长的时间，这构成了大规模生产的痛点。

像这样，搞定游戏逻辑和代码库的重编译的需要性，对于推动生产力有重大意义。也总有人抱怨代码库的编译时间太长。

一些脚本语言，例如 Lua，也简单易学。这意味着非程序员也能编写一些玩法表现，哪怕他没有主体源码的访问权限。

非程序员拥有了编码环境（例如你的脚本语言）后，程序员就能有效地约束这个环境，无论他的做法是好是坏。这种特点保证了非程序员无法跨越某些边界，去越过支持他们工作的上下文编写破坏性代码。（比如栈溢出，指针越界;)）

脚本语言同样让你可以通过炒鸡简单的方式，为你的游戏添加模块。你能通过开放一些有限的脚本平台访问接口，让玩家修改玩法特性。举个例子，看看[Isaac的绑定](https://bindingofisaacrebirth.gamepedia.com/Modding_Tutorials)。

最后，在运行时通过点击某个按钮来实现代码的刷新（热更新）真是灰常灰常优雅。

## 2. Why should I not use a scripting language?

Debugging for scripting languages is a big pain in the ass, unless you are going to make an IDE for your scripting language. It is sadly not as easy to debug as compared to using a host language (like c++), which have so many fancy tools (visual studio, etc.)

The major turn-off for most will be the performance. It is a main issue that scripting languages will always be slower compared to it’s source code equivalent. If speed is a critical must, then it’s better to run everything on the main source code.

## 2. 为啥我不只是用脚本语言？

在脚本语言中调试常常让人菊花一紧（a big pain in the ass 还能是什么？便秘，拉不出，一大坨恶心的东东;)），除非你打算为脚本语言做个IDE。让人忧伤的是，它的调试没法跟一些热门语言（如C++）相比，这些语言都配备很多精心设计的工具（visual studio 等等）。

一个项目，让大多数人失去兴趣的主要因素是其性能。主要问题是，脚本语言通常要比支持它的语言要慢。如果速度是大家追求的关键点，最好的方式是让所有的东西都在主体源码上跑。

## 3. Some prerequisites before we start.

一些开始前的必要准备。

### 3.1 Sol2 Lua Binding Library for C++

Sol2 is a C++ library binding to Lua. It is really easy to set up, and easy to use.

There are plenty of tutorials out there on the github page. If you would like to know more, I would recommend you to go to the github page and follow the tutorials there for more grasp on the topic. There are plenty of great tutorials there on how to use the library.

Here’s the link!

### 3.1 Sol2：绑定 Lua 的 C++ 库

Sol2 是一个绑定 Lua 的 C++ 库. 它的安装相当简单，使用也简单。

Github page上也有了大量的教程。如果你想知道更多，我推荐你去 Github page 上逛逛，走一遍上面的教程会更加领会相关的概念。那里也有好多使用该库的教程。

这里是[传送门](https://github.com/ThePhD/sol2)!

### 3.2 Lua Library

You’re going to need download the Lua library. You can go to the Lua website and download the library. There’s no need to compile the library. Just download the pre-compiled binaries.

Go to Download -> Binaries

and get the version you need.

Here’s the link!

### 3.2 Lua 库

你需要下载 Lua 库。你可以去 Lua 官网下载这个库。也没有必要编译这个库。仅仅下载预编译版的二进制格式即可。

通过网页导航 Download -> Binaries

然后拿你想要的版本。

这里是[传送门](https://www.lua.org/)!

### 3.3 Middleclass Library

This is an OOP library for Lua.  It is very lightweight, easy to use, and to set up.

The github page have really good example that shows everything you need to know about using the library. Take a look at the page to see how to use the library out of the scope of the tutorial.

Here’s the link!

### 3.3 类库中间件

这是个 Lua 的面向对象库。它非常轻量，安装相当简单，使用也简单。

Github page 上已经有了相当优秀的示例教你如何使用该库。去看看在本教程的目标外是如何使用这个库的。

这里是[传送门](https://github.com/kikito/middleclass)!