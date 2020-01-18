# IPv6 only VPS 安装 V2ray
# 介绍
这篇文章主要内容是在IPv6 only的VPS上进行安装V2ray，并且通过Cloudflare使普通的IPv4网络也可以进行访问。  
下面的一些前提步骤网络上都有比较详细的文章，可以去参考，本文主要针对IPv6的部分进行详细讲解。  
如有任何问题欢迎指出。

# 环境
VPS操作系统：Centos7  

# 前提条件
首先完成以下步骤  
1. 购买一个国外VPS，镜像最好选择Centos7
1. 购买一个域名（例如xxx.yyy.com），DNS解析指向VPS（DNS解析中添加类型为AAAA的记录，值为VPS的IPv6地址）

下面命令中所有出现xxx.yyy.com的地方都替换为DNS中指向VPS的域名

# 手动安装
通过ssh连接上VPS，然后切换到root用户，我们开始执行下面的命令

## 关闭防火墙（如果已开启）
>systemctl stop firewalld  
>systemctl disable firewalld

## 安装相关组件
>yum -y install bind-utils wget unzip zip curl tar  
## 安装nginx
>rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm  
>yum -y install nginx
>systemctl enable nginx.service

## 下载一个伪装的网站源码
>rm -rf /usr/share/nginx/html/*  
>cd /usr/share/nginx/html/  
>wget https://github.com/atrandys/v2ray-ws-tls/raw/master/web.zip  
>unzip web.zip
## 下载支持IPv6的nginx配置文件
>cd /etc/nginx/conf.d/  
>rm * -rf  
>wget https://github.com/whosy/v2ray-config/blob/master/http.conf

### 注意：在上面这一步进行完之后，要手动修改http.conf中的server_name为你自己购买的域名
## 重启nginx
>systemctl restart nginx.service

## 申请https证书，把下面命令中的xxx.yyy.com替换为你自己购买的域名
>mkdir /usr/src/cert  
>curl https://get.acme.sh | sh  
>~/.acme.sh/acme.sh --issue -d xxx.yyy.com --webroot /usr/share/nginx/html/  
>~/.acme.sh/acme.sh --installcert -d xxx.yyy.com --key-file /usr/src/cert/private.key --fullchain-file /usr/src/cert/fullchain.cer  
>wget https://github.com/whosy/v2ray-config/blob/master/https.conf
### 注意：在上面这一步进行完之后，要手动修改https.conf中的server_name为你自己购买的域名
## 重启nginx
>systemctl restart nginx.service

这个时候在浏览器里输入https://xxx.yyy.com应该能够看到一个网站了

## 安装v2ray
>curl -L -s https://install.direct/go.sh  
>cd /etc/v2ray/  
>wget https://github.com/whosy/v2ray-config/blob/master/config.json  
>systemctl enable v2ray  
>systemctl start v2ray  

## 客户端配置
客户端这里以windows系统下的v2rayN软件为例。  
首先打开软件后，添加VMess服务器，如下图，用户id、额外id可以在config.json中自己设置，但是要保证服务端和客户端中使用的是一样的。  

![配置图片](https://github.com/whosy/v2ray-config/blob/master/client_config.png)

## 套上Cloudflare
注册Cloudflare账号，然后把原来域名的dns解析服务器设置为Cloudflare提供的dns解析服务器，然后开启Cloudflare的cdn，小云朵变为橙色。具体内容可以参考网络资料，很简单。

## 总结
本文中大部分命令是参考了Trojan的一键安装脚本，由于这个脚本只支持IPv4命令，所以在IPv6 only的VPS上是无法使用的，所以我们只能把脚本中的命令拆分开来运行。  
还有就是nginx的配置文件中，要加上IPv6的端口监听，网络上大部分资料都只监听IPv4端口，这个要注意下。  

# Over！
