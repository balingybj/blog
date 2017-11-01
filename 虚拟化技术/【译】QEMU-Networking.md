>   本文翻译自QEMU 的 wiki 页面[QEMU/Networking](https://en.wikibooks.org/wiki/QEMU/Networking)。

# QEMU/Networking

QEMU 通过模拟一些流行的网卡、创建[虚拟局域网](https://en.wikipedia.org/wiki/Virtual_LAN)来支持网络功能。有四种方式可以将 QEMU 客户机连接起来：用户模式、套接字重定向、Tap 和 VED 网络。

## 用户模式网络

如果不指定任何网络选项，QEMU 将默认模拟一个 Intel e1000 PCI 网卡、一个用户模式网络栈并桥接到宿主机网络。以下三条命令是等价的：

```shell
$ qemu -m 256 -hda disk.img &
```

```shell
$ qemu -m 256 -hda disk.img -net nic -net user &
```

```shell
$ qemu-system-i386 -m 256 -hda disk.img -netdev user,id=network0 -device e1000,netdev=network0,mac=52:54:00:12:34:56 &
```

要在 Linux 内核中使用该设置，你必须在编译时设置`CONFIG_E1000=y`选项。

`-net`选项在新的 QEMU 版本中被`-netdev`取代。

客户机系统将看到一张 E1000 网卡，一个虚拟的DHCP 服务运行在 10.0.2.2 上，并会分配一个从 10.0.2.14 的地址。一个虚拟的 DNS 服务运行在 10.0.2.3 上。一个虚拟的 SAMBA 文件服务运行在 10.0.2.4 上，该服务允许客户机访问宿主机上的文件。

用户模式网络很适合访问网络资源，包括 internet。特别地，用户模式允许从客户机 ssh 到宿主机。但是在默认情况下，用户模式网络充当了一个防火墙，它不允许任何进入到客户机的流量。它不支持除了 TCP 和 UDP 以外的协议，例如，不支持 ping 和其他 ICMP 工具。

### 端口重定向

为了在用户模式网络下从网络连接到客户机系统，你可以将宿主机的端口重定向到客户机的端口。可用于文件共享，在客户机中运行 web server 和 SSH server。

下面的示例展示了如何设置启动 QEMU，在用户模式网络下让一个 Windows XP 客户机共享文件和 web 页面。宿主机上的 TCP 端口 5555 被重定向到客户机的端口 80（web 服务），宿主机的 TCP 端口 5556 被重定向到客户机的端口 445（Windows 网络）：

```shell
$ qemu -m 256 -hda disk.img -redir tcp:5555::80 -redir tcp:5556::445 &
...
$ mkdir -p /mnt/qemu
$ mount -t cifs //localhost/someshare /mnt/qemu -o user=test,pass=test,dom=workgroup,port=5556
$ firefox http://localhost:5555/
```

注意：当通过 Windows 网络共享客户机中的目录时，你必须指定 mount 操作登录客户机需要的用户名和密码。如果指定密码，mount 操作将会失败，返回一个 I/O 错误。

## TAP 接口

QEMU 可利用 TAP 接口为客户机系统提供完整的网络功能。适用于客户机系统运行着好几个网络服务，并且必须通过标准的端口连接，需要除了 TCP 和 UDP 以外的协议，并且多个 QEMU 示例需要彼此相连（尽管这也可以通过用户模式网络下的端口重定向或 socket 实现）。

在 QEMU 1.1 及更高版本中， [network bridge helper](http://wiki.qemu.org/Features/HelperNetworking) 可以为你自动设置 tun/tap，而不需要额外的脚本。

对应旧版本，设置 TAP 接口比用户模式网络稍微复杂一点。需要在宿主机上安装一个虚拟私有网络（ VPN），然后在宿主机网络和虚拟网络之间创建一个 bridge。 

下面示例的环境为 Fedora 8，并设置了静态 IP。该过程在其他 Linux 发行版上也是类似的。

### TAP/TUN 设备

根据  [tuntap.txt](http://www.kernel.org/doc/Documentation/networking/tuntap.txt)，我们先创建 TAP/TUN 设备：

```shell
$ sudo mkdir /dev/net
$ sudo mknod /dev/net/tun c 10 200
$ sudo /sbin/modprobe tun
```

### qemu-ifup

首先，设置一个脚本来创建 bridge 并打开 TAP 接口。我们称该脚本为`/etc/qemu-ifup`。

```shell
#!/bin/sh 
# 
# 该脚本用于在 QEMU 中的桥接模式下打开 tun 设备
# 第一个参数是 tap 设备的名称（例如，tap0）
# 有一些常量特定于你的宿主机，请根据情况修改它们
#
ETH0IPADDR=192.168.0.3
MASK=255.255.255.0
GATEWAY=192.168.0.1
BROADCAST=192.168.0.255
#
# 先关闭 eth0，然后用 IP 0.0.0.0 启动它
#
/sbin/ifdown eth0
/sbin/ifconfig eth0 0.0.0.0 promisc up
#
# 打开 tap 设备 （名称由 QEMU 传入的第一个参数指定）
#
/usr/sbin/openvpn --mktun --dev $1 --user `id -un`
/sbin/ifconfig $1 0.0.0.0 promisc up
#
# 在 eht0 和 tap 设备之间创建一个 bridge
# 
/usr/sbin/brctl addbr br0
/usr/sbin/brctl addif br0 eth0
/usr/sbin/brctl addif br0 $1
# 
# 只有一个 bridge，所以不可能出现环，关闭生成树协议（译者注：该协议用于检测是否出现环）
#
/usr/sbin/brctl stp br0 off 
# 
# 启动 bridge给其分配 IP 地址 ETH0IPADDR，并添加默认路由
#
/sbin/ifconfig br0 $ETH0IPADDR netmask $MASK broadcast $BROADCAST
/sbin/route add default gw $GATEWAY
#
# 关闭防火墙 - 如果你没使用防火墙，注释掉本条命令
#
/sbin/service firestarter stop 
```

### qemu-ifdown

你还需要一个脚本在 QEMU 退出后重置网络。为了保持一致性，我们称该脚本为`/etc/qemu-ifdown`。

```shell
#!/bin/sh 
# 
# 该脚本用于在 QEMU 退出后关闭并删除网桥 br0
#
# 关闭 eth0 和 br0
#
/sbin/ifdown eth0
/sbin/ifdown br0
/sbin/ifconfig br0 down 
# 
# 删掉 br0
#
/usr/sbin/brctl delbr br0 
# 
# 以“正常”模式启动 eth0
#
/sbin/ifconfig eth0 -promisc
/sbin/ifup eth0 
#
# 删掉 tap 设备
#
/usr/sbin/openvpn --rmtun --dev $1
#
# 重新启动防火墙
# 
/sbin/service firestarter start 
```

### 允许用户调用脚本

QEMU 1.1 及更高版本使用 [helper program](http://wiki.qemu.org/Features/HelperNetworking#Setup) 就行，根本不需要任何脚本，并可 setuid root。

对于旧版本，上面的两个脚本需要用 superuser（超级用户）身份执行，以便它们修改系统的网络设置。最便利的方法是允许 QEMU 用户使用`sudo`命令调用脚本。设置方法是在`/etc/sudoers`中添加如下内容：

```shell
User_Alias QEMUERS = fred, john, milly, ...

Cmnd_Alias QEMU = /etc/qemu-ifup, /etc/qemu-ifdown

QEMUERS ALL=(ALL) NOPASSWD: QEMU
```

### 使用 TAP 接口启动 QEMU

现在我们创建一个脚本启动 QEMU，创建一个 VLAN，并在其退出时做清理。使用的 TAP 设备名称为 tap0。通过指定`script=no`告知 QEMU 直接使用 tap 设备而无需调用脚本 -  这样可以以普通用户执行 QEMU，而无需 root 用户。

```shell
#!/bin/sh 
sudo /etc/qemu-ifup tap0
qemu -m 256 -hda disk.img -net nic -net tap,ifname=tap0,script=no,downscript=no
sudo /etc/qemu-ifdown tap0
```

执行该脚本，它将创建一个 TAP 设备，桥接它到 eth0，启动 QEMU，在退出时移除 bridge 和 TAP 设备。

### Windows Vista 及以上 – 网络位置

Windows Vista 及更高版本将网络连接分为共有和私有。该分类决定了连接上的防火墙规则。Windows 维护了一个已知用户列表，如果它发现一个网络连接不在该表中，它就提示用户指出该连接是否为“家庭”、“工作组”或者“公共”网络。网络由其默认网关的 MAC 地址标识，但 QEMU 每次启动时都会随机分配该 MAC 地址。这就造成每次 用 QEMU 启动 Windows 会话时都会弹出一个窗口要求你指出“网络位置”。这个问题不严重，但是很烦。

解决办法是强迫 netdev 接口总是使用同一个 MAC 地址。QEMU 好像不提供这种设置选项。但可以在 ifup 脚本中设置。通过使用已经取代 ifconfig 的 Iproute2，命令如下：

```shell
$ ip link set dev tapn address 52:54:00:12:34:56
```

它将宿主机端的接口的 MAC 地址改为设定值，该值可以为本地网络中任何合法的、唯一的 MAC 地址。

## 套接字

QEMU 可以通过 TCP 会 UDP 套接字在 VLAN 中连接多个 QEMU 客户机系统。

[描述在这里](http://www.gnome.org/~markmc/qemu-networking.html)

>   该链接页面我也翻译了（我果然是翻译狂魔），见[【译】QEMU 网络](./【译】QEMU 网络.md)

## SMB 服务

如果宿主机系统安装了[SMB](https://en.wikipedia.org/wiki/Server_Message_Block) 服务（*nix 系统上为 SAMBA/CIFS）,QEMU 可以为使用了`-smb`选项的系统模拟一个 SMB 服务。指定要共享的目录，该目录将作为`\\10.0.2.4\qemu`提供给客户机（或者你可以将 10.0.2.4 放到宿主机或 lmhosts 文件中，并映射到`\\smbserver\qemu`）。

```shell
$ qemu -m 256 disk.img -smb /usr/workspace/testing01
```

该设置并非必须，因为 QEMU 中的客户机通常可以访问宿主机中的 SMB 服务。但是，在不需要通过为每个客户机配置 SMB 共享的情况下，为每个客户机设置独立的工作空间是非常有用的。

>   译者：这段的意思是说这样设置就不需要进入每个客户系统单独设置了吗？

## 额外链接

>   见原文

## 参考

>   见原文

