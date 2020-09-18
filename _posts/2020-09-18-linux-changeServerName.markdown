---
layout: post
category: "linux"
title:  "Linux系统中搭建openVPN(服务器)"
tags: [linux,改名字]
---

### 简介
>更改服务器名字（阿里云）  
>本文以CentOS 7.8 64位  
>转自 https://www.sindns.net/info/a/news/20140731/693.html

vi /etc/sysconfig/network
修改主机名(HOSTNAME=)

输入以下命令：
hostname 新主机名
然后用ssh重新登录，就会显示新的主机名。
