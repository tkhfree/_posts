---
title: Python总结1
toc: true
thumbnail: 
comments: true
tags:
  - Python
date: 2020-06-29 09:00:45
urlname:
categories:
---

## 数字类型

### 整形

int long

### 浮点型

float

### 复数

In [11]:

```
a = 1
type(a)
b=1.0
type(b)
c=1111111111111111111111111111111
type(c)
```

Out[11]:

```
int
```

## 表达式

不支持i++，使用i+=1

In [15]:

```
a=6+8
print (a)
14
```

## 字符串和列表

字符串就是特殊形式的列表，但是字符串的值是常量，对于字符串的更改都是更改的其副本，列表是变量，对于列表更改内容，列表地址不变。

字符串养成习惯用双引号来表示

python没有数组的概念，用列表表示，列表中可以存放任意类型的数据，同一列表可以放置多种数据

In [55]:

```
# 取下标操作
str_test="hello"
list_test=["h","e","l","l","o"]
print (str_test[0])
print (list_test[0])
# 取1个到第3个之间的元素
print (str_test[1:4])
print (list_test[0:4])
# 取第1个到5个之间的元素，每隔2个取1个
print (str_test[0:5:2])
print (list_test[0:5:2])
# 取某个元素的下标
print (str_test.index("e"))
print (list_test.index("o"))
# 逆序取得某个元素
print (str_test[-1])
print (list_test[-1])
# 追加元素
print (str_test+",world")
print (list_test+[",","world"])
# 计算长度
print (len(str_test))
print (len(list_test))
# 多次复制
print(str_test*3)
print(list_test*4)
# 复制新变量
str_new_test = str_test[:]
list_new_test = list_test[:]
print(str_new_test)
print(list_new_test)
# python中变量名有点像c中的指针
str_new_test1 = str_new_test
print(id(str_new_test))
print(id(str_new_test1))
print(id(str_test))

# 列表变量可以更改元素
list_test[1]="i"
print(list_test)
# 列表追加
list_test.append("world")
print(list_test)

print(list_test+["python"])
# 列表在第1位插入
list_test[1:1]=[1234567]
print(list_test)

# 字符串的大小写转变
str_test_upper = str_test.upper()
print(str_test_upper)
h
h
ell
['h', 'e', 'l', 'l']
hlo
['h', 'l', 'o']
1
4
o
o
hello,world
['h', 'e', 'l', 'l', 'o', ',', 'world']
5
5
hellohellohello
['h', 'e', 'l', 'l', 'o', 'h', 'e', 'l', 'l', 'o', 'h', 'e', 'l', 'l', 'o', 'h', 'e', 'l', 'l', 'o']
hello
['h', 'e', 'l', 'l', 'o']
140384645081328
140384645081328
140384645081328
['h', 'i', 'l', 'l', 'o']
['h', 'i', 'l', 'l', 'o', 'world']
['h', 'i', 'l', 'l', 'o', 'world', 'python']
['h', 1234567, 'i', 'l', 'l', 'o', 'world']
HELLO
```

## 列表

In [*]:

```
# 几种构建列表方式
list_test1=list("helloworld")
list_test2=["h","e","l","l","o"]
list_test3=[x for x in range(10)]
list_test4=map(lambda x:x ,[1,2,3,4,5])
print(list_test4)
print(list_test1)
```

## **lambda**

构造匿名函数

result = lambda x : x+1

等同于

def result(x): return x+1

## map

map(function,sequenec[,sequence, ...]) -->list

map接受两个参数，一个函数，一个可迭代的对象或者是序列，返回一个列表

## 元组

元组就是常量化的列表

tuple_test=(1,2,3,4,5)

元组表达跟列表基本一致，只是元组里的元素不能修改，列表的操作（除了修改）元组也都可以操作

列表转为元组

tup_test=tuple([1,2,3,4,5])

**什么时候用元组？当希望把一个序列作为参数传递时，不希望被修改**