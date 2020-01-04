# INTEGRATING LUA INTO C++ (PART 1B: THE BASICS)

集成 Lua 到 C++（1B：基础）

This is Part 1b of my Integrating Lua into C++ Tutorial series.

- Part 1a: Introduction and Setting Up
- Part 1b: The Basics (Current)
- Part 2: Interacting with Classes
- Part 2.5: Adding Templates to ScriptObject Class

Now that we have everything set up, let’s start!

本篇是我的《集成Lua到C++系列教程》的1a部分。

- [Part 1a: 介绍和安装](https://mansurbm.com/2018/06/16/integrating-lua-with-cpp-part-1a/)
- [Part 1b: 基础(本篇)](https://mansurbm.com/2018/06/17/integrating-lua-with-cpp-part-1b/)
- [Part 2: 类的交互](https://mansurbm.com/2018/06/23/integrating-lua-with-c-part-2/)
- [Part 2.5: 为脚本对象添加模板](https://mansurbm.com/2018/06/24/integrating-lua-with-c-part-2_5/)

现在我们已经安装好了所有东西，一起愉快地玩耍起来吧!

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

**This should also print out:**

```bash
Hello World!
```

`state.do_file("scripts/HelloWorld.lua");`

You might notice the slight change of code here. Instead of do_script, we used do_file, to indicate that we want to run the code from a ‘.lua’ file instead of a string.

## 2. Accessing Lua variables from c++

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

```cpp
state.do_string("num = testCppFunction(1);
print(": Called testCppFunction! Return value:" .. num)");
```

This line calls the testCppFunction by calling “testCppFunction(1)” in the string.

You can also do this with functional objects, lambda, etc. Try it out!

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

## 5. Summary

This part of the tutorial is a lot longer than I wanted it to be. It’s also not so interesting but well, it’s a necessary evil as you need the knowledge here before you can continue on to the next part.

## 6. Codes

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