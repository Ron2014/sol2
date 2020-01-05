# C++ in Lua

Using user defined types (“usertype”s, or just “udt”s) is simple with sol. If you don’t call any member variables or functions, then you don’t even have to ‘register’ the usertype at all: just pass it through. But if you want variables and functions on your usertype inside of Lua, you need to register it. We’re going to give a short example here that includes a bunch of information on how to work with things.

通过 sol 使用用户自定义类型(“usertype”s, 或者缩写成 “udt”s)非常简单。如果你不需要调用任何成员变量或者函数，那你根本就不用“注册” usertype：（将变量名称）传递过去（到 lua state 中）就行了。但是，如果你想在 usertype 中封装一些变量和函数，交给 Lua 脚本使用，你就得注册之。我们打算给出一个简短的示例来介绍它是如何工作的。

Take this `player` struct in C++ in a header file:

在头文件中定义 player 结构：

player.hpp

```cpp
#include <iostream>

struct player {
public:
	int bullets;
	int speed;

	player()
		: player(3, 100) {

	}

	player(int ammo)
		: player(ammo, 100) {

	}

	player(int ammo, int hitpoints)
		: bullets(ammo), hp(hitpoints) {

	}

	void boost() {
		speed += 10;
	}

	bool shoot() {
		if (bullets < 1)
			return false;
		--bullets;
		return true;
	}

	void set_hp(int value) {
		hp = value;
	}

	int get_hp() const {
		return hp;
	}

private:
	int hp;
};
```

It’s a fairly minimal class, but we don’t want to have to rewrite this with metatables in Lua. We want this to be part of Lua easily. The following is the Lua code that we’d like to have work properly:

这是个相当小巧的类，但是我们不需要在 Lua 层用 metatbale 重写它。让它成为 Lua 的一部分其实很简单。接下来就是让其正确工作的 Lua 代码（绑定后的目标）：

player_script.lua

```lua
p1 = player.new(2)

-- p2 从一开始存活到现在
-- 在 C++ 层执行：lua["p2"] = player(0);
local p2shoots = p2:shoot()
assert(not p2shoots)
-- p2 的 ammo 是 0（ammo 赋值给成员变量 bullets）
	
-- 通过合适的 setter 设置变量
p1.hp = 545
-- 通过合适的 unqualified_getter 获取变量
print(p1.hp)
assert(p1.hp == 545)

local did_shoot_1 = p1:shoot()
print(did_shoot_1)
print(p1.bullets)
local did_shoot_2 = p1:shoot()
print(did_shoot_2)
print(p1.bullets)
local did_shoot_3 = p1:shoot()
print(did_shoot_3)
	
-- 读取 bullets（有 getter）
print(p1.bullets)
-- 下面代码会发生错误：因为该变量只读（没有 setter）
-- p1.bullets = 20

p1:boost()
-- brake 函数不是 C++ 定义的函数，而是在运行时通过 Lua 脚本定义的函数。
p1:brake()
```

To do this, you bind things using the new_usertype and method as shown below. These methods are on both table and state(_view), but we’re going to just use it on state:

为了实现这个目标，你得用下面提到的 new_usertype 和 method 进行绑定。这些函数可以放到 table 里也可以放到 state(_view) 里，我们把放到 state 里：

```cpp
#define SOL_ALL_SAFETIES_ON 1
#include <sol/sol.hpp>

#include "player.hpp"

#include <iostream>

int main() {
	sol::state lua;

	lua.open_libraries(sol::lib::base);

    // 注意，你能在注册 usertype 之前创建 userdata
    // 就算注册放在后面，它含有的 metatable 也是正确的

    // 创建一个 'p2' 变量，其类型为 'player'，构造函数参数 ammo 为0
	lua["p2"] = player(0);

    // 构造 usertype metatable
	sol::usertype<player> player_type = lua.new_usertype<player>("player",
		// 定义3个构造函数（无参数，1个 int 参数，2个 int 参数）
		sol::constructors<player(), player(int), player(int, int)>());

    // 标准成员函数，1个返回值
	player_type["shoot"] = &player::shoot;
	// 标准成员函数
	player_type["boost"] = &player::boost;

	// gets or set the value using member variable syntax
    // 通过成员变量语法指定这个值（hp）的 get 或 set 函数
	player_type["hp"] = sol::property(&player::get_hp, &player::set_hp);

    // 读取或写入变量（这个 player::speed 就是个成员变量啊，没有访问器封装）
	player_type["speed"] = &player::speed;
    // 只读，不可写的成员变量
    // .set(foo, bar) 义同 [foo] = bar;
	player_type.set("bullets", sol::readonly(&player::bullets));
    // player_type["bullets] = sol::readonly(&player::bullets);

	lua.script_file("prelude_script.lua");
	lua.script_file("player_script.lua");
	return 0;
}
```

There is one more method used in the script that is not in C++ or defined on the C++ code to bind a usertype, called brake. Even if a method does not exist in C++, you can add methods to the class table in Lua:

在这里，多了一个脚本中使用的函数并没有在 C++ 代码，以及 C++ 绑定 usertype 的代码中定义，它的名字叫 'brake'。尽管函数不存在 C++ 中，你也可以通过 Lua 层的类 table 添加：

prelude_script.lua

```lua
function player:brake ()
	self.speed = 0
	print("we hit the brakes!")
end
```

That script should run fine now, and you can observe and play around with the values. Even more stuff [you can do](https://sol2.readthedocs.io/en/latest/api/usertype.html) is described elsewhere, like initializer functions (private constructors / destructors support), “static” functions callable with `name.my_function( ... )`, and overloaded member functions. You can even bind global variables (even by reference with std::ref) with sol::var. There’s a lot to try out!

这段代码运行良好，而且你可以观察并调试这些值。你[能做](https://sol2.readthedocs.io/en/latest/api/usertype.html)更多的是，在别处（继续添加类的）描述。例如初始化函数（支持私有构造函数/析构函数），'static' 函数可以通过 `name.my_function( ... )` 的方式调用，还有重载成员函数。甚至可以通过 sol::var 绑定全局变量（甚至是通过 std::ref 声明的引用）。反正有很多种玩法。

This is a powerful way to allow reuse of C++ code from Lua beyond just registering functions, and should get you on your way to having more complex classes and data structures! In the case that you need more customization than just usertypes, however, you can customize sol to behave more fit to your desires by using the desired [customization and extension structures](https://sol2.readthedocs.io/en/latest/tutorial/customization.html).

不仅仅是注册函数，在 Lua 层实现对 C++ 代码的重用也有强大的支持。你真的可以用你自己（喜欢）的方式定义复杂的类和结构！在这中情况下，比起 usertypes，你需要更自由的自定义方式，然而，你得通过使用预期的[可定制、可扩展结构](https://sol2.readthedocs.io/en/latest/tutorial/customization.html) （其实就是重写一套对结构体类型的 check/get/push API）来自定义 sol ，使其更符合你的需求。

You can check out this code and more complicated code at the [examples directory](https://github.com/ThePhD/sol2/tree/develop/examples) by looking at the `usertype_`-prefixed examples.

从[范例目录](https://github.com/ThePhD/sol2/tree/develop/examples)搜索`usertype_`开头的示例，你能获取本篇的代码以及更复杂的代码。