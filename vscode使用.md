---
title: vscode使用
toc: true
thumbnail: 
comments: true
tags:
  - C
  - vscode
date: 2020-08-12 14:12:27
urlname:
categories:
---

vscode是一个文本编辑器，附带的各种各样的插件很丰富。提前使用MinGW安装好GNU的环境，主要是c和c++编译器，使用vscode进行c语言的编辑，最终组成一个很方便的ide。

> 最终效果：实时显示编译阶段的错误、代码片段、补全、格式化、单文件的编译与调试。

- 安装c/c++、Runner这两个插件，目的是调用MinGW的编译器。

- 创建文件夹，在.vscode里创建laungh.json、tasks.json两个文件。laungh.json用于调试；tasks.json用于编译。如果不需要调试可以只创建tasks.json。这两个文件就是为了替代在命令行输入gcc -o xxx xxx.c等这些指令。

- 因此创建launch.json文件可以在左侧第四个点击Run，选择C++ （GDB/LLDB）这个选项

  > 里面主要修改
  >
  > 
  >
  > "program": "${workspaceFolder}/${fileDirname}/${fileBasenameNoExtension}.exe",
  >
  > 运行的程序是当前工作空间下当前文件目录的exe文件
  >
  > 
  >
  > "miDebuggerPath": "c:/MinGW/bin/gdb.exe"
  >
  > 运行的调试gdb的目录

- 创建tasks.json文件可以使用 terminal->configuration task里寻找模版，如果没有手动设置

  > 里面主要设置
  >
  > 
  >
  > "type": "shell",
  >
  > 使用shell程序
  >
  > 
  >
  > "command": "c:/windows/mingw/bin/gcc",
  >
  > gcc所在的目录
  >
  > 
  >
  > "args": [
  >
  > ​                "-g",
  >
  > ​                "${file}",
  >
  > ​                "-o",
  >
  > ​                "${fileDirname}/${fileBasenameNoExtension}"
  >
  > ​            ],
  >
  > 命令行的参数

-----

>关于c的一些tips：
>
>函数指针：本质上还是一个指针，只不过指向函数的入口地址。函数指针有两个用途：调用函数和做函数的[参数](https://baike.baidu.com/item/参数/5934974)。
>
>返回值类型 ( * [指针变量](https://baike.baidu.com/item/指针变量)名) ([[形参](https://baike.baidu.com/item/形参)列表]);
>
>( * [指针变量](https://baike.baidu.com/item/指针变量)名)一定要有括号，不然就变成返回一个指针的函数类型了。
>
>int * func(int a );  //函数
>
>Int (* f)(int a ); //函数指针
>
>f = func; 函数指针指向func的入口地址。