---
layout: post
title: "如何在Windows平台使用VS搭建C++/Lua的开发环境"
date: 2014-08-31 00:28
comments: true
categories: Lua
tags: [VS2012, C++, Lua]
---

本文主要介绍如何在Windows平台利用VS搭建C++/Lua开发环境。这里的“C++/Lua开发环境”主要指的是C++调用Lua，以及Lua调用C++。Mac平台相对会比较方便，但是VS也不是很麻烦就是了。Mac上利用XCode搭建的教程可以参考[子龙山人的教程](http://4gamers.cn/archives/242)，当然也可以利用其他IDE，比如Eclipse+CDT+LDT来搭建，这都没有问题。

另外，本文不谈及Lua/C++的交互，相关内容可以参考[《Lua程序设计》](http://book.luaer.cn/)，或者[子龙山人的Lua系列教程](http://4gamers.cn/)。

<!-- more -->

####环境

 * Windows 8.1
 * VS2012
 * Lua5.2.3

####如何搭建

1.生成Lua静态库

(1)下载lua src。

最新版本是5.2.3。 [下载地址](http://www.lua.org/download.html)。

(2)新建VS Win32控制台应用程序，取名为Lua。在应用程序设置中选择应用程序类型为静态库，附加选项中取消预编译头的勾选。

<div align="center"><img src="http://www-fusijie-com.qiniudn.com/vslua_pic_1.png" alt="" border="0" title="" /><br></br></div>


(3)在Lua的VS项目文件夹中新建LuaSrc目录，用于存放Lua源码。解压下载的lua src，拷贝src目录下所有 * .c和 * .h文件到LuaSrc。

(4)在Lua的VS项目文件夹中新建bin目录，用于存放Lua.lib。

(5)在VS环境中，右键点击Lua项目，选择添加->现有项，导入LuaSrc目录下所有的文件。

(6)右键点击Lua项目，选择属性，在顶部选择所有配置，然后修改配置属性->常规->输出目录为

	$(SolutionDir)bin

(7)为了禁止一些安全警告（Windows程序员知道为什么），需要再修改配置属性->C/C++->预处理器->预处理器定义，在末尾添加

	;_CRT_SECURE_NO_DEPRECATE;_SCL_SECURE_NO_DEPRECATE

(8)选择release模式，点击项目，生成Lua.lib即可。Lua.lib生成在bin目录下。

2.调用Lua静态库

(1)在Lua解决方案下新建名字为HelloLua的Win32控制台程序，采用默认选项，不做修改。

(2)右键点击HelloLua项目，选择属性，修改配置属性->C/C++->附加包含目录，新增

	..\LuaSrc

(3)在修改配置属性->链接器->输入->附加依赖项，新增

	..\bin\Lua.lib

或者使用代码链接lua库，即在HeloLua.cpp中添加如下代码，

``` cpp 调用lua.lib

	#pragma comment (lib,"../bin/Lua.lib")

```

(4)设置HelloLua项目为默认启动项，点击生成项目即可。

(5)因为此时main函数并没有执行任何代码，所以控制台一闪而过。右键HelloLua项目的源文件，添加新建项，取名hellolua.lua。

(6)一个简单的调用示例：

``` cpp demo

*hellolua.lua*

	print("Hello, Lua")

*HelloLua.cpp*

	#include "stdafx.h"
	#pragma comment (lib,"Lua.lib")

	#include "lua.hpp"

	int _tmain(int argc, _TCHAR* argv[])
	{
		lua_State* lua_state = luaL_newstate(); 
		luaL_openlibs(lua_state);
		luaL_dofile(lua_state,"hellolua.lua");
		lua_close(lua_state);
		getchar();
		return 0;
	}

```

*效果*

<div align="center"><img src="http://www-fusijie-com.qiniudn.com/vslua_pic_2.png" alt="" border="0" title="" /><br></br></div>

3.生成Lua.exe和Luac.exe

由于下载的lua源码中包含lua.c和luac.c，这两个文件都有main入口，同时编译的话会出错，所以只要删除其中一个，另一个就可以正常编译。

(1)新建VS空项目，取名为Lua。

(2)在Lua的VS项目文件夹中新建LuaSrc目录，用于存放Lua源码。解压下载的lua src，拷贝src目录下是所有 * .c和 * .h文件。

(3)在VS环境中，右键点击Lua项目，选择添加->现有项，导入LuaSrc目录下所有的文件。

(4)右键点击Lua项目，选择属性，在顶部选择所有配置，然后修改配置属性->C/C++->预处理器->预处理器定义，在末尾添加

	;_CRT_SECURE_NO_DEPRECATE;_SCL_SECURE_NO_DEPRECATE

(5)在Lua项目的源文件，找到luac.c，右键移除。

(6)选择release模式，点击项目生成lua.exe即可。

(7)同理在同个解决方案下创建LuaC空项目，按以上步骤生成luac.exe。只是第五步要改为“找到lua.c，右键移除”。

(8)lua.exe和luac.exe生成在Lua项目目录下的release目录。

<div align="center"><img src="http://www-fusijie-com.qiniudn.com/vslua_pic_3.png" alt="" border="0" title="" /><br></br></div>

C++调用Lua项目：[https://github.com/fusijie/Cpp_Lua_VS2012](https://github.com/fusijie/Cpp_Lua_VS2012)

Lua和LuaC项目：[https://github.com/fusijie/Lua_LuaC_exe](https://github.com/fusijie/Lua_LuaC_exe)

如果你不想这么麻烦，也可以直接从上述2个github地址直接clone我的项目。
