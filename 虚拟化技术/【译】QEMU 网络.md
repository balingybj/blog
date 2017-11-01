>   本文翻译自[QEMU Networking](https://people.gnome.org/~markmc/qemu-networking.html)

# QEMU 网络

QEMU 有许多非常好的方式为客户机设置网络。但繁多的选项让人困惑，所以我把我发现的内容写下来。请原谅我这个”字符画艺术家“:-)

>   译者注：本文没有用图片，而是用字符画代替了。我很欣赏这种节约网络流量又简单的方法。

## VLAN（虚拟局域网）和网卡

一个 VLAN 是一个在 qemu 进程上下文中运行的网络交换机。在每个 qemu 进程中可以存在多个 vlans。每个 vlan 可以连接多个接口。通过 vlan 上一个相连接口上发送的内容都会被该 vlan 所有其他接口接收到。

最明显的接口就是用户为客户机 OS 创建的网卡。客户机 OS 通过该网卡发送的帧将出现在该网卡所属的 vlan 上。从其他接口发送到 vlan 的任何帧都会被该网卡收到，然后被客户机 OS 收到。

```
         +-----------+
         |   Guest   |
         |    OS     |
         |   +---+   |
         |   |NIC|   |
         +---+-+-+---+
               ^           +--------+
               |           |        +<------> Other IF
               +---------->+  VLAN  |
                           |        +<------> Other IF
                           +--------+
```

例如：

```shell
$ qemu -net nic,model=rtl8139,vlan=1,macaddr=52:54:00:12:34:56 ...
```

将创建一个 rtl8139 网卡，指定了其 MAC 地址，并连接到编号为1的 vlan。然后就可以使用更多的`-net`选项将其他接口连接到该 vlan。

## VLAN 相互连接

我们还可以将来自不同 qemu 进程中的 vlan 连接到一起。工作原理是一个 qemu 进程连接到另一个 qemu 进程中的 socket。当一个帧出现在前者中时，它将通过该 socket 转发帧到相连的 qemu 进程中，反之亦然。

```
 +-----------+                                      +-----------+
 |   Guest   |                                      |   Guest   |
 |     A     |                                      |     B     |
 |   +---+   |                                      |   +---+   |
 |   |NIC|   |                                      |   |NIC|   |
 +---+-+-+---+                                      +---+-+-+---+
       ^       +--------+                +--------+       ^
       |       |        |                |        |       |
       +------>+ VLAN 0 +<--> Socket <-->+ VLAN 2 +<------+
               |        |                |        |
               +--------+                +--------+
```

例如，你可以这样启动一个客户机 A：

```shell
$ qemu -net nic -net socket,listen=:8010 ...
```

该 qemu 进程承载一个客户机，该客户机的网卡连接到 VLAN 0，VLAN 0 上又创建一个 socket 在 8010 端口上监听连接。

然后你可以这样启动客户机 B：

```shell
$ qemu -net nic,vlan=2 -net socket,vlan=2,connect=127.0.0.1:8010 ...
```

该 qemu 进程承载的客户机有一个连接到 vlan 2 的网卡，vlan 2 上又创建了一个 socket 并连接到了前面 VLAN 0 中的 socket。

如此，任何客户机 A 发送的数据帧都会被客户机 B 收到，反之亦然。

（注意，这两个 vlan 不比使用不同的编号，这里只是为了便于说明）

这一概念的扩展是使用多播 socket 连接 vlans。例如：

```shell
$ qemu -net nic -net socket,mcast=230.0.0.1:1234 ...
$ qemu -net nic -net socket,mcast=230.0.0.1:1234 ...
```

这样你就有两个客户机，它们的 vlan 0 通过一个多播 bus（总线）相互连接。任意数量的客户机都可以连接到该多播地址，并接收到任何客户机发送到该 vlan 的数据帧。

## 将 VLAN 连接到 TAP 设备

另一种方法是通过宿主机中的**设备**访问 vlan。通过该设备传输的任何帧都会出现在对应 qemu 进程中的 vlan 上（这样就可以被该 vlan 中的其他接口收到了），并且任何发生到该 vlan 的帧都会被该设备收到。

```
 +-----------+                            +-------+
 |   Guest   |                            |  TAP  |
 |    OS     |                            | Device| 
 |   +---+   |                            |(qtap0)|
 |   |NIC|   |                            +---+---+
 +---+-+-+---+                                |   
       ^       +------+                   +---+-----+
       |       |      |                   | Kernel  |
       +------>+ VLAN +<-->   File    <-->+ TUN/TAP |
               |      |     Descriptor    | Driver  |
               +------+                   +---------+
```

```shell
$ qemu -net nic -net tap,ifname=qtap0 ...
```

该选项使用到了内核中的 TUN/TAP 设备驱动。该驱动允许用户空间应用获取一个文件描述符，该文件描述符关联了一个网络设备。任何通过该文件描述符发送到内核的帧都会被该设备收到，任何传输到该设备的帧都会被应用收到。

如果你给对应的 TAP 设备分配一个 IP 地址，客户机中的应用程序就可以连接到宿主机上在该 IP 地址监听的应用程序。并且，如果你在宿主机中允许端口转发，从客户机发送出来的数包就可以被宿主机内核转发到 internet。

本质上，TAP 设备就像一个连接到物理网络的网络设备，客户机也可以连接到该设备上。

TAP 设备通过打开`/dev/net/tun`和调用 TUNSETIFF ioctl() 获得。通常非特权用户不能这么做，一般只有 root 用户能用这种方法。

## 使用 VDE 连接 VLAN

前一个想法的扩展是使用[VDE (Virtual Distributed Ethernet)](http://vde.sf.net/)。这是一个用户空间程序，它能获取一个 TAP 设备，并允许多个其他程序连接它，并将这些连接桥接到 TAP 设备。实际上，这种方式非常类似于将多个 qemu vlan 连接到一起，并将其中一个 vlan 连接到一个 TAP 设备。

```
    +-----------+                           +-----------+
    |   Guest   |                           |   Guest   |
    |     A     |                           |     B     |
    |   +---+   |                           |   +---+   |
    |   |NIC|   |                           |   |NIC|   |
    +---+-+-+---+                           +---+-+-+---+
          ^       +------+         +------+       ^
          |       |      | +-----+ |      |       |
          +------>+ VLAN +-+ VDE +-+ VLAN +<------+
                  |      | +--+--+ |      |
                  +------+    |    +------+
                              |
                          +---+-----+  +--------+
                          | Kernel  |  |  TAP   |
                          | TUN/TAP +--+ Device |
                          | Driver  |  | (qtap) |
                          +---------+  +--------+
```

```shell
$ vde_switch -hub -tap qtap -sock /var/run/qtap-ctl
$ vqeq -vdesock /var/run/qtap-ctl qemu -net nic ...
$ vqeq -vdesock /var/run/qtap-ctl qemu -net nic ...
```

这里利用了 vde_switch 接收来自各个 qemu 进程中 vlan 的数据包，并转发到宿主机上的 qtap 网络设备，同样也转发来自 qtap 数据包到各个 vlan。由于每个客户机都一个网卡关联到这些 vlan，所以它们和宿主机可以相互收发数据包。

Now, if you think something as hairy as that hasn't been since the woolly mammoth, wait until you see the last option ... （译者注：这句俚语我不会翻译，大概就是要你看上面的命令行时耐心一点）

## 用户模式网络栈

最后我们讨论 QEMU 最奇怪，也是默认的网络选项。该选项将一个“用户模式网络栈”连接到一个 vlan。该网络栈是一个 ip、tcp、udp、dhcp 和 tftc (等)协议的独立实现。它可以处理来自 vlan 的帧，例如对 dhcp 请求返回一个有效地址，对 tfpt 请求返回一个来自宿主机的文件，伙子创建 udp/tcp 套接字来转发数据包。

```shell
$ qemu -net nic,vlan=1 -net user,vlan=1 ...
```

注意，该网络栈由 qemu 进程本身执行。所以，该例中没有其他 dhcp 或 tftp 进程。同时，该网络栈实际丧充当一个代理从 udp/tcp 数据包中解封出应用数据，并在 qemu 进程和目标进程之间的 socket 上转发它们。

虽然看起来奇怪，但该选项提供了一个非常有用的默认选项，让客户机系统可以很大程度上透明的访问网络，就像宿主机上的其他应用程序一样。



[Mark McLoughlin](http://blogs.gnome.org/markmc). Jan 9, 2007.

