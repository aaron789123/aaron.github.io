---
layout: post
category: "linux"
title:  "Linux系统中搭建LNMP和wordpress"
tags: [linux,LNMP, Nginx, MySQL, PHP, wordpress]
---

### 简介
>搭建LNMP(linux,Nginx, MySQL, PHP, wordpress)
>环境CentOS 7.8 64位  
>转自 https://www.linuxprobe.com/chapter-20.html    
>https://www.linuxprobe.com/centos7-install-wordpress-detail-steps.html

### 事前准备
~~~
yum install -y apr* autoconf automake bison bzip2 bzip2* compat* cpp curl curl-devel fontconfig fontconfig-devel freetype freetype* freetype-devel gcc gcc-c++ gd gettext gettext-devel glibc kernel kernel-headers keyutils keyutils-libs-devel krb5-devel libcom_err-devel libpng libpng-devel libjpeg* libsepol-devel libselinux-devel libstdc++-devel libtool* libgomp libxml2 libxml2-devel libXpm* libtiff libtiff* make mpfr ncurses* ntp openssl openssl-devel patch pcre-devel perl php-common php-gd policycoreutils telnet t1lib t1lib* nasm nasm* wget zlib-devel
~~~

### 下载安装LNMP动态网站部署架构所需的16个软件源码包和1个用于检查效果的论坛网站系统软件包
~~~
 cd /usr/local/src
 wget https://www.linuxprobe.com/Software/cmake-2.8.11.2.tar.gz
 wget https://www.linuxprobe.com/Software/Discuz_X3.2_SC_GBK.zip
 wget https://www.linuxprobe.com/Software/freetype-2.5.3.tar.gz
 wget https://www.linuxprobe.com/Software/jpegsrc.v9a.tar.gz
 wget https://www.linuxprobe.com/Software/libgd-2.1.0.tar.gz
 wget https://www.linuxprobe.com/Software/libmcrypt-2.5.8.tar.gz
 wget https://www.linuxprobe.com/Software/libpng-1.6.12.tar.gz
 wget https://www.linuxprobe.com/Software/libvpx-v1.3.0.tar.bz2
 wget https://www.linuxprobe.com/Software/mysql-5.6.19.tar.gz
 wget https://www.linuxprobe.com/Software/nginx-1.6.0.tar.gz
 wget https://www.linuxprobe.com/Software/openssl-1.0.1h.tar.gz
 wget https://www.linuxprobe.com/Software/php-5.5.14.tar.gz
 wget https://www.linuxprobe.com/Software/pcre-8.35.tar.gz
 wget https://www.linuxprobe.com/Software/t1lib-5.1.2.tar.gz
 wget https://www.linuxprobe.com/Software/tiff-4.0.3.tar.gz
 wget https://www.linuxprobe.com/Software/yasm-1.2.0.tar.gz
 wget https://www.linuxprobe.com/Software/zlib-1.2.8.tar.gz
~~~

### CMake是Linux系统中一款常用的编译工具  
~~~
tar xzvf cmake-2.8.11.2.tar.gz
cd cmake-2.8.11.2/
./configure
make 
make install
~~~

###  配置Mysql服务
~~~
cd ..
useradd mysql -s /sbin/nologin
mkdir -p /usr/local/mysql/var
chown -Rf mysql:mysql /usr/local/mysql
tar xzvf mysql-5.6.19.tar.gz
cd mysql-5.6.19/
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/var -DSYSCONFDIR=/etc
make
make install
rm -rf /etc/my.cnf
cd /usr/local/mysql
./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var
ln -s my.cnf /etc/my.cnf 
cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
chmod 755 /etc/rc.d/init.d/mysqld
~~~

编辑刚复制的MySQL数据库脚本文件，把第46、47行的basedir与datadir参数分别修改为MySQL数据库程序的保存目录和真实数据库的文件内容。  
~~~
vim /etc/rc.d/init.d/mysqld 
 46 basedir=/usr/local/mysql
 47 datadir=/usr/local/mysql/var
~~~
