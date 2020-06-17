---
layout: post
title: 'CentOS基于ss-redir实现透明代理'
date: 2018-05-16
author: 李大鹏
cover: ''
tags: CentOS
---

### 一、透明代理的实现原理
透明代理的意思是客户端根本不需要知道有代理服务器的存在，它改变你的request fields（报文），并会传送真实IP，多用于路由器的NAT转发中。注意，加密的透明代理则是属于匿名代理，意思是不用设置使用代理了。
#### 1. 代理结构1
![](http://files.pandaleo.cn/de025760c2a0cb8ad31b57e1b27806fb.png)
#### 2. 内部原理
![](http://files.pandaleo.cn/120eed1cfd39ea48dc0e9dde7b312526.png)

### 二、安装应用
#### 1. 安装shadowsocks-libev
```
## 添加shadowsocks-libev 的 yum-repo
yum-config-manager --add-repo https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
yum makecache
yum install shadowsocks-libev -y
```

#### 2. 安装chinadns
```
## 获取 chinadns 源码
wget https://github.com/shadowsocks/ChinaDNS/releases/download/1.3.2/chinadns-1.3.2.tar.gz
## 解压 chinadns 源码
tar xf chinadns-1.3.2.tar.gz

## 编译 chinadns
cd chinadns-1.3.2/
./configure
make && make install

## chinadns 相关文件
mkdir /etc/chinadns/
cp -af chnroute.txt /etc/chinadns/
cp -af iplist.txt /etc/chinadns/
```
#### 3. 安装dnsforwarder
```
## 获取 dnsforwarder 源码
git clone https://github.com/holmium/dnsforwarder.git

## 编译 dnsforwarder
cd dnsforwarder/
./configure
make && make install

## 初始化 dnsforwarder
dnsforwarder -p
cp -af default.config ~/.dnsforwarder/config
```
#### 4. 安装ipset
```
yum install ipset -y
```
### 三、配置部署
#### 1. ss-redir
```
nohup ss-redir -s 服务器IP -p 服务器Port -m 加密方式 -k 密码 -b 0.0.0.0 -l 60080 -u < /dev/null &>> /var/log/ss-redir.log &
```
#### 2. ss-tunnel
```
nohup ss-tunnel -s 服务器IP -p 服务器Port -m 加密方式 -k 密码 -b 0.0.0.0 -l 60053 -L 8.8.8.8:53 -u < /dev/null &>> /var/log/ss-tunnel.log &
```
#### 3. chinadns
```
nohup chinadns -b 0.0.0.0 -p 65353 -s 114.114.114.114,127.0.0.1:60053 -c /etc/chinadns/chnroute.txt -l /etc/chinadns/iplist.txt -m < /dev/null &>> /var/log/chinadns.log &
```
#### 4. dnsforwarder
```
## ~/.dnsforwarder/config 配置文件
## 把原来的配置文件清空，使用以下配置

++++++++++++++++++ config ++++++++++++++++++
#### 日志相关 ####
LogOn true # 启用日志
LogFileThresholdLength 5120000 # 日志大小临界值，大于该值则将原文件备份，使用新文件记录日志
LogFileFolder /var/log/ # 日志文件所在的文件夹

#### 监听地址 ####
UDPLocal 0.0.0.0:53 # 可以有多个，使用逗号隔开，默认端口53

#### 上游dns ####
UDPGroup 127.0.0.1:65353 * on # chinadns 作为上游 dns 服务器
BlockNegativeResponse true # 过滤上游 dns 未成功的响应

#### dns缓存 ####
UseCache true # 启用缓存（文件缓存）
MemoryCache false # 不使用内存缓存
CacheSize 30720000 # 缓存大小，不能小于 102400

IgnoreTTL true # 忽略 TTL 值

ReloadCache true # 启动时加载已有的文件缓存
OverwriteCache true # 当已有的文件缓存载入失败时，覆盖原文件
++++++++++++++++++ config ++++++++++++++++++

## 运行 dnsforwarder
dnsforwarder -d

## 查看运行状态
ps -ef | grep dnsforwarder
ss -lnp | grep :53
```
#### 5. ipset
```
# 获取大陆地址段
curl -sL http://f.ip.cn/rt/chnroutes.txt | egrep -v '^\s*$|^\s*#' > chnip.txt

# 添加 chnip 表
ipset -N chnip hash:net
for i in `cat chnip.txt`; do echo ipset -A chnip $i >> chnip.sh; done
bash chnip.sh

# 持久化 chnip 表
ipset -S chnip > /etc/ipset.chnip
```
#### 6. iptables
```
# 新建 mangle/SS-UDP 链，用于透明代理内网 udp 流量
iptables -t mangle -N SS-UDP

# 放行保留地址、环回地址、特殊地址
iptables -t mangle -A SS-UDP -d 0/8 -j RETURN
iptables -t mangle -A SS-UDP -d 127/8 -j RETURN
iptables -t mangle -A SS-UDP -d 10/8 -j RETURN
iptables -t mangle -A SS-UDP -d 169.254/16 -j RETURN
iptables -t mangle -A SS-UDP -d 172.16/12 -j RETURN
iptables -t mangle -A SS-UDP -d 192.168/16 -j RETURN
iptables -t mangle -A SS-UDP -d 224/4 -j RETURN
iptables -t mangle -A SS-UDP -d 240/4 -j RETURN

# 放行发往 ss 服务器的数据包，注意替换为你的服务器IP
iptables -t mangle -A SS-UDP -d 服务器IP -j RETURN

# 放行大陆地址
iptables -t mangle -A SS-UDP -m set --match-set chnip dst -j RETURN

# 重定向 udp 数据包至 60080 监听端口
iptables -t mangle -A SS-UDP -p udp -j TPROXY --tproxy-mark 0x2333/0x2333 --on-ip 127.0.0.1 --on-port 60080

# 内网 udp 数据包流经 SS-UDP 链
iptables -t mangle -A PREROUTING -p udp -s 192.168/16 -j SS-UDP

# 新建 nat/SS-TCP 链，用于透明代理本机/内网 tcp 流量
iptables -t nat -N SS-TCP

# 放行环回地址，保留地址，特殊地址
iptables -t nat -A SS-TCP -d 0/8 -j RETURN
iptables -t nat -A SS-TCP -d 127/8 -j RETURN
iptables -t nat -A SS-TCP -d 10/8 -j RETURN
iptables -t nat -A SS-TCP -d 169.254/16 -j RETURN
iptables -t nat -A SS-TCP -d 172.16/12 -j RETURN
iptables -t nat -A SS-TCP -d 192.168/16 -j RETURN
iptables -t nat -A SS-TCP -d 224/4 -j RETURN
iptables -t nat -A SS-TCP -d 240/4 -j RETURN

# 放行发往 ss 服务器的数据包，注意替换为你的服务器IP
iptables -t nat -A SS-TCP -d 服务器IP -j RETURN

# 放行大陆地址段
iptables -t nat -A SS-TCP -m set --match-set chnip dst -j RETURN

# 重定向 tcp 数据包至 60080 监听端口
iptables -t nat -A SS-TCP -p tcp -j REDIRECT --to-ports 60080

# 本机 tcp 数据包流经 SS-TCP 链
iptables -t nat -A OUTPUT -p tcp -j SS-TCP
# 内网 tcp 数据包流经 SS-TCP 链
iptables -t nat -A PREROUTING -p tcp -s 192.168/16 -j SS-TCP

# 内网数据包源 NAT
iptables -t nat -A POSTROUTING -s 192.168/16 -j MASQUERADE

# 持久化 iptables 规则
iptables-save > /etc/iptables.tproxy
```
#### 7. 策略路由
```
# 新建路由表 100，将所有数据包发往 loopback 网卡
ip route add local 0/0 dev lo table 100

# 添加路由策略，让所有经 TPROXY 标记的 0x2333/0x2333 udp 数据包使用路由表 100
ip rule add fwmark 0x2333/0x2333 lookup 100
```
#### 8. 网卡转发
```
# 检查是否已开启网卡转发功能
cat /proc/sys/net/ipv4/ip_forward
# 若为 1，则说明已开启，无需配置

# 若为 0，请进行下面两个步骤：
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```
#### 9. 修改dns
```
## 本机，使用 127.0.0.1:53/udp
cat /etc/resolv.conf
nameserver 127.0.0.1
```
### 四、代理测试
#### 1. 测试tcp
```
# 访问各大网站，若均有网页源码输出，说明 tcp 转发已成功
curl -sL www.baidu.com
curl -sL www.google.com
curl -sL www.google.com.hk
curl -sL www.google.co.jp
curl -sL www.youtube.com
curl -sL mail.google.com
curl -sL facebook.com
curl -sL twitter.com
curl -sL www.wikipedia.org
```
#### 2. 测试 udp
```
## 先使用 192.168.1.1:53 解析被污染的域名
dig @192.168.1.1 www.twitter.com
dig @192.168.1.1 www.twitter.com
dig @192.168.1.1 www.twitter.com
# 记录出现的几个 ip，因为是通过 ss-tunnel 解析的，因此这些 ip 都是未受污染的

## 再使用 OpenDNS 443/udp 端口进行 dns 解析
dig @208.67.222.222 -p 443 www.twitter.com
dig @208.67.222.222 -p 443 www.twitter.com
dig @208.67.222.222 -p 443 www.twitter.com
# 多执行几次，如果解析出来的 ip 和上面的一样，说明 udp 转发已成功
# 同时，查看 /var/log/ss-redir.log 日志可以看到 udp_relay 成功的信息
```
