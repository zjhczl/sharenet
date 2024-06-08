# 共享网络
参考：http://wiki.neardi.com/wiki/linux_guide/zh_CN/docs/demo/demo_hostapd.html
## 无线网络共享
通过无线网卡创建wifi共享网络

### 安装hostapd开启热点
#### 安装hostapd
```
sudo apt install hostapd

```
#### 配置hostapd

sudo vim /etc/hostapd/hostapd.conf
```
interface=wlan0
driver=nl80211
ssid=neardi_rk3568
hw_mode=a
channel=36
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```
sudo vim /etc/default/hostapd,在末尾添加一行
```
# Note that -B (daemon mode) and -P (pidfile) options are automatically
# configured by the init.d script and must not be added to DAEMON_OPTS.
#
#DAEMON_OPTS=""
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
重启服务
```
sudo systemctl unmask hostapd
sudo systemctl restart hostapd
```
### DHCP Server
#### 安装DHCP server
```
sudo apt install isc-dhcp-server

```

### 配置dhcp
#### 设置无线网卡ip地址（网关地址）
```
sudo ifconfig wlan0 192.168.200.1
```
永久设置则编辑 /etc/network/interfaces设置静态ip

sudo vim etc/default/isc-dhcp-server在末尾添加
```
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 usb0".
INTERFACESv4="wlan0"
INTERFACESv6=""
```
sudo vim /etc/dhcp/dhcpd.conf 在末尾添加
```              
subnet 192.168.200.0 netmask 255.255.255.0 {                          
	range 192.168.200.100 192.168.200.200;                  
	option routers 192.168.200.1;           
	option domain-name-servers 114.114.114.114, 8.8.8.8;           
}       
```

```
sudo systemctl restart isc-dhcp-server
```
### 共享网络
#### 开启ip转发
```
#开启网络转发功能
sudo echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
#网络转发的功能生效
sudo sysctl -p

```
#### 网卡流量桥接
```
#将从usb0网卡输出的数据包进行源地址伪装，让其他设备通过usb0网卡访问外部网络，而不会被外部网络拒绝
sudo iptables -t nat -A POSTROUTING -o usb0 -j MASQUERADE
#允许从usb0网卡输入的数据包，若状态是RELATED或者ESTABLISHED，转发到wlan0网卡输出，让usb0网卡收到的响应数据包返回到wlan0
sudo iptables -A FORWARD -i usb0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
#允许从wlan0网卡输入的数据包，无条件地转发到usb0网卡输出，让wlan0网卡的设备通过usb0网卡访问外部网络
sudo iptables -A FORWARD -i wlan0 -o usb0 -j ACCEPT
```

#### 使用xx使iptables命令永久生效
```
sudo apt-get update
sudo apt-get install iptables-persistent
sudo netfilter-persistent save
```

## 有线网络共享
### 设置网关网卡eth0地址为静态ip
```
192.168.200.1
```
### 使用iptables转发流量
```
#将从usb0网卡输出的数据包进行源地址伪装，让其他设备通过usb0网卡访问外部网络，而不会被外部网络拒绝
sudo iptables -t nat -A POSTROUTING -o usb0 -j MASQUERADE

```
### 其他设备连接网卡eth0
其它设备设置网关为192.168.200.1

