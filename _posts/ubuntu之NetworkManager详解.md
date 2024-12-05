---
title: Ubuntu之NetworkManager详解
categories: linux
tags:
  - ubuntu
  - dns
  - nmcli
  - network
  - ip
  - config
  - centos
---
## 背景
NetworkManager是一个程序，用于为系统自动提供检测和配置以自动连接到网络。NetworkManager的功能对于无线和有线网络都非常有用。对于无线网络，NetworkManager首选已知的无线网络，并且能够切换到最可靠的网络。支持NetworkdManager的应用程序可以从联机和脱机模式切换。与无线连接相比NetworkManager更喜欢有线连接，它支持调制解调器连接和某些类型的VPN。NetworkManager最初是由RedHat开发的，现在由GNOME项目托管。
NetworkManager服务是管理和监控网络设置的守护进程，是2004年RedHat启动的项目，皆在能够让Linux用户更轻松的处理现代网络需求，尤其是无线网络，能够自动发现网卡并配置IP地址。Centos7之前的版本都是通过network.service管理网络配置。到了Centos7就同时支持network.service和NetworkManager.service(简称NM)，相当于在Centos7的一个过渡，默认情况下这2个服务都有开启，但是因为NetworkManager.service当时的兼容性不好，大部分人都会将其关闭。在RHEL 8/Centos 8上已废弃network.service（默认不安装），只能通过NetworkManager进行网络配置。

NetworkManager主要管理2个对象： Connection（网卡连接配置） 和 Device（网卡设备），他们之间是多对一的关系，但是同一时刻只能有一个Connection对于Device才生效。

在RHEL 8/Centos 8有三种方法配置网络：

- 通过nmcli connection add命令配置，会自动生成ifcfg文件。
- 手动配置ifcfg文件，通过nmcli connection reload来加载生效。
- 手动配置ifcfg文件，通过传统network.service来加载生效。

## 查看
在Linux中默认以systemd的形式运行，执行`systemctl  status network-manager.service`Or`systemctl status NetworkManager.service`查看信息：
```shell
# systemctl status network-manager.service  
# systemctl status NetworkManager.service
● NetworkManager.service - Network Manager  
   Loaded: loaded (/lib/systemd/system/NetworkManager.service; enabled; vendor preset: enabled)  
   Active: active (running) since Sun 2019-10-13 19:38:49 CST; 1h 21min ago  
     Docs: man:NetworkManager(8)  
 Main PID: 862 (NetworkManager)  
    Tasks: 4 (limit: 4915)  
   CGroup: /system.slice/NetworkManager.service  
           ├─ 862 /usr/sbin/NetworkManager --no-daemon  
           └─1930 /sbin/dhclient -d -q -sf /usr/lib/NetworkManager/nm-dhcp-helper -pf /run/dhclient-wlp3s0.pid -lf /var/lib/NetworkManager/dhclient-9ec39b13-6d0c-468c-9a5b-fc4a2a6ea85e-wlp3s0.lease -cf /var/lib/NetworkManager/dhclient-wlp3s0.conf wlp3s
```
## 使用
NetworkManager分为两个程序，一个`/usr/sbin/NetworkManager`的服务端，一个`/sbin/dhclient`客户端。

NetworkManager的配置工具有多种形式，如下：

1. nmcli：命令行。这是最常用的工具。
2. nmtui：在shell终端开启文本图形界面。
3. nm-applet：GUI界面配置工具。

### nmcli使用方法

nmcli使用方法非常类似linux ip命令、cisco交换机命令，并且支持tab补全，也可在命令最后通过-h、–help、help查看帮助。在nmcli中有2个命令最为常用：

#### nmcli connection

译作`连接`，可理解为配置文件，相当于ifcfg-ethX。可以简写为nmcli c  
connection有2种状态：  
▷ 活跃（带颜色字体）：表示当前该connection生效  
▷ 非活跃（正常字体）：表示当前该connection不生效

#### nmcli device

译作`设备`，可理解为实际存在的网卡（包括物理网卡和虚拟网卡）。可以简写为nmcli d  
device有4种常见状态：  
▷ connected：已被NM纳管，并且当前有活跃的connection  
▷ disconnected：已被NM纳管，但是当前没有活跃的connection  
▷ unmanaged：未被NM纳管  
▷ unavailable：不可用，NM无法纳管，通常出现于网卡link为down的时候（比如ip link set ethX down）
#### nmcli status code
nmcli 如果成功退出状态值为0，如果发生错误则返回大于0的值。

```text
0: 成功-指示操作已成功 
1: 位置或指定的错误 
2: 无效的用户输入，错误的nmcli调用 
3: 超时了（请参阅 --wait 选项） 
4: 连接激活失败 
5: 连接停用失败 
6: 断开设备失败 
7: 连接删除失败 
8: 网络管理器没有运行 
10: 连接、设备或接入点不存在 
65: 当使用 --complete-args 选项，文件名应遵循。
```
#### 常用命令
```shell
#查看ip（类似于ifconfig、ip addr）  
nmcli  
# 创建connection，配置静态ip（等同于配置ifcfg，其中BOOTPROTO=none，并ifup启动）  
nmcli c add type ethernet con-name ethX ifname ethX ipv4.addr 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.method manual  
# 创建connection，配置动态ip（等同于配置ifcfg，其中BOOTPROTO=dhcp，并ifup启动）  
nmcli c add type ethernet con-name ethX ifname ethX ipv4.method auto  
# 修改ip（非交互式）  
nmcli c modify ethX ipv4.addr '192.168.1.200/24'  
nmcli c up ethX  
# 修改ip（交互式）  
nmcli c edit ethX  
nmcli> goto ipv4.addresses  
nmcli ipv4.addresses> change  
Edit 'addresses' value: 192.168.1.200/24  
Do you also want to set 'ipv4.method' to 'manual'? [yes]: yes  
nmcli ipv4> save  
nmcli ipv4> activate  
nmcli ipv4> quit  
# 启用connection（相当于ifup）  
nmcli c up ethX  
# 停止connection（相当于ifdown）  
nmcli c down  
# 删除connection（类似于ifdown并删除ifcfg）  
nmcli c delete ethX  
# 查看connection列表  
nmcli c show  
# 查看connection详细信息  
nmcli c show ethX  
# 重载所有ifcfg或route到connection（不会立即生效）  
nmcli c reload  
# 重载指定ifcfg或route到connection（不会立即生效）  
nmcli c load /etc/sysconfig/network-scripts/ifcfg-ethX  
nmcli c load /etc/sysconfig/network-scripts/route-ethX  
# 立即生效connection，有3种方法  
nmcli c up ethX  
nmcli d reapply ethX  
nmcli d connect ethX  
# 查看device列表  
nmcli d  
# 查看所有device详细信息  
nmcli d show  
# 查看指定device的详细信息  
nmcli d show ethX  
# 激活网卡  
nmcli d connect ethX  
# 关闭无线网络（NM默认启用无线网络）  
nmcli r all off  
# 查看NM纳管状态  
nmcli n  
# 开启NM纳管  
nmcli n on  
# 关闭NM纳管（谨慎执行）  
nmcli n off  
# 监听事件  
nmcli m  
# 查看NM本身状态  
nmcli  
# 检测NM是否在线可用  
nm-online
```
#### 使用示例
##### 例子1：创建一个连接（connection）
```shell
nmcli c add type ethernet con-name ethX-test ifname ethX ipv4.addresses '192.168.1.100/24,92.168.1.101/32' ipv4.routes '10.0.0.0/8 192.168.1.10,192.168.0.0/16 192.168.1.11' ipv4.gateway 192.168.1.254 ipv4.dns '8.8.8.8,4.4.4.4' ipv4.method manual
```
- type ethernet：创建连接时候必须指定类型，类型有很多，可以通过 nmcli c add type-h看到，这里指定为ethernet。  
- con-name ethX ifname ethX：第一个ethX表示连接（connection）的名字，这个名字可以任意定义，无需和网卡名相同；第二个ethX表示网卡名，这个ethX必须是在 nmcli d里能看到的。  
- ipv4.addresses ‘192.168.1.100/24,192.168.1.101/32’：配置2个ip地址，分别为192.168.1.100/24和192.168.1.101/32  
- ipv4.gateway 192.168.1.254：网关为192.168.1.254  
- ipv4.dns ‘8.8.8.8,4.4.4.4’：dns为8.8.8.8和4.4.4.4  
- ipv4.method manual：配置静态IP

##### 例子2：对IP地址的操作
【**修改**】这些属性可以用nmcli connection show bond-bond0进行获取，然后可以修改、添加或删除属性，若要设置属性，只需指定属性名称后跟值，空值将删除属性值，同一属性添加多个值使用+。同一属性删除指定值用-加索引（**modify**）
**添加多个ip**（+ipv4.addresses）
```shell
# 添加三个ip $ nmcli connection modify bond-bond0 +ipv4.addresses 192.168.182.102/24 $ nmcli connection modify bond-bond0 +ipv4.addresses 192.168.182.103/24 $ nmcli connection modify bond-bond0 +ipv4.addresses 192.168.182.104/24 # 查看，查不到。 $ nmcli -f IP4 connection show bond-bond0 IP4.ADDRESS[1]: 192.168.182.152/24 IP4.GATEWAY: 192.168.182.2 IP4.ROUTE[1]: dst = 192.168.182.0/24, nh = 0.0.0.0, mt = 300 IP4.ROUTE[2]: dst = 0.0.0.0/0, nh = 192.168.182.2, mt = 300 IP4.DNS[1]: 8.8.8.8 # 启用配置 $ nmcli connection up bond-bond0 # 再次查看，就可以查到刚才添加的三个ip $ nmcli -f IP4 connection show bond-bond0 IP4.ADDRESS[1]: 192.168.182.152/24 IP4.ADDRESS[2]: 192.168.182.102/24 IP4.ADDRESS[3]: 192.168.182.103/24 IP4.ADDRESS[4]: 192.168.182.104/24 IP4.GATEWAY: 192.168.182.2 IP4.ROUTE[1]: dst = 192.168.182.0/24, nh = 0.0.0.0, mt = 300 IP4.ROUTE[2]: dst = 192.168.182.0/24, nh = 0.0.0.0, mt = 300 IP4.ROUTE[3]: dst = 192.168.182.0/24, nh = 0.0.0.0, mt = 300 IP4.ROUTE[4]: dst = 192.168.182.0/24, nh = 0.0.0.0, mt = 300 IP4.ROUTE[5]: dst = 0.0.0.0/0, nh = 192.168.182.2, mt = 300 IP4.DNS[1]: 8.8.8.8
```
**删除指定ip**（-ipv4.addresses）
```shell
# 删除刚刚添加的三个ip $ nmcli connection modify bond-bond0 -ipv4.addresses 192.168.182.102/24 $ nmcli connection modify bond-bond0 -ipv4.addresses 192.168.182.103/24 $ nmcli connection modify bond-bond0 -ipv4.addresses 192.168.182.104/24 # 启用配置 $ nmcli connection up bond-bond0 # 查看 $ nmcli -f IP4 connection show bond-bond0
```
【**添加**】这是创建一个新的连接，需要指定新创建连接的属性，语法与modify相同（**add**）
```shell
# 查看当前所有连接，c是connection的缩写 $ nmcli c show $ nmcli con add con-name eth1 type ethernet autoconnect yes ifname ens33 $ nmcli c show # con-name 连接名称 # type 连接类型 # autoconnect 是否自动连接 # ifname 连接到的设备名称
```
##### 例子3：对DNS的配置
###### 方式1：修改or添加NM的配置文件
修改/etc/NetworkManager/conf.d/dns.conf(如没有此文件，新建即可),增加如下两行配置： 
```conf
[main]
dns=null
```
 修改/etc/NetworkManager/conf.d/dns-servers.conf(如没有此文件，新建即可),增加如下两行配置,设置自定义dns： 
```conf
[global-dns-domain-*]
servers=::1,127.0.0.1,8.8.8.8
```
  重启软件
```shell
sudo systemctl restart NetworkManager
```
###### 方式2：配置时指定网络设备的DNS
```shell
#显示当前网络连接
nmcli connection show
# NAME UUID TYPE DEVICE 
# eno1 5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03 802-3-ethernet eno1
#修改当前网络连接对应的DNS服务器，这里的网络连接可以用名称或者uuid来标识
nmcli con mod eno1 ipv4.dns "223.5.5.5 8.8.8.8"
#应用DNS配置，使之生效
nmcli con up eno1
```