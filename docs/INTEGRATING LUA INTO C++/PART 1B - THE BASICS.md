# INTEGRATING LUA INTO C++ (PART 1B: THE BASICS)

集成 Lua 到 C++（1B：基础）

This is Part 1b of my Integrating Lua into C++ Tutorial series.

- [Part 1a: Introduction and Setting Up (Current)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: The Basics](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: Interacting with Classes](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: Adding Templates to ScriptObject Class](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

Now that we have everything set up, let’s start!

本篇是我的《集成Lua到C++系列教程》的1b部分。

- [Part 1a: 介绍和安装(本篇)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: 基础](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: 类的交互](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: 为脚本对象添加模板](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

现在，我们已经安装好了一切。一起来愉快的玩耍吧！

## 1. Running a simple Lua-code from c++

First off, let’s try doing something simple. Let’s try running a Lua code from a string.

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_string("print(\'Hello World!\')");
}
```

**This should print out:**

```bash
Hello World!
```

Congratulations! You have run your first Lua-code in c++! As you can see, you don’t really need all these fancy ‘.lua’ files to run Lua-code. You can just use strings for the trivial stuffs.

You might be wondering what the code snippet above is doing.

`sol::state state;`

This line creates a lua state to be used in the codebase. Any functions or variables you define would ‘live in’ the state that it’s created in.

`state.open_libraries(sol::lib::base, sol::lib::package);`

This line loads up the default libraries that gives you certain functionalities of the Lua language. This includes I/O and other important functionalities. For our test case, we load up these libraries in order to run Lua’s print function.

If you are intending to create a sandbox for your game, you might not want to load up these libraries as you would essentially be giving your players too much power. You wouldn’t really want to give your modders functionalities to access I/O as it could lead to a security risk.

`state.do_string("print(\'Hello World!\')");`

This line would run the Lua code snippet from a string. As you might guess, looking at the code, this line would print “Hello World!” to the console window.

Now let’s try running the same code snippet from a ‘.lua’ file.

In the default directory of your project, create a ‘scripts’ folder to store all your scripts.

Create a file ‘HelloWorld.lua’ in the script folder and add this code inside:

```cpp
print('Hello World!')
```

Now try running this code in c++:

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/HelloWorld.lua");
}
```

`This should also print out:`

```bash
Hello World!
```

`state.do_file("scripts/HelloWorld.lua");`

You might notice the slight change of code here. Instead of do_script, we used do_file, to indicate that we want to run the code from a ‘.lua’ file instead of a string.

## 1. 在 C++ 层跑一个简单的 Lua 代码

开始之前，让我试着做些简单的事情。来，将字符串作为 Lua 代码跑起来。

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_string("print(\'Hello World!\')");
}
```

**它将输出：**

```bash
Hello World!
```

恭喜你！你已经在 C++ 层跑起来了第一个 Lua 代码！正如你所见，你不需要精心设计 .lua 文件就能跑 Lua 代码的。你通过字符串就能搞定这些琐碎的东西。

你可能会好奇上面的代码片段干了啥。

`sol::state state;`

这行创建了一个 lua table 用于代码底层。任何你定义的函数或变量都将创建并“存活”在这个 state 中。

`state.open_libraries(sol::lib::base, sol::lib::package);`

这行加载了默认库，用于提供 Lua 语言的特定功能，包括 I/O 和其他重要功能。在这个测试用例中，我们加载这些库以便于使用 Lua 的 print 函数。

如果你倾向于为你的游戏创建一个沙盒环境，你可能不想加载这些库，因为这些库本身给玩家提供了过多的权限。你并不想让你的模块化功能能够访问 I/O 从而埋下安全隐患。

`state.do_string("print(\'Hello World!\')");`

这行将字符串单做 Lua 代码片段执行。正如你可能猜到的，一看便知，这行会打印“Hello World!”到控制台窗口。

现在，让我们试着用 .lua 文件跑相同的代码。

在默认的项目路径（工作目录）中，创建一个 'script' 文件夹来保存你的脚本。

在脚本文件夹中，创建一个文件 ‘HelloWorld.lua’ 并写上：

```
print("Hello World!")
```

现在，让我们在 C++ 中跑起来：

```cpp
#include <sol.hpp>

int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/HelloWorld.lua");
}
```

**它也会输出：**

```bash
Hello World!
```

`state.do_file("scripts/HelloWorld.lua");`

你可能会注意到此处的微调。替换了 do_script，我们改用 do_file，表明我们期望从 .lua 文件来执行代码，而不是字符串。

## 2. Accessing Lua variables from C++

In this part of the tutorial, we will be covering on how to access lua variables in c++ code. 

Accessing lua variables from c++ is fairly simple.

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;  
  state.open_libraries(sol::lib::base, sol::lib::package);
  state["luaNum"] = 3;  
  state.do_string(std::string("print(\'printing in Lua: \' .. luaNum)");
  
  int luaNumValue = state["luaNum"];  
  std::cout << "printing in c++: " << luaNumValue << '\n';
} 
```

Here we have a numeric lua variable named ‘luaNum’. As you can see, in the snippet code above, we are able to both set and get the value just like a normal variable in c++ just by knowing the lua variable’s name.

**If nothing goes wrong, you’ll see:**

```bash
printing in Lua: 3
printing in c++: 3
```

`state["luaNum"] = 3;`

This line defines a numeric lua value with the variable name of “luaNum”. As stated above, every time you reference “luaNum” in your lua codebase, you would be referencing this variable.

`state.do_string(std::string("print(\'printing in Lua: \' .. luaNum)");`

This line prints the value of “luaNum”. This might be off topic but “..” is the concatenation operator for strings in lua.

## 2. 在 C++ 层访问 Lua 的变量

该部分，我们将探讨怎么在 C++ 层访问 Lua 变量的问题。

其实也相当简单。

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;	
  state.open_libraries(sol::lib::base, sol::lib::package);
  state["luaNum"] = 3;	
  state.do_string(std::string("print(\'printing in Lua: \' .. luaNum)");
  
  int luaNumValue = state["luaNum"];	
  std::cout << "printing in c++: " << luaNumValue << '\n';
} 
```

这里我们有个 number 类型的 Lua 变量，名字叫 'luaNum'。 如你所见，在上面的代码片段中，我们既能设置也能获取该值，就像是一个普通的 C++ 变量，仅仅只是把它用 Lua 变量名标记一样。

**如果没毛病的化，你会看到：**

```bash
printing in Lua: 3
printing in c++: 3
```

`state["luaNum"] = 3;`


这行定义了一个 number 类型的 Lua 变量，取名 'luaNum'。 正如声明那样，每次通过引用 'luaNum'，都能在 Lua 的底层代码中索引至该值。

`state.do_string(std::string("print(\'printing in Lua: \' .. luaNum)");`

该行打印了 'luaNum‘ 的值。也许有点跑题，但是 '..' 是 Lua 字符串的连接符还是要提一下。

## 3. Running c++ functions from Lua

We shall first try calling c++ functions from lua. Right now we’ll only run normal functions. We shall look at classes later in part 2 of the tutorial.

Since we already covered that whatever we can do in the ‘.lua’ files can also be done using strings, we shall just cover the topic using strings in order to simplify the tutorial.

Let’s run this code!

```cpp
#include <sol.hpp>
//this function simply takes in a number and returns it
int testCppFunction(int number)
{
  return number;
}
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  
  //the name of the function in lua is defined by the string
  state.set_function("testCppFunction", testCppFunction);
  
  //runs the lua script via string, calling the function and prints its return value
  state.do_string("num = testCppFunction(1); print(\"Called testCppFunction with return value(\" .. num .. \")!\")");
}
```

**If you run this code correctly, you should print out:**

```bash
Called testCppFunction with return value (1)!
```

`lua.set_function("testCppFunction", testCppFunction);`

This lines binds the c++ function into the Lua state with the alias “testCppFunction”

`state.do_string("num = testCppFunction(1); print(": Called testCppFunction! Return value:" .. num)");`

This line calls the testCppFunction by calling “testCppFunction(1)” in the string.

You can also do this with functional objects, lambda, etc. Try it out!

## 3. 在 Lua 层调用 C++ 函数

我们先学如何在 Lua 层调用 C++ 函数。现在，我们将仅调用普通的函数，对于类的函数将会放到教程的第二部分。

从我们探讨过 .lua 文件中能做的任何事情都可以通过字符串实现，我们就该想到通过使用字符串来简化教学。

来，跑跑这个代码！

```cpp
#include <sol.hpp>
// 该函数简单地接受一个 number 并将其返回
int testCppFunction(int number)
{
  return number;
}

int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  
  // lua中的函数，定义的时候需要用一个字符串表示其名字。
  state.set_function("testCppFunction", testCppFunction);
  
  // 通过字符串执行 Lua 脚本，脚本中调用了上面的函数并打印返回值。
  state.do_string("num = testCppFunction(1); print(\"Called testCppFunction with return value(\" .. num .. \")!\")");
}
```

**如果你正确执行该代码，你就会打印出:**

```bash
Called testCppFunction with return value (1)!
```

`lua.set_function("testCppFunction", testCppFunction);`

该行将 C++ 函数通过别名 'testCppFunction' 绑定到 Lua state 中。

`state.do_string("num = testCppFunction(1); print(": Called testCppFunction! Return value:" .. num)");`

该行通过字符串中的 'testCppFunction(1)' 调用了函数 testCppFunction。

你也能通过函数对象、lambda表达式等方式实现。自己去试吧。

## 4. Running Lua functions from c++

Before we do anything fancy like running the lua function from c++, let’s first run the code from a Lua file.

Create a Lua file called “testLuaFunction1.lua” in your scripts folder with the following code snippet:

```lua
function testLuaFunction1()
  print("Calling testLuaFunction1!")
end
```

If you did everything right, running this should print out “Calling testLuaFunction1!”.

In this lua file, we can see definition of the function testLuaFunction1 as well as a call to run the actual function.

Now let’s try running the Lua function from c++ code!

create a new lua file called ‘testLuaFunction2’ in your scripts folder with the following code snippet:

```lua
function testLuaFunction2(number)
  print("Calling testLuaFunction2 with input value (" .. number .. ")!")
  return number + 10
end
```

As you can see, the difference between this function and the previous function is that this function takes in a number and return said number + 10.

Alternatively, you can just remove the function call from ‘testLuaFunction1.lua’ and do all the tests by calling the function there.

There are multiple ways of calling a lua function from C++ code. Lets do the easiest way first. Im pretty sure that you already know how.

Well, you know the drill.

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/testLuaFunction2.lua");
  
  state.do_string("testLuaFunction2(1)");
}
```

**Running this code, you should get:**

```bash
Calling testLuaFunction2 with input value (1)!
```

In the string, we used ‘1’ as the argument and as such, we printed ‘1’ as the prefix in the output.

Now we shall encapsulate the lua function into a c++ object and call the function in c++.

This should be pretty useful for you. Let’s do this!

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  //define the function in the current lua state
  state.do_file("scripts/testLuaFunction2.lua");
  
  //initialize the c++ sol::function with the lua function previously defined in the current lua state.
  sol::function testLuaFunction2 = state["testLuaFunction2"];
  
  testLuaFunction2(2);
}
```

**Now you should get:**

```bash
Calling testLuaFunction2 with input value (2)!
```

`sol::function testLuaFunction2 = state["testLuaFunction2"];`

This sol::function is a functional object referencing to the lua function “testLuaFunction2” in the lua state. Invoking the function would invoke the referenced lua function.

Nice, isn’t it? Now you can have a functional object in c++ behave like and look like a c++ function but is a lua function.

If you’ve really been observant, you would had noticed that ‘testLuaFunction2’ returns a value, more specifically the input value + 10.

Let’s try printing the return value of the function.

```cpp
#include <sol.hpp>
#include <iostream>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  //define the function in the current lua state
  state.do_file("scripts/testLuaFunction2.lua");
  int ret = state.do_string("return testLuaFunction2(1)");
  std::cout << "Return value: " + std::to_string(ret) << std::endl;
  sol::function testLuaFunction2 = state["testLuaFunction2"];
  ret = testLuaFunction2(2);
  std::cout << "Return value: " + std::to_string(ret) << std::endl;
}
```

**Now you should get:**

```bash
Calling testLuaFunction2 with input value (1)!
Return value: 11
Calling testLuaFunction2 with input value (2)!
Return value: 12
```

## 4. 在 C++ 层调用 Lua 函数

在我们进行一些骚操作（如在 C++ 层执行 Lua 函数）之前，让我们先用 Lua 文件跑代码。

在脚本文件夹中创建 Lua 文件 'testLuaFunction1.lua'，填入下面代码片段：

```lua
function testLuaFunction1()
  print("Calling testLuaFunction1!")
end
```

如果一切顺利的化，执行这段代码将打印 “Calling testLuaFunction1!”.

在这个 Lua 文件中，我们能看到 testLuaFunction1 的函数定义就像是个真实函数（C++函数）一样。

现在让我们在 C++ 代码中调用该 Lua 函数！

在脚本目录中创建一个新的文件 'testLuaFunction2'，填入以下代码片段：

```lua
function testLuaFunction2(number)
  print("Calling testLuaFunction2 with input value (" .. number .. ")!")
  return number + 10
end
```

如你所见，该函数与之前的函数区别在于多了一个 number 参数，并返回 number + 10。

当然，你也可以不去调用 '‘testLuaFunction1.lua' 中的函数，而调用这一个带参数的。

这就是 C++ 层的多种途径调用 Lua 函数。先试试最简单的方式。我敢相信你已经知道该怎么做了。

好吧，你知道这个演示。

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/testLuaFunction2.lua");
  
  state.do_string("testLuaFunction2(1)");
}
```

**执行这个代码，你会得到:**

```bash
Calling testLuaFunction2 with input value (1)!
```

在字符串中，我们使用 '1' 作为参数，并且最终打印出了 '1' 作为输出的前缀。

现在，我们要将 Lua 函数封装到 C++ 对象中，并在 C++ 层调用。

这段代码会很有用。来试试看。

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);

  // 在当前 lua state 中 定义函数
  state.do_file("scripts/testLuaFunction2.lua");
  
  // 通过先前在 lua state 中定义的 Lua 函数 初始化 C++ sol::function
  sol::function testLuaFunction2 = state["testLuaFunction2"];
  
  testLuaFunction2(2);
}
```

**现在你将得到:**

```bash
Calling testLuaFunction2 with input value (2)!
```

`sol::function testLuaFunction2 = state["testLuaFunction2"];`

这个 sol::function 是个函数对象，它指向 lua state 中的 Lua 函数 'testLuaFunction2'。调用它就会调用它引用着的 Lua 函数。

很不错，不是么？现在你拥有了一个 C++ 层的函数对象，它看起来就像个 C++ 函数，但是实际上是 Lua 函数。

如果你非常善于观察，会发现 ‘testLuaFunction2’ 有返回值，具体来讲，是返回了 输入参数 + 10。

我们来把这个返回值打印出来。

```cpp
#include <sol.hpp>
#include <iostream>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);

  // 在当前 lua state 中 定义函数
  state.do_file("scripts/testLuaFunction2.lua");
  int ret = state.do_string("return testLuaFunction2(1)");
  std::cout << "Return value: " + std::to_string(ret) << std::endl;
  sol::function testLuaFunction2 = state["testLuaFunction2"];
  ret = testLuaFunction2(2);
  std::cout << "Return value: " + std::to_string(ret) << std::endl;
}
```

**现在你将得到:**

```bash
Calling testLuaFunction2 with input value (1)!
Return value: 11
Calling testLuaFunction2 with input value (2)!
Return value: 12
```

## 5. Summary

This part of the tutorial is a lot longer than I wanted it to be. It’s also not so interesting but well, it’s a necessary evil as you need the knowledge here before you can continue on to the next part.

## 5. 总结

本篇教程的长度有点超预期。也不好玩，但是在进入下一部分的学习之前，还是很有必要了解的。

## Codes

main.cpp

```cpp
#include <sol.hpp>
#include <iostream>

void test1()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_string("print(\'Hello World!\')");
}
void test2()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/HelloWorld.lua");
}
void test3()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state["luaNum"] = 3;
  state.do_string("print(\'printing in Lua: \' .. luaNum)");
  int luaNumValue = state["luaNum"];
  std::cout << "printing in c++: " << luaNumValue << '\n';
}

int testCppFunction(int number)
{
  return number;
}
void test4()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.set_function("testCppFunction", testCppFunction);
  state.do_string("num = testCppFunction(1); print(\"Called testCppFunction with return value(\" .. num .. \")!\")");
}

void test5()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/testLuaFunction1.lua");
}
void test6()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/testLuaFunction2.lua");
  state.do_string("testLuaFunction2(1)");
}
void test7()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/testLuaFunction2.lua");
  sol::function testLuaFunction2 = state["testLuaFunction2"];
  testLuaFunction2(2);
}
void test8()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.do_file("scripts/testLuaFunction2.lua");
  int ret = state.do_string("return testLuaFunction2(1)");
  std::cout << "Return value: " + std::to_string(ret) << std::endl; 
  sol::function testLuaFunction2 = state["testLuaFunction2"];
  ret = testLuaFunction2(2);
  std::cout << "Return value: " + std::to_string(ret) << std::endl;
}

int main()
{
  //Change value of testNum to change to the different test-cases
  int testNum = 1;
  switch(testNum)
  {
  case 1:
    test1();
    break;
  case 2:
    test2();
    break;
  case 3:
    test3();
    break;
  case 4:
    test4();
    break;
  case 5:
    test5();
    break;
  case 6:
    test6();
    break;
  case 7:
    test7();
    break;
  case 8:
    test8();
    break;
  };
}
```

HelloWorld.lua

```lua
print('Hello World!')
```

testLuaFunction1.lua

```lua
function testLuaFunction1()
  print('Calling testLuaFunction1()!')
end
testLuaFunction1()
```

testLuaFunction2.lua

```lua
function testLuaFunction2(number)
  print("Calling testLuaFunction2 with input value (" .. number .. ")!")
  return number + 10
end
```