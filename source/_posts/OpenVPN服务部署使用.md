title: OpenVPN服务部署使用
author: liudong
date: 2016-09-08 20:00:00
# 类别
categories:
  - blog
# 标签
tags:
  - 运维
---
作者：liudong at 2016-09-08 20:00:00

OpenVPN服务部署使用
## 1 服务端部署（Ubuntu）
### 1.1 安装OpenVPN 所需插件
```
$ sudo apt-get install openssl
$ sudo apt-get install libssl-dev
$ sudo apt-get install libpam0g-dev
$ sudo apt-get install liblzo2-dev
```
<!--more-->

### 1.2 安装OpenVPN
注:以下安装方式任选一种,推荐apt-get方式安装
#### 1.2.1 apt-get安装OpenVPN
```
$apt-get install openvpn
$cd /etc/openvpn
$mkdir conf log
```
#### 1.2.2 源码安装OpenVPN（建议使用2.2.2版本）
```
$ wget http://swupdate.openvpn.org/community/releases/openvpn-2.2.2.tar.gz  
$ tar -zxvf openvpn-2.2.2.tar.gz  
$ mkdir /data/openvpn && cd openvpn-2.2.2  
$ ./configure --enable-password-save --prefix=/etc/openvpn  
$ make && sudo make install  
```
注：--enable-password-save该选项是避免手工输入客户端密码；--prefix选项是真正的安装路径
### 1.3 开启内核转发并配置源地址路由
```
$ echo "1" > /proc/sys/net/ipv4/ip_forward 
$ iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```
### 1.4 服务端配置
#### 1.4.1 生成密钥
``` 
$ cd openvpn-2.2.2/easy-rsa/2.0 
$ source ./vars     # 在此之前，可以修改vars文件对国家省份等修改;配置dh的位数(默认是1024，可以改成export KEY_SIZE=2048)和下文生成的dh2048.pem相对应
$ ./clean-all 
$ ./build-ca 
$ ./build-key-server server # 产生服务器证书，此处的server是文件名参数，可以任意修改。
$ ./build-key-pass client1 # 生成客户端key，pass表示需要输入一个密码作为客户端启动时的凭证； ./build-key则不需要输入密码
$ ./build-dh # 产生Diffie Hellman参数
至此一个客户端所需的证书已经完毕，都在easy-rsa/2.0/keys文件夹下面，其中ca.crt server.crt server.csr server.key dh1024.pem是服务端所需证书文件，ca.crt ca.key client1.crt client1.csr client1.key是客户端所需证书文件。
注：可以继续使用./build-key产生更多客户端证书,一个客户端证书只能同时用于一个客户端连接。
```
#### 1.4.2 服务端配目录及文件
```
$ cd openvpn && mkdir conf # openvpn就是第2步中openvpn的安装目录 
$ cp openvpn-2.2.2/sample-config-files/server.conf conf/
$ cp openvpn-2.2.2/easy-rsa/2.0/keys/{ca.crt,server.crt,server.csr,server.key,dh1024.pem} conf/ # 拷贝openvpn-2.2.2/easy-rsa/2.0/keys/下的相关证书文件到openvpn/conf/目录下，注意:2048位的key则是dh2048.pem; 1024位的key则是dh1024.pem
```
#### 1.4.3 服务端配置文件参数指定
```
$ vim conf/server.conf
dev tap
proto tcp
port 1194
ca /path/to/openvpn/conf/ca.crt
cert /path/to/openvpn/conf/server.crt
key /path/to/openvpn/conf/server.key
dh /path/to/openvpn/conf/dh1024.pem
user nobody
group nogroup
server 10.8.0.0 255.255.255.0     # 分配给clinet的ip段
second time period
keepalive 10 120                            # 每10秒ping一次，120秒内客户端没有动作就断开连接
persist-key
persist-tun
verb 4
log-append /path/to/openvpn/log/openvpn.log
status /path/to/openvpn/log/openvpn-status.log
client-to-client
crl-verify /path/to/openvpn/conf/crl.pem            # 客户端证书连接限制
comp-lzo
```
### 1.5 启动OpenVPN服务端
```
sudo /path/to/openvpn/sbin/openvpn --config /path/to/openvpn/conf/server.conf --daemon
```
### 1.6 检查验证  
```
$ ifconfig|grep inet|grep 10.8.0.1  
inet 10.8.0.1 netmask 0xffffff00 broadcast 10.8.0.255  
```
注：得到IP为：10.8.0.1 则说明VPN服务端配置成功  

### 1.7 设置OpenVPN服务端开机启动
```
$ vim /etc/rc.local
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
/path/to/openvpn/sbin/openvpn --config /path/to/openvpn/conf/server.conf --daemon
```
## 2 OpenVPN 客户端部署(MAC系统)  
### 2.1 安装OpenVPN 所需插件
```
$ sudo brew install openssl
```

### 2.2 安装OpenVPN
注:以下安装方式任选一种,推荐Brew方式安装
#### 2.2.1 Brew 安装
```
直接brew安装(推荐)
$brew install openvpn
$cd /usr/local/Cellar/openvpn/2.3.11_1
$mkdir conf log
$ln -s /usr/local/Cellar/openvpn/2.3.11_1/sbin/openvpn /usr/local/bin/openvpn
```

#### 2.2.2 源码安装（建议使用2.2.2版本）
```
$ wget http://swupdate.openvpn.org/community/releases/openvpn-2.2.2.tar.gz  
$ tar -zxvf openvpn-2.2.2.tar.gz  
$ mkdir /data/openvpn && cd openvpn-2.2.2  
$ ./configure --enable-password-save --prefix=/etc/openvpn  
$ make && sudo make install  
```
注：--enable-password-save该选项是避免手工输入客户端密码；--prefix选项是真正的安装路径
### 2.3 服务端生成客户端所需密钥(客户端部署可忽略此步骤) 
#### 2.3.1 服务端连接
```
服务端所在机器：xxx.xxx.xxx.xxx  
$ssh root@xxx.xxx.xxx.xxx  #连接方式
$cd /media/openvpn/  服务端所在路径
$cd /media/openvpn-2.2.2/easy-rsa/2.0  生成密钥所需路径
```
#### 2.3.2 生成密钥  
```
$ source ./vars  
$ ./build-key-pass client-A    #此处设置密码为：openvpn123 
```
注：生成客户端key，pass表示需要输入一个密码作为客户端启动时的凭证； ./build-key则不需要输入密码  
```
$ ./build-dh  
```
#### 2.3.3设置客户端密钥验证信息  
```
$ vim /media/openvpn/conf/ccd/client-A  
ifconfig-push 10.8.0.119 255.255.255.0  
```
注：此处的验证信息文件名需要和生成密钥时输入的名字保持一致;  
10.8.0.119 指客户端被虚拟出来的IP  
### 2.4 客户端配置  
#### 2.4.1 拷贝密钥到客户端  
```
$scp root@xxx.xxx.xxx.xxx:/media/openvpn-2.2.2/easy-rsa/2.0/keys/{ca.crt,ca.key,client-A.crt,client-A.csr,client-A.key}   /usr/local/Cellar/openvpn/2.3.11_1/conf/
注:密钥可以由维护人员发放,联系刘东;
```
#### 2.4.2 配置客户端密码文件
```
$ vim /usr/local/Cellar/openvpn/2.3.11_1/conf/password.txt  
openvpn123  
```
注:客户端密码文件和服务端生成密钥时输入的密码一致
#### 2.4.3 客户端配置文件
```
$ vim /usr/local/Cellar/openvpn/2.3.11_1/conf/client.conf  
client  
dev tap  
proto tcp  
remote xxx.xxx.xxx.xxx 1194   #指定服务端外网IP及端口
nobind  
user nobody  
group nogroup  
ca /usr/local/Cellar/openvpn/2.3.11_1/conf/ca.crt  
cert /usr/local/Cellar/openvpn/2.3.11_1/conf/client-A.crt  
key /usr/local/Cellar/openvpn/2.3.11_1/conf/client-A.key  
ping 15  
ping-restart 45  
ping-timer-rem  
persist-key  
persist-tun  
ns-cert-type server  
comp-lzo  
verb 4  
log-append /usr/local/Cellar/openvpn/2.3.11_1/log/openvpn.log  
status /usr/local/Cellar/openvpn/2.3.11_1/log/openvpn-status.log  
tcp-queue-limit 4096 # 256  
bcast-buffers 4096  
```
### 2.5 启动客户端  
#### 2.5.1 命令行启动OpenVPN
```
$ sudo /usr/local/bin/openvpn --config /usr/local/Cellar/openvpn/2.3.11_1/conf/client.conf --askpass /usr/local/Cellar/openvpn/2.3.11_1/conf/password.txt --daemon  
```
#### 2.5.2 GUI启动OpenVPN
* 下载Tunnelblick客户端   
直接官网下载: [https://tunnelblick.net/downloads.html](https://tunnelblick.net/downloads.html)    
* 安装Tunnelblick客户端  
Tunnelblick具体安装使用流程见：[Mac系统Tunnelblick下载以及安装流程](http://blog.csdn.net/sinat_25471067/article/details/43482837)

### 2.6 检查验证  
```
$ ifconfig|grep inet|grep 10.8.0.119  
inet 10.8.0.119 netmask 0xffffff00 broadcast 10.8.0.255  
```
注：得到IP为：10.8.0.x 则说明VPN客户端配置成功  
```
$ ping 10.8.0.1    #检查是否能ping通内网等机器
```
### 2.7 服务加入开机自启动
```
vim /etc/rc.local
/usr/local/bin/openvpn --config /usr/local/Cellar/openvpn/2.3.11_1/conf/client.conf --askpass /usr/local/Cellar/openvpn/2.3.11_1/conf/password.txt --daemon
```
## 3 OpenVPN 客户端部署(Ubuntu系统)  
### 3.1 安装OpenVPN 所需插件
```
$ sudo apt-get install openssl
$ sudo apt-get install libssl-dev
$ sudo apt-get install libpam0g-dev
$ sudo apt-get install liblzo2-dev
```
### 3.2 安装OpenVPN
注:以下安装方式任选一种,推荐apt-get方式安装
#### 3.2.1 apt-get安装OpenVPN
```
$apt-get install openvpn
$cd /etc/openvpn
$mkdir conf log
```
#### 3.2.2 源码安装OpenVPN（建议使用2.2.2版本）
```
$ wget http://swupdate.openvpn.org/community/releases/openvpn-2.2.2.tar.gz  
$ tar -zxvf openvpn-2.2.2.tar.gz  
$ mkdir /data/openvpn && cd openvpn-2.2.2  
$ ./configure --enable-password-save --prefix=/etc/openvpn  
$ make && sudo make install  
```
注：--enable-password-save该选项是避免手工输入客户端密码；--prefix选项是真正的安装路径
### 3.3 服务端生成客户端所需密钥(客户端部署可忽略此步骤) 
#### 3.3.1 服务端连接
```
服务端所在机器：xxx.xxx.xxx.xxx  
$ssh root@xxx.xxx.xxx.xxx  #连接方式
$cd /media/openvpn/  服务端所在路径
$cd /media/openvpn-2.2.2/easy-rsa/2.0  生成密钥所需路径
```
#### 3.3.2 生成密钥  
```
$ source ./vars  
$ ./build-key-pass client-B    #此处设置密码为：openvpn123 
```
注：生成客户端key，pass表示需要输入一个密码作为客户端启动时的凭证； ./build-key则不需要输入密码  
```
$ ./build-dh  
```
#### 3.3.3设置客户端密钥验证信息  
```
$ vim /media/openvpn/conf/ccd/client-B  
ifconfig-push 10.8.0.120 255.255.255.0  
```
注：此处的验证信息文件名需要和生成密钥时输入的名字保持一致;  
10.8.0.120 指客户端被虚拟出来的IP  
### 3.4 客户端配置  
#### 3.4.1 拷贝密钥到客户端  
```
$scp root@xxx.xxx.xxx.xxx:/media/openvpn-2.2.2/easy-rsa/2.0/keys/{ca.crt,ca.key,client-B.crt,client-B.csr,client-B.key}   /etc/openvpn/conf
注:密钥可以由维护人员发放,联系刘东;
```
#### 3.4.2 配置客户端密码文件
```
$ vim /etc/openvpn/conf/password.txt  
openvpn123
```
注:客户端密码文件和服务端生成密钥时输入的密码一致
#### 3.4.3 客户端配置文件
```
$ vim /etc/openvpn/conf/client.conf  
client  
dev tap  
proto tcp  
remote xxx.xxx.xxx.xxx 1194   #指定服务端外网IP及端口

nobind  
user nobody  
group nogroup  
ca /etc/openvpn/conf/ca.crt  
cert /etc/openvpn/conf/client-B.crt  
key /etc/openvpn/conf/client-B.key  

ping 15  

ping-restart 45  
ping-timer-rem  
persist-key  
persist-tun  
ns-cert-type server  
comp-lzo  

verb 4  
log-append /etc/openvpn/log/openvpn.log  
status /etc/openvpn/log/openvpn-status.log  

tcp-queue-limit 4096 # 256  
bcast-buffers 4096  
```
### 3.5 启动客户端  
```
$ sudo openvpn --config /etc/openvpn/conf/client.conf --askpass /etc/openvpn/conf/password.txt --daemon  
```
### 3.6 检查验证
```
$ ifconfig|grep inet|grep 10.8.0.120  
inet 10.8.0.120 netmask 0xffffff00 broadcast 10.8.0.255  
```
注：得到IP为：10.8.0.x 则说明VPN客户端配置成功  
```
$ ping 10.8.0.1    #检查是否能ping通内网等机器
```

### 3.7 服务加入开机自启动
```
vim /etc/rc.local
openvpn --config /etc/openvpn/conf/client.conf --askpass /etc/openvpn/conf/password.txt --daemon
```
