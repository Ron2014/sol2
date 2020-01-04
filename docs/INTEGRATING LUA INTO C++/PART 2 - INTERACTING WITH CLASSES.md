# INTEGRATING LUA INTO C++ (PART 2: INTERACTING WITH CLASSES)

This is Part 2 of my Integrating Lua into C++ Tutorial series.

Part 1a: Introduction and Setting Up
Part 1b: The Basics
Part 2: Interacting with Classes (Current)
Part 2.5: Adding Templates to ScriptObject Class
For this part of the tutorial series, we will be discussing on how to interact with classes and objects, both in Lua and C++.

# Using a C++ class in Lua

First off, let’s bind a simple C++ class so that we can use the class in our lua code. You can name your class anything, but I’m going to use something simple, ‘CppObject’.

Create a class with the following interface and implementation:

CppObject.hpp

#pragma once
#include <iostream>
class CppObject
{
  void TestFunction1()
  {
    std::cout << "TestFunction1 Called!" << std::endl;
  }
};
This is a simple c++ class with a function that prints out “Test1Function1 Called!” when called.

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
This code helps to bind the C++ object into a Lua object with the alias ‘CppObject.

Lets dissect the code line by line.

state.new_usertype<CppObject>("CppObject", "TestFunction1", &CppObject::TestFunction1);

This line defines the new Lua Usertype and also defines the member function ‘TestFunction1’ to specify that the class contains this function.

state.do_string("cppObj = CppObject:new() cppObj:TestFunction1()");

In this line, we create a new object ‘cppObj’ of type ‘CppObject’ and invoked it’s member function.

If all goes well, you should printed:

TestFunction1 Called!

Congratulations! You now know how to play around with C++ Objects in your lua code. If you are really interested in learning more about this, The tutorials over in the Sol2 github page really covers this extensively.

Using a Lua class in Lua
Before we go over using a Lua class in C++, let’s first do it in Lua.

The Lua language does not have any implementation of classes but they do have something called metatable that allows you to do really cool stuffs. One of them is to implement a class system. For this, we will be using a library called middleclass. Middleclass is an OOP library that I really like to use. It’s very lightweight, and easy to use.

I will only be covering the basics, so head over to their github page for more info.

For starters, download the middleclass file and then add it into your scripts folder. I’ll show you how to set it up later.

For now, go to your scripts folder and create a new lua file with the following code:

TestClass.lua

local class = require 'middleclass'
TestClass = class('TestClass')
function TestClass:initialize()
  print('TestClass Created! ')
end
function TestClass:TestFunctionCall()
  print('TestClass TestFunction Called!')
end
The code above defines the class that we will be using, ‘TestClass’.

The initialize function will be called whenever a TestClass object is created. You can say that it’s the constructor of the class.

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

Here, I initialized the middleclass library by adding it into the luastate”s package path.

state.do_file("scripts/TestClass.lua");

I then ran the ‘TestClass.lua’ file that defines my TestClass Lua class.

state.do_string("testObj = TestClass:new() testObj.TestFunctionCall()");

I then created a TestClass object and called it’s member function ‘TestFunctionCall’

If everything went well, you should be printing this on the console:

TestClass Created!
TestClass TestFunction Called!

This is just a basic functionality for using MiddleClass together with Sol2. I hope you find it useful.

Using a Lua class in C++
Now we get to the more interesting part of the tutorial, using a lua object in c++. I find that the best way for this is to create a wrapper class.

You actually do not need to create a wrapper class like ScriptObject as you can just save it into a sol::table but I would recommend making one to make your life easier.

ScriptObject.hpp

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
Whew this is rather long. This class omits out quite a few functions for it to be usable, but it contains the bare-bones of what we need for the tutorial. I wanted to keep it simple. Don’t worry though, I’ll share the full class implementation later.

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
This is the non-default constructor of the ScriptObject.

This function creates a Lua object of type ‘luaClassName’ and gives them a unique name for use in the luaState. The name is then saved in m_scriptVarName.

m_luaObjectData = (*m_pLuaState)[m_scriptVarName];

If you remember the previous tutorial, then this should be rather familiar. Here, we save the lua object into m_luaObjectData for easy access.

m_pLuaState->script(luaScript, [&isValidCreation](lua_State* state, sol::protected_function_result res) { isValidCreation = false; return res; });

The most complex part about this function would most probably be in this line. I created this lambda function to suppress an exception being thrown if the lua code called is not valid. In this case, it would usually be called if the Lua-class of the object you are trying to create is not valid.

If an exception is thrown, we will set isValidCreation to false, and then stop with the creation of the object.

If m_initialized boolean is set to true at the end of the function, it would mean that your object is successfully created.

void CallFunction(const std::string &fnName)
{
  if (m_initialized)
    m_luaObjectData[fnName]();
}
This helper function helps us call the lua object’s member functions.

~ScriptObject()
{
  if (m_initialized && m_pLuaState)
    m_pLuaState->do_string(m_scriptVarName + " = nil");
}
In the destructor, we set the object to nil and then let the garbage collection do all the work.

Alright, so we have already defined our ScriptObject wrapper class. So now let’s try using it.

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
So here we create our ScriptObject object and then call the “TestFunctionCall” function. The behaviour is totally the same as the previous example.

If everything went well, same as before, you should be printing this on the console:

TestClass Created!
TestClass TestFunction Called!

Summary
This part of the tutorial is where you start having the basic knowledge on OOP on Lua, as well as handling classes in both Lua and C++ codes.