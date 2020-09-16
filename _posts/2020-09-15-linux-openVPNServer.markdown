---
layout: post
category: "linux"
title:  "Linux系统中搭建openVPN"
tags: [linux,openVPN]
---

### 简介
>搭建openVPN流程(服务端)  
>本文以CentOS 7.8 64位  
>转自 http://www.zhangblog.com/2020/05/09/openvpn01/  
>https://help.aliyun.com/knowledge_detail/42521.html?spm=5176.7841507.2.7.Yv82xS  

### yum源安装openssl和相关的依赖包
~~~
yum -y install openssl.x86_64 pam-devel.x86_64 lz4-devel lzo-devel pam-devel openssl-devel systemd-devel sqlite-devel autoconf automake libtool libtool-ltdl
~~~

### 下载openVPN源代码，解压，编译，安装，软连接
~~~
cd /usr/local/
wget https://github.com/OpenVPN/openvpn/archive/v2.4.9.tar.gz
mv v2.4.9.tar.gz openvpn-2.4.9.tar.gz
tar xf openvpn-2.4.9.tar.gz 
cd openvpn-2.4.9/
autoreconf -i -v -f 
./configure --prefix=/usr/local/openvpn --enable-lzo --enable-lz4 --enable-crypto --enable-server --enable-plugins --enable-port-share --enable-iproute2 --enable-pf --enable-plugin-auth-pam --enable-pam-dlopen --enable-systemd
make && make install
ln -s /usr/local/openvpn/sbin/openvpn /usr/local/sbin/openvpn
~~~
通过拷贝来的源码，在配置文件中保留原来的配置，所以需要使用autoreconf来更新已经生成的配置文件

### 修改配置文件
~~~
vim /usr/local/openvpn/lib/systemd/system/openvpn-server@.service
### 找到 ExecStart 这行，改为如下
ExecStart=/usr/local/openvpn/sbin/openvpn --config server.conf
~~~

### 配置系统服务，并开机自启动
~~~
cp -a /usr/local/openvpn/lib/systemd/system/openvpn-server@.service /usr/lib/systemd/system/openvpn.service
systemctl enable openvpn.service
~~~

### easy-rsa下载和解压(生成证书)
~~~
wget https://github.com/OpenVPN/easy-rsa/archive/v3.0.7.tar.gz 
mv v3.0.7.tar.gz easy-rsa-3.0.7.tar.gz
tar xf easy-rsa-3.0.7.tar.gz
~~~

### 配置修改，可以更改国家城市，拥有者，长度，算法，有效期
~~~
cd easy-rsa-3.0.7/easyrsa3
cp -a vars.example vars
vim vars
~~~

### 生成服务端和客户端证书
初始化与创建CA根证书, 会在当前目录创建PKI目录，用于存储一些中间变量及最终生成的证书
~~~
./easyrsa init-pki
~~~

### 设置CA Key Passphrase和common name
~~~
./easyrsa build-ca
~~~

### 生成服务端证书
输入上面那个CA密码，为服务端生成证书对并在本地签名。nopass参数生成一个无密码的证书；
~~~
./easyrsa build-server-full server nopass
./easyrsa gen-dh
~~~

### 生成客户端证书，需要上面那个CA密码
~~~
./easyrsa build-client-full client nopass    # 无密码
./easyrsa build-client-full AAron    # 有密码
~~~

### 提高安全性，生成ta.key
~~~
openvpn --genkey --secret ta.key
~~~

### 整理服务端证书
~~~
mkdir -p /etc/openvpn/server/
cp -a pki/ca.crt /etc/openvpn/server/
cp -a pki/private/server.key /etc/openvpn/server/
cp -a pki/issued/server.crt /etc/openvpn/server/
cp -a pki/dh.pem /etc/openvpn/server/
cp -a ta.key /etc/openvpn/server/
~~~

### 创建服务端配置文件
参照/usr/local/openvpn-2.4.9/sample/sample-config-files/server.conf文件
~~~
vim /etc/openvpn/server/server.conf   # 配置文件内容 local改为自己IP
local 0.0.0.0
port 1194
proto tcp
dev tun
ca /etc/openvpn/server/ca.crt
cert /etc/openvpn/server/server.crt
key /etc/openvpn/server/server.key
dh /etc/openvpn/server/dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "route 172.16.10.0 255.255.255.0"
;client-to-client
;duplicate-cn
keepalive 10 120
tls-auth /etc/openvpn/server/ta.key 0
cipher AES-256-CBC
compress lz4-v2
push "compress lz4-v2"
;comp-lzo
max-clients 1000
user nobody
group nobody
persist-key
persist-tun
status openvpn-status.log
log  /var/log/openvpn.log
verb 3
;explicit-exit-notify 1
~~~

### 查看防火墙和开启端口
~~~
firewall-cmd --state
systemctl status firewalld.service
firewall-cmd --zone=public --add-port=1194/tcp --permanent
~~~

### 启动openvpn服务并查看进程与端口
~~~
systemctl start openvpn.service
ps -ef | grep 'open'
##nobody   19095   1  0 01:19 ?   00:00:00 /usr/local/openvpn/sbin/openvpn --config server.conf
netstat -lntup | grep '19095' 
##tcp    0      0 0.0.0.0:1194    0.0.0.0:*     LISTEN     19095/openvpn
~~~

