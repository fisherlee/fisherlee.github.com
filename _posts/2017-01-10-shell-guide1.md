---
layout:     post
title:      "sell简单介绍"
date:       2017-01-10 10:58:11 +0800
categories: shell
--

#### Shell介绍

Linux下面的Shell种类众多，常见的有：
网上抄了一段介绍


> Unix/Linux上常见的Shell脚本解释器有bash、sh、csh、ksh等，习惯上把它们称作一种Shell。我们常说有多少种Shell，其实说的是Shell脚本解释器。
bash
> 
> bash是Linux标准默认的shell，本教程也基于bash讲解。bash由Brian Fox和Chet Ramey共同完成，是BourneAgain Shell的缩写，内部命令一共有40个。
> 
> Linux使用它作为默认的shell是因为它有诸如以下的特色：
可以使用类似DOS下面的doskey的功能，用方向键查阅和快速输入并修改命令。
自动通过查找匹配的方式给出以某字符串开头的命令。
包含了自身的帮助功能，你只要在提示符下面键入help就可以得到相关的帮助。
> 
> ###### sh
> sh 由Steve Bourne开发，是Bourne Shell的缩写，sh 是Unix 标准默认的shell。
> 
> ###### ash
> ash shell 是由Kenneth Almquist编写的，Linux中占用系统资源最少的一个小shell，它只包含24个内部命令，因而使用起来很不方便。
> 
> ###### csh
> csh 是Linux比较大的内核，它由以William Joy为代表的共计47位作者编成，共有52个内部命令。该shell其实是指向/bin/tcsh这样的一个shell，也就是说，csh其实就是tcsh。
> 
> ###### ksh
> ksh 是Korn shell的缩写，由Eric Gisin编写，共有42条内部命令。该shell最大的优点是几乎和商业发行版的ksh完全兼容，这样就可以在不用花钱购买商业版本的情况下尝试商业版本的性能了。
> 
> 注意：bash是 Bourne Again Shell 的缩写，是linux标准的默认shell ，它基于Bourne shell，吸收了C shell和Korn shell的一些特性。bash完全兼容sh，也就是说，用sh写的脚本可以不加修改的在bash中执行。


#### Shell脚本

简单的一段脚本，命名为`abc.sh`，采用bash


``` shell
    #!/bin/bash
    str="hello shell script"
    echo $str
``` 
```
在终端进入到`abc.sh`的目录，输入`chomd +x ./abc.sh`使其有执行的权限
然后执行`./abc.sh`就能看到输入结果
```
#### Shell基本知识

##### 定义变量
- “=”两边不能有空格，正确`ss="abc"`，错误`ss = "abc"`
- 引用变量“$”，如：`echo $ss`，输出"ss"的内容，也可以写成`echo ${ss}`
- 字符串拼接`echo ${ss}"defg"`
- 只读变量`readonly ss`
- 删除变量`unset ss`
- 截取字符串，`string="hello shell script"`，`echo ${string:1:4}`，表示从下标1开始，长度是4，即`ello`
- 查找字符串，`echo expr index "$string" sh`，查找‘sh’的位置
- 定义一个数组，`array=("a" "b" "c" "d")`，数组中的元素用空格分隔
- 读取数组 `echo $(array[0])`
- 获取数组长度 `$(#array[@])或者是$(#array[*])`，取得第一个元素的长度`$(#array[0])`

##### 参数传递

