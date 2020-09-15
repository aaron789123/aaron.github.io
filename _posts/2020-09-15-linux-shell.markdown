---
layout: post
category: "linux"
title:  "linux shell 脚本学习笔记1"
tags: [linux,shell,学习笔记 ]
---

### 简介
>该文章为Linux Shell的学习笔记
>

### 1.基础知识
shell脚本以shebang(sharp-bang)为起始:  
```
#!/bin/bahs  
```
  
脚本有两种执行方式:  
  1.将脚本名作为命令行参数：
```
  bash myScript.sh
```
  如果使用bash，则不需要添加shebang  
  2.赋予权限，直接执行
```
  chmod 755 myScript.sh
  ./myScript.sh
  #OR
  # chmod a+x myScript.sh
  # /home/path/sample.sh
```
Bash shell历史记录文件：~./bash_history

有两个方法终端打印
1.echo
双引号将会解释特殊字符，但是单引号不会。  
```
echo 'Hi !'
echo "Hi /!"
```
2.printf; 可以使用格式化字符串指定字符串的宽度、左右对齐等。
```
#!/bin/bash
#printf.sh

printf "%-5s %10s %-4s\n" No Name Mark
printf "%-5s %10s %-4.2f\n" 1 Aaron 98.88888
printf "%-5s %10s %-4.2f\n" 2 Byrant 99.99999

```
输出结果为：
```
No          Name Mark
1          Aaron 98.89
2         Byrant 100.00
```
