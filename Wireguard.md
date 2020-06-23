# wiregard

###### [官网](https://www.wireguard.com/install/)
###### [参考](https://withdewhua.space/2018/11/10/wireguard#%E5%AE%89%E8%A3%85-wireguard)
> install 
```
sudo apt-get install wireguard

or # 如果不可以直接安装，通过加源安装。

echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable.list
printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstable
apt update
apt install wireguard -y
apt install resolvconf -y
```
> Creat wireguard file 
```
mkdir -p /etc/wireguard && chmod 0777 /etc/wireguard
cd /etc/wireguard
umask 077
```
> generate private and public
```
wg genkey | tee server_privatekey | wg pubkey > server_publickey
wg genkey | tee client_privatekey | wg pubkey > client_publickey
```
## 服务器配置文件

> vim /etc/wireguard/wg0.conf
```
# 重要！如果名字不是eth0, 以下PostUp和PostDown处里面的eth0替换成自己服务器显示的名字
# ListenPort为端口号，可以自己设置想使用的数字
# 以下内容一次性粘贴执行，不要分行执行
echo "
[Interface]
  PrivateKey = $(cat server_privatekey)
  Address = 10.0.0.1/24
  PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
  ListenPort = 50814
  DNS = 8.8.8.8
  MTU = 1420

[Peer]
  PublicKey = $(cat client_publickey)
  AllowedIPs = 10.0.0.2/32 " > wg0.conf
```
> 是否启动开机自启
```
systemctl enable wg-quick@wg0

systemctl disable wg-quick@wg0
```
## 客户端配置文件

> vim /etc/wireguard/wg0.conf
```
# Endpoint是自己服务器ip和服务端配置文件中设置的端口号，自己在本地编辑好再粘贴到SSH里
# 以下内容一次性粘贴执行，不要分行执行
echo "
[Interface]
  PrivateKey = $(cat client_privatekey)
  Address = 10.0.0.2/24
  DNS = 8.8.8.8
  MTU = 1420

[Peer]
  PublicKey = $(cat server_publickey)
  Endpoint = 1.2.3.4:50814
  AllowedIPs = 0.0.0.0/0, ::0/0
  PersistentKeepalive = 25 " > wg0.conf
```
> 启动与关闭
```
# 启动WireGuard
wg-quick up wg0

# 停止WireGuard
wg-quick down wg0

# 查看WireGuard运行状态
wg
```
> 查看并导出配置文件 这是二维码方式打开
```
apt install qrencode -y
qrencode -t ansiutf8 < /etc/wireguard/wg0.conf
```
# 范文
###### server 
```
[Interface]
  PrivateKey = IJu+ssDn3evBmNSsds3PM7ZwXyXxbOVbU6iT/Z9UzEo=
  Address = 192.168.100.1/24
  PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp2s0 -j MASQUERADE
  PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp2s0 -j MASQUERADE
  ListenPort = 50814
  DNS = 8.8.8.8
  MTU = 1400

[Peer]
  PublicKey = U9MU+Pgo4KK9guU55ErJPs/r9Qfljp0Zj+csaeBcXgQ=
  AllowedIPs = 192.168.100.2/32
```
###### Client
```
[Interface]
  PrivateKey = iDIcLwUYsdfglwdRngUsLafcga+0upKBAsmajhWI2s=
  Address = 192.168.100.2/24
  DNS = 8.8.8.8
  MTU = 1400

[Peer]
  PublicKey = nEQ2Rzd053CMUzrePg93R1Umz4UL3JBKEwdUEE1ASo=
  Endpoint = 10.0.9.160:50814
  AllowedIPs = 192.168.100.0/24
  PersistentKeepalive = 25
  ```
