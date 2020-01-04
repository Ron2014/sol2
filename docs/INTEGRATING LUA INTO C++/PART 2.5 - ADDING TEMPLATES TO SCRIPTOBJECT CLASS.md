# INTEGRATING LUA INTO C++ (PART 2.5: ADDING TEMPLATES TO SCRIPTOBJECT CLASS)

This is Part 2.5 of my Integrating Lua into C++ Tutorial series.

Part 1a: Introduction and Setting Up
Part 1b: The Basics
Part 2: Interacting with Classes
Part 2.5: Adding Templates to ScriptObject Class (Current)
This is an extension of Part 2. Here, we will extend the ScriptObject class to provide better functionality


This part of the tutorial uses a lot of templates. Please touch up on that before continuing on with the tutorial.

The important things you should take note before coming back would be:

Templated Functions
Specialized Templated Functions
Variadic templates
What the hell, I’ll touch a bit upon the topic, at least some parts of it that he needs, but you really should properly learn about templates.

Short lesson on Templates
The topic of templates is a huge beast. I won’t be covering the whole topic, and I won’t be going in depth as I will only be covering enough materials to be used in this tutorial.

I will mostly be covering templates for functions, as well as variadic templates.

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
The code snippet above is a simple templated function that takes in a value of any type, convert it to string, and then print it out.

If you run the code successfully, you should get:

Value: 3

It behaves just like:

std::string convertArgsToLuaArgs(int value)
{
  return std::to_string(value);
}
The compiler is able to deduce that when we inserted ‘num’ as the argument, the desired parameter type is integer. It then instantiates a new function with the desired value type.

std::string convertArgsToLuaArgs(std::string value)
{
  return "\'" + value + "\'";
}
std::string convertArgsToLuaArgs(const char* str)
{
  return convertArgsToLuaArgs(std::string(str));
}
The function above is just a normal function. In the event that a string is passed in as an argument, it will call this function instead of the default templated function.

template <typename T, typename...Args>
std::string convertArgsToLuaArgs(const T& t, const Args&...args)
{
  return convertArgsToLuaArgs(t) + "," + convertArgsToLuaArgs(args...);
}
The function above is a variadic templated function. In layman terms, this function can take multiple amount of arguments and convert it into a format usable as input for our Lua functions.

std::string convertArgsToLuaArgs()
{
  return "";
}
The thing about our usage is that it works in a recursive manner, in which a variable is ‘processed’ in every recursion, and removed from the parameter pack. It will continue on until there’s no more arguments and will call the empty function.

Due to that, we have to define an empty function with the same name.

Let’s consolidate all those and try it out.

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
If you run the code successfully, you should get:

Value: 3,’test’,5.400000

New ScriptObject Class
Let’s start implementing it into our ScriptObject class.

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
The main difference between the new and the old ScriptObject class lies with the constructor and the CallFunction function. Both of these functions now takes in a variadic number of arguments, which will then be sent into their respective Lua functions.

I also added a get, tryGet, and set function for the Lua class’ member variables.

Now let’s create a new Lua class in our trusty ‘TestClass.lua’ named ‘TestClass2’ that takes in an argument in it’s constructor, as well as a function that takes in an input and returns the value after manipulating it.

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
Now Let’s try this out!

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
If all goes well, you should get:

TestClass2 Created with Input (5,’test’)!
TestClass2 TestFunction Called with Input (3, 7)!
Return Value: 10
numValue: 5
strValue: test

Summary
This new ScriptObject class would allow for an easy use to all the functionalities required of an OOP Lua class in C++. As always, the code above was edited and filled only with the bare minimum functionality for the sake of simplicity. The more complete codebase will be posted below, as usual.

But seriously though, if this article is the first exposure you have to templates, please go search up on a better tutorial because that isn’t the main scope of this tutorial. The information I provided is just the bare minimum needed for the tutorial. Template Programming is a huge beast, that is beneficial to learn for any C++ programmer.