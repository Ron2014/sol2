# INTEGRATING LUA INTO C++ (PART 2.5: ADDING TEMPLATES TO SCRIPTOBJECT CLASS)

集成 Lua 到 C++（2.5：为脚本对象添加模板）

This is Part 2.5 of my Integrating Lua into C++ Tutorial series.

- [Part 1a: Introduction and Setting Up (Current)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: The Basics](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: Interacting with Classes](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: Adding Templates to ScriptObject Class](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

This is an extension of Part 2. Here, we will extend the ScriptObject class to provide better functionality

This part of the tutorial uses a lot of templates. Please touch up on that before continuing on with the tutorial.

The important things you should take note before coming back would be:

- Templated Functions
- Specialized Templated Functions
- Variadic templates

What the hell, I’ll touch a bit upon the topic, at least some parts of it that he needs, but you really should properly learn about templates.

本篇是我的《集成Lua到C++系列教程》的2.5部分。

- [Part 1a: 介绍和安装(本篇)](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: 基础](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: 类的交互](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: 为脚本对象添加模板](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

本篇是第2部分的扩展。在这里，我们将会扩展 ScriptObject 类来提供更牛逼的功能。

本教学将会用到很多 C++ 模板。请在继续我们的教学之前，请自行接触相关知识。

重点是，回来之前你至少做了一下笔记：

- 模板函数
- 特定的模板函数
- 可变参数的模板

哎，我会谈到到一点点上面的概念，仅限于教程会用到的内容，但是我还是建议你私下（完整地）学习模板相关知识。

## 1. Short lesson on Templates

The topic of templates is a huge beast. I won’t be covering the whole topic, and I won’t be going in depth as I will only be covering enough materials to be used in this tutorial.

I will mostly be covering templates for functions, as well as variadic templates.

```cpp
#include <iostream>
template <typename T>
std::string convertArgsToLuaArgs(T value)
{
  return std::to_string(value);
}
int main()
{
  int num = 3;
  std::cout << "Value: " + convertArgsToLuaArgs(num) << std::endl;
}
```

The code snippet above is a simple templated function that takes in a value of any type, convert it to string, and then print it out.

**If you run the code successfully, you should get:**

```bash
Value: 3
```

It behaves just like:

```cpp
std::string convertArgsToLuaArgs(int value)
{
  return std::to_string(value);
}
```

The compiler is able to deduce that when we inserted ‘num’ as the argument, the desired parameter type is integer. It then instantiates a new function with the desired value type.

```cpp
std::string convertArgsToLuaArgs(std::string value)
{
  return "\'" + value + "\'";
}
std::string convertArgsToLuaArgs(const char* str)
{
  return convertArgsToLuaArgs(std::string(str));
}
```

The function above is just a normal function. In the event that a string is passed in as an argument, it will call this function instead of the default templated function.

```cpp
template <typename T, typename...Args>
std::string convertArgsToLuaArgs(const T& t, const Args&...args)
{
  return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
}
```

The function above is a variadic templated function. In layman terms, this function can take multiple amount of arguments and convert it into a format usable as input for our Lua functions.

```cpp
std::string convertArgsToLuaArgs()
{
  return "";
}
```

The thing about our usage is that it works in a recursive manner, in which a variable is ‘processed’ in every recursion, and removed from the parameter pack. It will continue on until there’s no more arguments and will call the empty function.

Due to that, we have to define an empty function with the same name.

Let’s consolidate all those and try it out.

```cpp
#include <string>
#include <iostream>
template <typename T>
std::string convertArgsToLuaArgs(T value)
{
  return std::to_string(value);
}
template <typename T, typename...Args>
std::string convertArgsToLuaArgs(const T& t, const Args&...args)
{
  return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
}
inline std::string convertArgsToLuaArgs(int value)
{
  return std::to_string(value);
}
inline std::string convertArgsToLuaArgs(std::string value)
{
  return "\'" + value + "\'";
}
inline std::string convertArgsToLuaArgs(const char* str)
{
  return convertArgsToLuaArgs(std::string(str));
}
inline std::string convertArgsToLuaArgs()
{
  return "";
}
int main()
{
  std::cout << "Value: " + convertArgsToLuaArgs(3, "test", 5.4) << std::endl;
}
```

**If you run the code successfully, you should get:**

```bash
Value: 3,’test’,5.400000
```

## 1. 模板简易教程

模板这个概念是个大怪。我一时半会无法把它讲全，我也不会讲很深，但是我会尽最大努力分享教程中会用到的资料。

我用模板，甚至是可变参数模板，大多数用来实现函数。

```cpp
#include <iostream>
template <typename T>
std::string convertArgsToLuaArgs(T value)
{
  return std::to_string(value);
}
int main()
{
  int num = 3;
  std::cout << "Value: " + convertArgsToLuaArgs(num) << std::endl;
}
```

上面的代码片段实现了一个简单的模板函数，它接受任何类型的参数，将其转化为字符串并输出。

**没有意外的化，你能得到:**

```bash
Value: 3
```

它看起来就像是:

```cpp
std::string convertArgsToLuaArgs(int value)
{
  return std::to_string(value);
}
```

当我们放入 'num' 作为参数时，编译器能够推断出这是个整型类型。然后它会实例化一个新的函数，这个函数接受期望的特定类型。

```cpp
std::string convertArgsToLuaArgs(std::string value)
{
  return "\'" + value + "\'";
}
std::string convertArgsToLuaArgs(const char* str)
{
  return convertArgsToLuaArgs(std::string(str));
}
```

上面的代码仅表示普通函数。在这种情况下，传递字符串作为参数，不会调用默认的模板函数，而是调用参数定义为 std::string 的那个函数。

```cpp
template <typename T, typename...Args>
std::string convertArgsToLuaArgs(const T& t, const Args&...args)
{
  return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
}
```

上面的例子是个可变参数模板函数。用外行话讲，这个函数能接受多个、多样变量，然后将它们都转化成固定的格式，如同调用 Lua 函数那样。

```cpp
std::string convertArgsToLuaArgs()
{
  return "";
}
```

我们是通过递归的方式实现的，在这种方式中，会在一轮递归中处理一个参数，并将其从参数列表中移除。之后，函数会继续执行，直到参数列表为空，我们就调用上面的空函数。

因此，我们不得不定义一个同名的空函数。

让我们来巩固这些，输出点东西来。

```cpp
#include <string>
#include <iostream>
template <typename T>
std::string convertArgsToLuaArgs(T value)
{
  return std::to_string(value);
}
template <typename T, typename...Args>
std::string convertArgsToLuaArgs(const T& t, const Args&...args)
{
  return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
}
inline std::string convertArgsToLuaArgs(int value)
{
  return std::to_string(value);
}
inline std::string convertArgsToLuaArgs(std::string value)
{
  return "\'" + value + "\'";
}
inline std::string convertArgsToLuaArgs(const char* str)
{
  return convertArgsToLuaArgs(std::string(str));
}
inline std::string convertArgsToLuaArgs()
{
  return "";
}
int main()
{
  std::cout << "Value: " + convertArgsToLuaArgs(3, "test", 5.4) << std::endl;
}
```

**顺利的话，你会得到:**

```bash
Value: 3,’test’,5.400000
```

## 2. New ScriptObject Class

Let’s start implementing it into our ScriptObject class.

```cpp
class ScriptObject
{
public:
  template <typename...Args>
  explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName, const Args&...args)
    :
    m_pLuaState(nullptr),
    m_initialized(false)
  {
    if (!pLuaState) return;
    m_pLuaState = pLuaState;
    static std::size_t scriptID = 0;
    m_scriptVarName = luaClassName + "_" + std::to_string(scriptID++).c_str();
    bool isValidCreation = true;
    std::string script = m_scriptVarName + " = " + luaClassName + ":new(" + convertArgsToLuaArgs(args...) + ")";
    m_pLuaState->script(script,
      [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });
    if (isValidCreation)
    {
      m_luaObjectData = (*m_pLuaState)[m_scriptVarName];
    }
    m_initialized = isValidCreation && m_luaObjectData.valid();
  }
  ~ScriptObject()
  {
    if (m_initialized && m_pLuaState)
    {
      m_pLuaState->script(m_scriptVarName + " = nil");
    }
  }
  template <typename RetType = void, typename...Args>
  RetType CallFunction(const std::string &fnName, const Args&...args)
  {
    if (m_initialized && m_pLuaState)
    {
      std::string scriptStr = "return " + m_scriptVarName + ":" + fnName + "(" + convertArgsToLuaArgs(args...) + ")";
      return static_cast<RetType>(m_pLuaState->script(scriptStr));
    }
  }
  template <typename T>
  T Get(const std::string& name) const
  {
    return m_luaObjectData[name];
  }
  template <typename T>
  bool TryGet(const std::string& name, T* pValue) const
  {
    bool valid = m_luaObjectData[name].valid();
    if (valid && pValue)
    {
      *pValue = m_luaObjectData[name];
    }
    return valid && pValue;
  }
  template <typename T>
  void Set(const std::string& name, const T& value)
  {
    m_luaObjectData[name] = value;
  }
private:
  template <typename T, typename...Args>
  static std::string convertArgsToLuaArgs(const T& t, const Args&...args)
  {
    return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
  }
  template <typename T>
  static std::string convertArgsToLuaArgs(const T& t)
  {
    return std::to_string(t);
  }
  static std::string convertArgsToLuaArgs(const std::string &str)
  {
    return "\'" + str + "\'";
  }
  static std::string convertArgsToLuaArgs(const char* str)
  {
    return convertArgsToLuaArgs(std::string(str));
  }
  static std::string convertArgsToLuaArgs()
  {
    return "";
  }
private:
  std::string m_scriptVarName;
  sol::table m_luaObjectData;
  sol::state *m_pLuaState;
  bool m_initialized;
};
```
The main difference between the new and the old ScriptObject class lies with the constructor and the CallFunction function. Both of these functions now takes in a variadic number of arguments, which will then be sent into their respective Lua functions.

I also added a get, tryGet, and set function for the Lua class’ member variables.

Now let’s create a new Lua class in our trusty ‘TestClass.lua’ named ‘TestClass2’ that takes in an argument in it’s constructor, as well as a function that takes in an input and returns the value after manipulating it.

```lua
local class = require 'middleclass'
TestClass2 = class('TestClass2')
function TestClass2:initialize(num, str)
  print('TestClass2 Created with Input (' .. num .. ',' .. str .. ')!')
  self.numValue = num
  self.strValue = str
end
function TestClass2:TestFunction(number1, number2)
  print('TestClass2 TestFunction Called with Input (' .. number1 .. ', ' .. number2 .. ')!')
  return number1 + number2
end
```

Now Let’s try this out!

```cpp
#include "ScriptObject.hpp"
#include <iostream>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];
  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
  state.do_file("scripts/TestClass.lua");
  ScriptObject obj(&state, "TestClass2", 5, "test");
  int retVal = obj.CallFunction<int>("TestFunction", 3, 7);
  std::cout << "Return Value: " << retVal << std::endl;
  std::cout << "numValue: " << obj.Get<int>("numValue") << std::endl;
  std::cout << "strValue: " << obj.Get<std::string>("strValue") << std::endl;
}
```

**If all goes well, you should get:**

```bash
TestClass2 Created with Input (5,’test’)!
TestClass2 TestFunction Called with Input (3, 7)!
Return Value: 10
numValue: 5
strValue: test
```

## 2. 新的 ScriptObject 类

让我们开动起来，把这些实现用到 ScriptObject 类中吧。ass.

```cpp
class ScriptObject
{
public:
  template <typename...Args>
  explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName, const Args&...args)
    :
    m_pLuaState(nullptr),
    m_initialized(false)
  {
    if (!pLuaState) return;
    m_pLuaState = pLuaState;
    static std::size_t scriptID = 0;
    m_scriptVarName = luaClassName + "_" + std::to_string(scriptID++).c_str();
    bool isValidCreation = true;
    std::string script = m_scriptVarName + " = " + luaClassName + ":new(" + convertArgsToLuaArgs(args...) + ")";
    m_pLuaState->script(script,
      [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });
    if (isValidCreation)
    {
      m_luaObjectData = (*m_pLuaState)[m_scriptVarName];
    }
    m_initialized = isValidCreation && m_luaObjectData.valid();
  }
  ~ScriptObject()
  {
    if (m_initialized && m_pLuaState)
    {
      m_pLuaState->script(m_scriptVarName + " = nil");
    }
  }
  template <typename RetType = void, typename...Args>
  RetType CallFunction(const std::string &fnName, const Args&...args)
  {
    if (m_initialized && m_pLuaState)
    {
      std::string scriptStr = "return " + m_scriptVarName + ":" + fnName + "(" + convertArgsToLuaArgs(args...) + ")";
      return static_cast<RetType>(m_pLuaState->script(scriptStr));
    }
  }
  template <typename T>
  T Get(const std::string& name) const
  {
    return m_luaObjectData[name];
  }
  template <typename T>
  bool TryGet(const std::string& name, T* pValue) const
  {
    bool valid = m_luaObjectData[name].valid();
    if (valid && pValue)
    {
      *pValue = m_luaObjectData[name];
    }
    return valid && pValue;
  }
  template <typename T>
  void Set(const std::string& name, const T& value)
  {
    m_luaObjectData[name] = value;
  }
private:
  template <typename T, typename...Args>
  static std::string convertArgsToLuaArgs(const T& t, const Args&...args)
  {
    return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
  }
  template <typename T>
  static std::string convertArgsToLuaArgs(const T& t)
  {
    return std::to_string(t);
  }
  static std::string convertArgsToLuaArgs(const std::string &str)
  {
    return "\'" + str + "\'";
  }
  static std::string convertArgsToLuaArgs(const char* str)
  {
    return convertArgsToLuaArgs(std::string(str));
  }
  static std::string convertArgsToLuaArgs()
  {
    return "";
  }
private:
  std::string m_scriptVarName;
  sol::table m_luaObjectData;
  sol::state *m_pLuaState;
  bool m_initialized;
};
```

新版 ScriptObject 和旧版的区别主要集中在构造函数和 CallFunction 函数。这俩函数现在都支持可变参数模板，然后它们会将参数（连接成字符串）传到 Lua 函数中。

我也增加了 get, tryGet, set 函数用来访问 Lua 对象的成员变量。

现在，让我们在 'TestClass.lua' 文件中创建一个新类 'TestClass2'，让它支持有参数的构造函数以及其他成员函数。

```lua
local class = require 'middleclass'
TestClass2 = class('TestClass2')
function TestClass2:initialize(num, str)
  print('TestClass2 Created with Input (' .. num .. ',' .. str .. ')!')
  self.numValue = num
  self.strValue = str
end
function TestClass2:TestFunction(number1, number2)
  print('TestClass2 TestFunction Called with Input (' .. number1 .. ', ' .. number2 .. ')!')
  return number1 + number2
end
```

把这个脚本用起来。

```cpp
#include "ScriptObject.hpp"
#include <iostream>
int main()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];
  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();
  state.do_file("scripts/TestClass.lua");
  ScriptObject obj(&state, "TestClass2", 5, "test");
  int retVal = obj.CallFunction<int>("TestFunction", 3, 7);
  std::cout << "Return Value: " << retVal << std::endl;
  std::cout << "numValue: " << obj.Get<int>("numValue") << std::endl;
  std::cout << "strValue: " << obj.Get<std::string>("strValue") << std::endl;
}
```

**如果顺利的话，你会得到:**

```bash
TestClass2 Created with Input (5,’test’)!
TestClass2 TestFunction Called with Input (3, 7)!
Return Value: 10
numValue: 5
strValue: test
```

## 3. Summary

This new ScriptObject class would allow for an easy use to all the functionalities required of an OOP Lua class in C++. As always, the code above was edited and filled only with the bare minimum functionality for the sake of simplicity. The more complete codebase will be posted below, as usual.

But seriously though, if this article is the first exposure you have to templates, please go search up on a better tutorial because that isn’t the main scope of this tutorial. The information I provided is just the bare minimum needed for the tutorial. Template Programming is a huge beast, that is beneficial to learn for any C++ programmer.

## 3. 总结

这个新版 ScriptObject 类提供了易总简易的用法来调用可能出现在 Lua 类机制中的所有函数（包括有参数的函数）。我们一如既往地让它尽量简单，所以实现的功能很少。像往常一样，后面会有完整得代码底层的展示。

但是严格来讲，如果本教程用到的模板是你第一次接触，请自行搜索一些更好的（模板编写）教程，因为本教程的主题不在于此。我提供的信息仅仅只是教程用到的最简化的版本。模板编程是个大怪，最好去学一下 C++ 编程。

## Codes

TestClass.lua

```lua
local class = require 'middleclass'
TestClass2 = class('TestClass2')

function TestClass2:initialize(num, str)
  print('TestClass2 Created with Input (' .. num .. ',' .. str .. ')!')
  self.numValue = num
  self.strValue = str
end
function TestClass2:TestFunction(number1, number2)
  print('TestClass2 TestFunctionRetInt Called with Input (' .. number1 .. ', ' .. number2 .. ')')
  return number1 + number2
end
```

ScriptObject.hpp

```cpp
#include <string>
#include <sol.hpp>
class ScriptObject
{
public:
  ScriptObject();
  template <typename...Args>
  explicit ScriptObject(sol::state *pLuaState, const std::string &luaClassName, const Args&...args);

  ScriptObject(const ScriptObject&other) = delete;
  ScriptObject& operator=(const ScriptObject&other) = delete;
  ~ScriptObject();

  template <typename...Args>
  bool Init(sol::state *pLuaState, const std::string &scriptName, const Args&...args);

  bool HasFunction(const std::string &fnName);
  const std::string& GetScriptVarName()const;
  const std::string& GetLuaClassName()const;

  template <typename RetType = void, typename...Args>
  RetType CallFunction(const std::string &fnName, const Args&...args);

  template <typename RetType = void, typename...Args>
  RetType UnsafeCallFunction(const std::string &fnName, const Args&...args);
private:
  template <typename T, typename...Args>
  static std::string convertArgsToLuaArgs(const T& t, const Args&...args);
  template <typename T>
  static std::string convertArgsToLuaArgs(const T& t);
  static std::string convertArgsToLuaArgs(const std::string &str);
  static std::string convertArgsToLuaArgs(const char* str);
  static std::string convertArgsToLuaArgs();
private:
  std::string m_luaClassName;
  std::string m_scriptVarName;
  sol::table m_luaObjectData;
  sol::state *m_pLuaState;
  bool m_initialized;
};
template <typename...Args>
ScriptObject::ScriptObject(sol::state *pLuaState, const std::string &luaClassName, const Args&...args)
  :
  m_pLuaState(nullptr),
  m_initialized(false)
{
  Init(pLuaState, luaClassName, args...);
}

template <typename...Args>
bool ScriptObject::Init(sol::state *pLuaState, const std::string &luaClassName, const Args&...args)
{
  if (!pLuaState) return false;

  m_pLuaState = pLuaState;
  m_initialized = true;
  m_luaClassName = luaClassName;

  static std::size_t scriptID = 0;

  m_scriptVarName = m_luaClassName + "_" + std::to_string(scriptID++).c_str();

  bool isValidCreation = true;
  std::string script = m_scriptVarName + " = " + m_luaClassName + ":new(" + convertArgsToLuaArgs(args...) + ")";
  m_pLuaState->script(script,
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
template <typename T, typename...Args>
std::string ScriptObject::convertArgsToLuaArgs(const T& t, const Args&...args)
{
  return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
}
template <typename T>
std::string ScriptObject::convertArgsToLuaArgs(const T& t)
{
  return std::to_string(t);
}

template <typename RetType, typename...Args>
RetType ScriptObject::CallFunction(const std::string &fnName, const Args&...args)
{
  if (m_initialized && m_pLuaState)
  {
    if (HasFunction(fnName))
      return UnsafeCallFunction<RetType, Args...>(fnName, args...);
  }
}
template <typename RetType, typename...Args>
RetType ScriptObject::UnsafeCallFunction(const std::string &fnName, const Args&...args)
{
  std::string scriptStr = std::string("return ") + m_scriptVarName + ":" + fnName + "(" + convertArgsToLuaArgs(args...) + ")";
  return static_cast<RetType>(m_pLuaState->script(scriptStr));
}
template <typename T>
T ScriptObject::Get(const std::string& name) const
{
  return m_luaObjectData[name];
}
template <typename T>
bool ScriptObject::TryGet(const std::string& name, T* pValue) const
{
  bool valid = m_luaObjectData[name].valid();
  if (valid && pValue)
  {
    *pValue = m_luaObjectData[name];
  }
  return valid && pValue;
}
template <typename T>
void ScriptObject::Set(const std::string& name, const T& value)
{
  m_luaObjectData[name] = value;
}
```

ScriptObject.cpp

```cpp
#include "ScriptObject.hpp"
ScriptObject::ScriptObject()
  :
  m_pLuaState(nullptr),
  m_initialized(false)
{

}
ScriptObject::~ScriptObject()
{
  if (m_initialized)
  {
    m_pLuaState->script(m_scriptVarName + " = nil");
  }
}
bool ScriptObject::HasFunction(const std::string &fnName)
{
  return m_initialized && (m_luaObjectData[fnName].valid());
}
const std::string& ScriptObject::GetScriptVarName()const
{
  return m_scriptVarName;
}
const std::string& ScriptObject::GetLuaClassName()const
{
  return m_luaClassName;
}
std::string ScriptObject::convertArgsToLuaArgs()
{
  return "";
}
std::string ScriptObject::convertArgsToLuaArgs(const std::string &str)
{
  return "\'" + str + "\'";
}
std::string ScriptObject::convertArgsToLuaArgs(const char *str)
{
  return convertArgsToLuaArgs(std::string(str));
}
```

ConvertArgsToLuaArgs.hpp

```cpp
#include <string>
template <typename T>
std::string convertArgsToLuaArgs(T value)
{
  return std::to_string(value);
}

template <typename T, typename...Args>
std::string convertArgsToLuaArgs(const T& t, const Args&...args)
{
  return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
}

inline std::string convertArgsToLuaArgs(int value)
{
  return std::to_string(value);
}

inline std::string convertArgsToLuaArgs(std::string value)
{
  return "\'" + value + "\'";
}

inline std::string convertArgsToLuaArgs(const char* str)
{
  return convertArgsToLuaArgs(std::string(str));
}

inline std::string convertArgsToLuaArgs()
{
  return "";
}
```

Main.cpp

```cpp
#include "ScriptObject.hpp"
#include "ConvertArgsToLuaArgs.hpp"
#include <iostream>

void test1()
{
  std::cout << "Value: " + convertArgsToLuaArgs(3, "test", 5.4) << std::endl;
}

void test2()
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

void test3()
{
  sol::state state;
  state.open_libraries(sol::lib::base, sol::lib::package);
  std::string package_path = state["package"]["path"];

  state["package"]["path"] = (package_path + ";scripts/middleclass.lua").c_str();

  state.do_file("scripts/TestClass.lua");

  ScriptObject obj(&state, "TestClass2", 5, "test");

  int retVal = obj.CallFunction<int>("TestFunction", 3, 7);
  std::cout << "Return Value: " << retVal << std::endl;

  std::cout << "numValue: " << obj.Get<int>("numValue") << std::endl;
  std::cout << "strValue: " << obj.Get<std::string>("strValue") << std::endl;
}

int main()
{
  //Change value of testNum to change to the different test-cases
  int testNum = 1;
  switch (testNum)
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
  };
}
```