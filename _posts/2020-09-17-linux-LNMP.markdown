---
layout: post
category: "linux"
title:  "Linux系统中搭建LNMP"
tags: [linux,LNMP, Nginx, MySQL, PHP]
---

### 简介
>搭建LNMP(linux,Nginx, MySQL, PHP)
>环境CentOS 7.8 64位  
>转自 https://zhuanlan.zhihu.com/p/34422328  
>http://dditblog.com/itshare_577.html  
>https://blog.csdn.net/wohiusdashi/article/details/89358071
>https://www.cnblogs.com/wenanry/archive/2012/04/16/2451798.html
>https://blog.csdn.net/sinat_34191046/article/details/83056535
>https://www.jb51.net/article/47916.htm

### 安装PHP
安装与PHP相关的依赖包  
~~~
cd /usr/local/
yum install vim gcc gcc++ wget libxml2-devel wget -y
~~~
 
官网下载适合自己版本的PHP( https://www.php.net/downloads )，解压，进入目录  
~~~
wget https://www.php.net/distributions/php-7.4.10.tar.gz
tar xf php-7.4.10.tar.gz
cd php-7.4.10
~~~

编译makefile文件(使用php-fpm，比php高效，需要加上--enable-fpm 选项)  
--prefix 就是指定安装PHP到哪个目录  
编译，安装  
~~~
./configure --prefix=/usr/local/php --enable-fpm
make && make install
~~~

如果编译出现报错：virtual memory exhausted: Cannot allocate memory  
make: *** [ext/fileinfo/libmagic/apprentice.lo] Error 1  
则在./configure后面添加--disable-fileinfo
~~~
./configure --prefix=/usr/local/php --enable-fpm --disable-fileinfo
make && make install
~~~

测试PHP是否安装成功  
新建文件index.php, 内容为：
~~~
<?php
echo 'hi world'
?>
~~~
保存，执行
~~~
/usr/local/php/bin/php index.php
~~~
将会输出hi world, 则成功。

### 安装MySQL
官网下载适合自己版本的PHP( https://dev.mysql.com/downloads/repo/yum/ )
~~~
cd /usr/local/
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
~~~

进行repo安装
执行完成后会在/etc/yum.repos.d/目录下生成两个repo文件mysql-community.repo mysql-community-source.repo
移动到/etc/yum.repos.d/目录，安装  
~~~
rpm -ivh mysql80-community-release-el7-3.noarch.rpm
cd /etc/yum.repos.d/
yum install -y mysql-server
~~~

测试是否安装成功
开启MySQL,查看进程,如果成功将会看到mysql的进程
~~~
systemctl start mysqld
ps aux | grep mysql
~~~

### Nginx安装
安装pcre扩展,这是正则匹配用到的扩展，官方网址 https://ftp.pcre.org/pub/pcre/
移动到用户级的程序目录，下载，解压，移动，检查，编译，安装
~~~
cd /usr/local/
wget https://ftp.pcre.org/pub/pcre/pcre-8.00.tar.gz
tar zxvf pcre-8.00.tar.gz
cd pcre-8.00
./configure
make & make install
~~~

如果该处make报错：  
source='pcrecpp.cc' object='pcrecpp.lo' libtool=yes 
        DEPDIR=.deps depmode=none /bin/sh ./depcomp 
        /bin/sh ./libtool --tag=CXX   --mode=compile  -DHAVE_CONFIG_H -I.   -I/usr/kerberos/include   -c -o pcrecpp.lo pcrecpp.cc
libtool: ignoring unknown tag CXX  
libtool: unrecognized option `-DHAVE_CONFIG_H'  
Try `libtool --help' for more information.  
make[1]: *** [pcrecpp.lo] Error 1  
则缺少安装gcc-c++  
~~~
yum -y install gcc-c++
./configure
make & make install
~~~

安装Nginx,官网网站：http://nginx.org/  
下载，解压，检查，编译  
~~~
wget http://nginx.org/download/nginx-1.18.0.tar.gz
tar zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
./configure --prefix=/usr/local/nginx --with-pcre=/usr/local/pcre-8.00/m
make & make install
~~~

启动Nginx,检查进程是否有启动  
~~~
/usr/local/nginx/sbin/nginx 
ps aux | grep nginx
~~~

下载浏览器links，打开网页,打印输出Welcome to nginx! 说明安装成功  
~~~
yum install -y links
links http://localhost
~~~

### 设置Nginx对于PHP的支持  
打开nginx.conf文件，添加下面句子（root为php文件目录），保存，退出  
~~~
vim /usr/local/nginx/conf/nginx.conf
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            include        fastcgi_params;
        } 
~~~

测试Nginx是否支持PHP

~~~
vim /usr/local/nginx/html/index.php
links 127.0.0.1/index.php
~~~

如果显示file not find,将nginx.conf改为下面这样
~~~
vim /usr/local/nginx/conf/nginx.conf
        location / {
            root   /usr/local/nginx/html;
            index  index.html index.htm;
        }
        location ~ \.php$ {
            root           /usr/local/nginx/html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

~~~


重启nginx和php-fpm  
安装psmisc里面包含killall  

~~~
/usr/local/nginx/sbin/nginx -s reload
killall php-fpm && /usr/local/php/sbin/php-fpm
yum install psmisc -y
~~~

查看PHP监听端口
~~~
netstat -tunlp | grep php
~~~

重启php时报错如下:  
[17-Sep-2020 03:37:14] ERROR: failed to open configuration file '/usr/local/php/etc/php-fpm.conf': No such file or directory (2)  
[17-Sep-2020 03:37:14] ERROR: failed to load configuration file '/usr/local/php/etc/php-fpm.conf'  
[17-Sep-2020 03:37:14] ERROR: FPM initialization failed  
找不到php-fpm.conf文件，复制一份php-fpm.conf.default为php-fpm.conf  
~~~
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
cp /usr/local/php/etc/php-fpm.d/www.conf.default  /usr/local/php/etc/php-fpm.d/www.conf
~~~

