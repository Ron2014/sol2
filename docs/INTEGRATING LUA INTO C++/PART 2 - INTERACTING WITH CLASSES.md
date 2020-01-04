# INTEGRATING LUA INTO C++ (PART 2: INTERACTING WITH CLASSES)

集成 Lua 到 C++（2：类的交互）

This is Part 2 of my Integrating Lua into C++ Tutorial series.

- [Part 1a: Introduction and Setting Up (Current)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: The Basics](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: Interacting with Classes](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: Adding Templates to ScriptObject Class](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

For this part of the tutorial series, we will be discussing on how to interact with classes and objects, both in Lua and C++.

本篇是我的《集成Lua到C++系列教程》的2部分。

- [Part 1a: 介绍和安装(本篇)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: 基础](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: 类的交互](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: 为脚本对象添加模板](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

本篇我们将讨论如何与类额对象进行互动，不管是在 Lua 层还是 C++ 层。

## 1. Using a C++ class in Lua

First off, let’s bind a simple C++ class so that we can use the class in our lua code. You can name your class anything, but I’m going to use something simple, ‘CppObject’.

Create a class with the following interface and implementation:

CppObject.hpp

```cpp
#pragma once
#include <iostream>
class CppObject
{
  void TestFunction1()
  {
    std::cout << "TestFunction1 Called!" << std::endl;
  }
};
```

This is a simple c++ class with a function that prints out “Test1Function1 Called!” when called.

```cpp
#include <sol.hpp>
#include "CppObject.hpp"
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.new_usertype<CppObject>("CppObject",
    "TestFunction1", &CppObject::TestFunction1
    );
  state.do_string("cppObj = CppObject:new() cppObj:TestFunction1()");
}
```

This code helps to bind the C++ object into a Lua object with the alias ‘CppObject.

Lets dissect the code line by line.

`state.new_usertype<CppObject>("CppObject", "TestFunction1", &CppObject::TestFunction1);`

This line defines the new Lua Usertype and also defines the member function ‘TestFunction1’ to specify that the class contains this function.

`state.do_string("cppObj = CppObject:new() cppObj:TestFunction1()");`

In this line, we create a new object ‘cppObj’ of type ‘CppObject’ and invoked it’s member function.

**If all goes well, you should printed:**

```bash
TestFunction1 Called!
```

Congratulations! You now know how to play around with C++ Objects in your lua code. If you are really interested in learning more about this, The tutorials over in the Sol2 github page really covers this extensively.

## 1. 在 Lua 层使用 C++ 类

开始之前，我们来绑定一个简单的 C++ 类，让它可以在 Lua 代码中被访问。你可以给你的类起任何名字，但是我打算起简单一点，'CppObject'。

使用如下接口和实现来创建一个类。

CppObject.hpp

```cpp
#pragma once
#include <iostream>
class CppObject
{
  void TestFunction1()
  {
    std::cout << "TestFunction1 Called!" << std::endl;
  }
};
```

这是个简单的 C++ 类，只有一个函数，调用时打印 “Test1Function1 Called!”。

```cpp
#include <sol.hpp>
#include "CppObject.hpp"
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  state.new_usertype<CppObject>("CppObject",
    "TestFunction1", &CppObject::TestFunction1
    );
  state.do_string("cppObj = CppObject:new() cppObj:TestFunction1()");
}
```

这段代码将 C++ 对象绑定到 Lua 对象上，还起了个别名 'CppObject'。

让我们来一行行剖析。

`state.new_usertype<CppObject>("CppObject", "TestFunction1", &CppObject::TestFunction1);`

这行定义了一个新的 Lua Usertype，还定义了它的成员函数 'TestFunction1'，将其指定为类的同名函数。

`state.do_string("cppObj = CppObject:new() cppObj:TestFunction1()");`

在这行，我们通过类型 'CppObject' 创建了一个新对象 'cppObj'，并调用它的成员函数。

**顺利的化，你会打印:**

```bash
TestFunction1 Called!
```

恭喜！你现在知道怎么在 Lua 代码中玩 C++ 对象了。如果你想要了解更多，在 [Sol2 github page](https://github.com/ThePhD/sol2) 上的教程会提供这些扩展知识。

## 2. Using a Lua class in Lua

Before we go over using a Lua class in C++, let’s first do it in Lua.

The Lua language does not have any implementation of classes but they do have something called metatable that allows you to do really cool stuffs. One of them is to implement a class system. For this, we will be using a library called middleclass. Middleclass is an OOP library that I really like to use. It’s very lightweight, and easy to use.

I will only be covering the basics, so head over to their github page for more info.

For starters, download the middleclass file and then add it into your scripts folder. I’ll show you how to set it up later.

For now, go to your scripts folder and create a new lua file with the following code:

TestClass.lua

```lua
local class = require 'middleclass'
TestClass = class('TestClass')
function TestClass:initialize()
  print('TestClass Created! ')
end
function TestClass:TestFunctionCall()
  print('TestClass TestFunction Called!')
end
```

The code above defines the class that we will be using, ‘TestClass’.

The initialize function will be called whenever a TestClass object is created. You can say that it’s the constructor of the class.

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];
  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
  state.do_file("scripts/TestClass.lua");
  state.do_string("testObj = TestClass:new() testObj.TestFunctionCall()");
}
state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
```

Here, I initialized the middleclass library by adding it into the luastate”s package path.

`state.do_file("scripts/TestClass.lua");`

I then ran the ‘TestClass.lua’ file that defines my TestClass Lua class.

`state.do_string("testObj = TestClass:new() testObj.TestFunctionCall()");`

I then created a TestClass object and called it’s member function ‘TestFunctionCall’

**If everything went well, you should be printing this on the console:**

```bash
TestClass Created!
TestClass TestFunction Called!
```

This is just a basic functionality for using MiddleClass together with Sol2. I hope you find it useful.

## 2. 在 Lua 层使用 Lua 类

在 C++ 层使用 Lua 类之前，先在 Lua 层做这些看看。

Lua 语言本身未实现类机制，但是它提供了一种叫 metatable 的东西允许你进行一些骚操作。其中之一就是实现类机制。因此，我们需要用上 middleclass 库。 Middleclass 是个面向对象库，我很喜欢用它。它非常轻量，而且使用方便。

我仅提供最基础的实现，所以要去它们的 github page 查更多信息。

新手们，下载 middleclass 文件并添加到你们的脚本目录中。我待会告诉你怎么设置它。

现在要做的，是进入你的脚本目录，创建一个新 Lua 文件，填入一下代码：

TestClass.lua

```lua
local class = require 'middleclass'
TestClass = class('TestClass')
function TestClass:initialize()
  print('TestClass Created! ')
end
function TestClass:TestFunctionCall()
  print('TestClass TestFunction Called!')
end
```

上面的代码定义了一个 'TestClass' 类，我们待会会用到。

initialize 函数会在 TestClass 对象的时候被撞见。你可以把它当成是类的构造函数。

```cpp
#include <sol.hpp>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];
  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
  state.do_file("scripts/TestClass.lua");
  state.do_string("testObj = TestClass:new() testObj.TestFunctionCall()");
}
state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
```

这里，我将 middleclass.lua 文件添加到 lua state 的 package 路径中，从而初始化这个库。

`state.do_file("scripts/TestClass.lua");`

然后我执行 TestClass.lua 文件，定义了 Lua 类 TestClass。

`state.do_string("testObj = TestClass:new() testObj.TestFunctionCall()");`

然后我创建了 TestClass 对象，并调用其成员函数 TestFunctionCall。

**顺利的话，你会在控制台看到如下打印**

```bash
TestClass Created!
TestClass TestFunction Called!
```

对于 MiddleClass 和 Sol2 来说，这只是个很基础的功能。希望你觉得有用。

## 3. Using a Lua class in C++

Now we get to the more interesting part of the tutorial, using a lua object in c++. I find that the best way for this is to create a wrapper class.

You actually do not need to create a wrapper class like ScriptObject as you can just save it into a sol::table but I would recommend making one to make your life easier.

ScriptObject.hpp

```cpp
#include <sol.hpp>
class ScriptObject
{
public:
  explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName)
    :
    m_pLuaState(pLuaState),
    m_initialized(false)
  {
    if (m_pLuaState)
    {
      static std::size_t scriptNum = 0;
      m_scriptVarName = luaClassName + "_" + std::to_string(scriptNum++);
      bool isValidCreation = true;
      std::string luaScript = m_scriptVarName + " = " + luaClassName + ":new()";
      m_pLuaState->script(luaScript,
        [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });
      if (isValidCreation)
        m_luaObjectData = (*m_pLuaState)[m_scriptVarName];
      m_initialized = isValidCreation && m_luaObjectData.valid();
    }
  }
  void CallFunction(const std::string &fnName)
  {
    if (m_initialized)
      m_luaObjectData[fnName]();
  }
  ~ScriptObject()
  {
    if (m_initialized && m_pLuaState)
      m_pLuaState->do_string(m_scriptVarName + " = nil");
  }
private:
  std::string m_scriptVarName; // The name of the lua object in the lua state
  sol::table m_luaObjectData;  // A variable holding the data of the created lua object
  sol::state *m_pLuaState;     // The lua state the lua object is created
  bool m_initialized;          // whether the scriptObject is initialized
};
```

Whew this is rather long. This class omits out quite a few functions for it to be usable, but it contains the bare-bones of what we need for the tutorial. I wanted to keep it simple. Don’t worry though, I’ll share the full class implementation later.

```cpp
explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName)
  :
  m_pLuaState(pLuaState),
  m_initialized(false)
{
  if (m_pLuaState)
  {
    static std::size_t scriptNum = 0;
    m_scriptVarName = luaClassName + "_" + std::to_string(scriptNum++);
    bool isValidCreation = true;
    std::string luaScript = m_scriptVarName + " = " + luaClassName + ":new()";
    m_pLuaState->script(luaScript,
      [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });
    if (isValidCreation)
      m_luaObjectData = (*m_pLuaState)[m_scriptVarName];
    m_initialized = isValidCreation && m_luaObjectData.valid();
  }
}
```

This is the non-default constructor of the ScriptObject.

This function creates a Lua object of type ‘luaClassName’ and gives them a unique name for use in the luaState. The name is then saved in m_scriptVarName.

`m_luaObjectData = (*m_pLuaState)[m_scriptVarName];`

If you remember the previous tutorial, then this should be rather familiar. Here, we save the lua object into m_luaObjectData for easy access.

`m_pLuaState->script(luaScript, [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });`

The most complex part about this function would most probably be in this line. I created this lambda function to suppress an exception being thrown if the lua code called is not valid. In this case, it would usually be called if the Lua-class of the object you are trying to create is not valid.

If an exception is thrown, we will set isValidCreation to false, and then stop with the creation of the object.

If m_initialized boolean is set to true at the end of the function, it would mean that your object is successfully created.

```cpp
void CallFunction(const std::string &fnName)
{
  if (m_initialized)
    m_luaObjectData[fnName]();
}
```

This helper function helps us call the lua object’s member functions.

```cpp
~ScriptObject()
{
  if (m_initialized && m_pLuaState)
    m_pLuaState->do_string(m_scriptVarName + " = nil");
}
```

In the destructor, we set the object to nil and then let the garbage collection do all the work.

Alright, so we have already defined our ScriptObject wrapper class. So now let’s try using it.

```cpp
#include <sol.hpp>
#include "ScriptObject.hpp"
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];
  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
  state.do_file("scripts/TestClass.lua");
  ScriptObject obj(&state, "TestClass");
  obj.CallFunction("TestFunctionCall");
}
```

So here we create our ScriptObject object and then call the “TestFunctionCall” function. The behaviour is totally the same as the previous example.

**If everything went well, same as before, you should be printing this on the console:**

```bash
TestClass Created!
TestClass TestFunction Called!
```

## 3. 在 C++ 层使用 Lua 类

现在我们进入到了教学相当有趣的部分，在 C++ 层使用 Lua 对象。我发现实现它的最好方式是写个包装类。

实际上你也没有必要像 ScriptObject 那样创建包装类，再将它保存成一个 sol::table。但是我还是推荐你做一个，来让生活敞亮一点。

ScriptObject.hpp

```cpp
#include <sol.hpp>
class ScriptObject
{
public:
  explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName)
    :
    m_pLuaState(pLuaState),
    m_initialized(false)
  {
    if (m_pLuaState)
    {
      static std::size_t scriptNum = 0;
      m_scriptVarName = luaClassName + "_" + std::to_string(scriptNum++);
      bool isValidCreation = true;
      std::string luaScript = m_scriptVarName + " = " + luaClassName + ":new()";
      m_pLuaState->script(luaScript,
        [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });
      if (isValidCreation)
        m_luaObjectData = (*m_pLuaState)[m_scriptVarName];
      m_initialized = isValidCreation && m_luaObjectData.valid();
    }
  }
  void CallFunction(const std::string &fnName)
  {
    if (m_initialized)
      m_luaObjectData[fnName]();
  }
  ~ScriptObject()
  {
    if (m_initialized && m_pLuaState)
      m_pLuaState->do_string(m_scriptVarName + " = nil");
  }
private:
  std::string m_scriptVarName; // lua state 中 lua 对象的名字
  sol::table m_luaObjectData;  // lua 对象的引用
  sol::state *m_pLuaState;     // lua state
  bool m_initialized;          // scriptObject 是否被初始化的标记
};
```

好吧，它有点长。这个类忽略掉了一部分有用的的函数，但是它用最精简的方式包含了教学需要的东西。我期望它能简单。别担心，待会我会展示它的完整实现（Codes那节有完整代码）。

```cpp
explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName)
  :
  m_pLuaState(pLuaState),
  m_initialized(false)
{
  if (m_pLuaState)
  {
    static std::size_t scriptNum = 0;
    m_scriptVarName = luaClassName + "_" + std::to_string(scriptNum++);
    bool isValidCreation = true;
    std::string luaScript = m_scriptVarName + " = " + luaClassName + ":new()";
    m_pLuaState->script(luaScript,
      [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });
    if (isValidCreation)
      m_luaObjectData = (*m_pLuaState)[m_scriptVarName];
    m_initialized = isValidCreation && m_luaObjectData.valid();
  }
}
```

这是个 ScriptObject 的非默认构造函数。

该函数通过字符串 'luaClassName' 来指定 Lua 类的名称，通过这个类实例化一个 Lua 对象，并起了一个唯一的名字，改名字保存在 m_scriptVarName 中。

`m_luaObjectData = (*m_pLuaState)[m_scriptVarName];`

如果你还记得之前的教程，你会对这段相当熟悉了。在这里，我们将 Lua 对象通过 C++ 变量 m_luaObjectData 引用起来，方便日后访问。

`m_pLuaState->script(luaScript, [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });`

这行怕是这个函数最复杂的地方了。我用了 lambda 函数，其作用是在字符串 luaScript 表示的 Lua 代码发生语法错误的时候，防止异常的抛出。在这种情况下，我们通常会怀疑这个 Lua 类的实例化相关的代码写错了。

如果异常抛出了，我们会把 isValidCreation 赋值为 false，并且停止创建该对象。

如果函数结束后，布尔值 m_initialized 最终为 true，就意味着对象成功创建了。

```cpp
void CallFunction(const std::string &fnName)
{
  if (m_initialized)
    m_luaObjectData[fnName]();
}
```

这个辅助函数帮我们调用 Lua 对象内部的成员函数。

```cpp
~ScriptObject()
{
  if (m_initialized && m_pLuaState)
    m_pLuaState->do_string(m_scriptVarName + " = nil");
}
```

析构函数中，我们将引用 Lua 对象的变量设为 nil，使得垃圾收集器有事可干。

好了，所以我们已经定义好了 ScriptObject 包装类，现在把它用起来吧。

```cpp
#include <sol.hpp>
#include "ScriptObject.hpp"
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];
  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
  state.do_file("scripts/TestClass.lua");
  ScriptObject obj(&state, "TestClass");
  obj.CallFunction("TestFunctionCall");
}
```

这里，我们创建了 ScriptObject 对象，并调用了 'TestFunctionCall' 函数。其行为正如之前的示例一样。

**顺利的话，我们会想之前教程那样，在控制台打印出:**

```bash
TestClass Created!
TestClass TestFunction Called!
```

## 4. Summary

This part of the tutorial is where you start having the basic knowledge on OOP on Lua, as well as handling classes in both Lua and C++ codes.

## 4. 总结

这个部分的教学包含 Lua 层的面向对象实现这样的基础知识，它和我们主要要讲的在 Lua 和 C++ 代码中处理各种类同等重要。

## Codes

CppObject.hpp

```cpp
#pragma once
#include <iostream>

class CppObject
{
public:
  void TestFunction1()
  {
    std::cout << "TestFunction1 Called!" << std::endl;
  }
};
```

ScriptObject.hpp

```cpp
class ScriptObject
{
public:
  ScriptObject() = default;
  explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName);
  ScriptObject(const ScriptObject&other) = delete;
  ScriptObject& operator=(const ScriptObject&other) = delete;
  ~ScriptObject();

  bool Init(sol::state *pLuaState, const std::string &luaClassName);

  bool HasFunction(const std::string &fnName);
  void CallFunction(const std::string &fnName);
private:
  std::string m_luaClassName;
  std::string m_scriptVarName;
  sol::table m_luaObjectData;
  sol::state *m_pLuaState;
  static std::size_t ms_scriptNum;
  bool m_initialized;
};
```

ScriptObject.cpp

```cpp
std::size_t ScriptObject::ms_scriptNum = 0;

ScriptObject::ScriptObject(sol::state *pLuaState, const std::string &luaClassName)
  :
  m_pLuaState(nullptr),
  m_initialized(false)
{
  Init(pLuaState, luaClassName);
}
ScriptObject::~ScriptObject()
{
  if (m_pLuaState)
    m_pLuaState->script(m_scriptVarName + " = nil");
}

bool ScriptObject::Init(sol::state *pLuaState, const std::string &luaClassName)
{
  m_pLuaState = pLuaState;
  m_initialized = true;
  m_luaClassName = luaClassName;

  m_scriptVarName = luaClassName + "_" + std::to_string(ms_scriptNum++);

  bool isValidCreation = true;
  std::string script = m_scriptVarName + " = " + m_luaClassName + ":new()";
  m_pLuaState->script(script.c_str(),
    [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });

  if (isValidCreation)
  {
    m_luaObjectData = (*m_pLuaState)[m_scriptVarName];
  }

  bool success = isValidCreation && m_luaObjectData.valid();
  if (!success)
  {
    m_initialized = false;
    m_luaClassName = "";
    m_scriptVarName = "";
    m_luaObjectData = sol::userdata();
  }
  return success;
}
bool ScriptObject::HasFunction(const std::string &fnName)
{
  return (m_luaObjectData[fnName].valid());
}

void ScriptObject::CallFunction(const std::string &fnName)
{
  if (m_pLuaState)
    m_pLuaState->script(m_scriptVarName + ":" + fnName + "()");
}
```
Main.cpp

```cpp
#include <sol.hpp>
#include "ScriptObject.hpp"
#include "CppObject.hpp"

void test1()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);

  state.new_usertype<CppObject>("CppObject",
    "TestFunction1", &CppObject::TestFunction1
    );

  state.do_string("cppObj = CppObject:new() cppObj:TestFunction1()");
}

void test2()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];

  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();

  state.do_file("scripts/TestClass.lua");

  state.do_string("testObject = TestClass:new() testObject.TestFunctionCall()");
}

void test3()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];

  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();

  state.do_file("scripts/TestClass.lua");

  ScriptObject obj(&state, "TestClass");

  obj.CallFunction("TestFunctionCall");
}
int main()
{
  //Change value of testNum to change to the different test-cases
  int testNum = 1;
  switch (testNum)
  {
  case 1:
    test3();
    break;
  case 2:
    test2();
    break;
  case 3:
    test3();
    break;
  };
}
```