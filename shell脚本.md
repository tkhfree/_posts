---
title: shell脚本
date: 2020-05-22 11:15:37
tags: Linux
---

[TOC]

shell脚本的一些方法，想到什么命令就记什么命令了。

### set

```shell
set -u -x -e -o pipefail
开局使用
-u是使用未定义的变量时报错并终止。
-x是先打印脚本里执行的命令再执行
-e只要遇到错误就终止执行 +e关闭这个功能
-o pipefail是针对管线的，因为对于管线命令来说，只要管线最后输出exit 0/执行正确，则-e -u等就不会终止执行，使用这个可以对管线每个命令进行判断
```

还有另一种针对单个命令执行判断的方法

```shell
command || exit 1
```

### ||与&& 

|| 与 &&的区别

```shell
command1 || command2
执行完command1不成功才执行command2

command1 && command2 
执行完command1成功之后执行command2
```

```shel
` `之内的内容是需要执行的命令，取执行命令的结果
```

```shell
grep -c ^head <file>
输出file文件head开头的字符串个数
```

### 运算符

```shell
取变量名用$,像c语言的指针概念。
$0 脚本本身的名称
$1 脚本后跟的第一个参数
$# 参数个数
$？最后命令的推出状态
$* 所有参数
$$ 脚本当成进程id号
$@ 与$*作用一样
```

```shel
expr是一款表达式计算器，可以计算数值运算的值
`expr 1 + 1 `
```

```shell
关系运算符（只适用于数字）
-eq 检查是否相等，相等返回true
-ne 检查是否相等，不相等返回true
```

### 脚本输出EOF用法

EOF通常都配合<<使用，unix环境下<<之后的字符是一个标志符，标识符之间的内容被输入，所以标识符可以是任何内容，只是通常用EOF（End Of File）代替。

```shell
<< EOF
...
EOF
```

...内容都被输入进去。

输入到哪用cat >>来指定文件

< :输入重定向
\> :输出重定向
\>> :输出重定向,进行追加,不会覆盖之前内容
<< :标准输入来自命令行的一对分隔号的中间内容.

```shell
tangkaifeideMacBook-Pro:Documents tang$ cat <<EOF >>test.sh 
> echo "this is a cat&EOF demo"
> echo "it can input n type"
> EOF
```



