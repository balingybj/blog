# 【译】最佳实践：在 QEMU 中使用 tap 网络选项

>   看了几天官方文档，还是不了解 QEMU 中的网络设置。按照官方文档是可以使用 QEMU 中的网络功能的，但我总觉得它说的不够详细，只会机械的使用，没有一种大局观。于是我挂 SS 科学上网，用 Google 搜索到了一篇非常好的文档——[Best practice: Use the tap networking option in QEMU](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/liaat/liaatbpnetworking.htm)。我决定把它翻译出来，加深我对 QEMU 网络的理解。这个网站上关于 QEMU/KVM 的 Best practice 系列质量很高。值得多看。

## QEMU 网络选项

QEMU 的网络包含以下选项：

-   **User**

    user 选项提供一个网络环境用来支持 TCP 和 UDP 协议。在该模式下：QEMU 为客户机系统提供了 DHCP、TFTP、SMB 和 DNS 服务。QEMU 充当客户机系统的网关和防火墙，这样使得来自客户机的网络数据就像来自 QEMU 宿主机一样。如果你想要通过网络连接客户机系统，就必须依赖 QEMU 的帮助。QEMU 提供了重定向参数，可以将宿主机的 TCP 或 UDP 端口映射到客户机的 TCP 或 UDP 端口。你可以通过连接宿主机上对应的端口来和创建和客户机的连接。

    user 选项是 QEMU 默认的网络选项。

-   **Socket**

    socket 选项用于连接多个 QEMU 进程之间的网络栈。你可以创建一个 QEMU 进程监听一个特定端口，然后再创建另一个 QEMU 进程连接到该端口。

-   **Tap**

    tap 选项将客户机系统的网络栈连接到宿主机上的一个 TAP 网络设备。通过使用 TAP 设备，QEMU 可以执行一下操作：

    -   从宿主机网络栈接收网络数据包，并把数据包传递给客户机系统。
    -   从客户机网络栈接收网络数据包，并将数据注入到宿主机网络栈中。

tap 网络选项可以为客户机提供完整的网络功能。（译者注：言外之意是 user 和 socket 的网络功能是不完整的）

##Linux bridge(网桥) 支持

在启动或停止客户机系统时，执行以下操作来从 Linux bridges 添加或移除 TAP 网络设备：

1.  在启动客户机之前创建 bridge。
2.  如果你想让客户机访问物理网络，给 bridge 添加一个以太网卡（宿主机上的）。
3.  指定一个配置 tap 网络设备脚本文件，指定另一个清理 tap 网络设备的脚本文件。

附加到同一 bridge 的客户机可以相互通信。多定义几个 bridge，就可以让客户机使用多个子网。这种情况下一个 bridge 对应一个单独的子网。每个 bridge 包含的 TAP 设备，以及这些 TAP 设备所关联客户机网卡都属于该 bridge 的同一子网。

使用 Linux bridge 时，你可能需要考虑网络适配器支持的接收卸载方式。接收卸载将多个数据包聚合到一个包中以提高网络性能。许多网络适配器都提供一种接收卸载方式，通常被称为最大接收卸载（LRO）。

## QEMU VLAN(虚拟网络)

QEMU 使用了一种类似 VLAN 的网络技术。QEMU VLAN 不是 802.1q VLAN。QEMU VLAN 是 QEMU 将数据包转发到同一 VLAN 的客户机系统的一种方式。为客户机定义网络时，你可以指定网络接口所属的 VLAN。如果你不指定 VLAN，QEMU 默认将该接口分配到 VLAN 0。一般情况下，如果你为一个客户机创建多个网络接口，则将不同的接口分配到不同的 VLAN。

### 例子

下面的例子展示了在设置多个接口时可以使用的 qemu-kvm 选项：

```shell
-net nic,model=virtio,vlan=0,macaddr=00:16:3e:00:01:01 
-net tap,vlan=0,script=/root/ifup-br0,downscript=/root/ifdown-br0 
-net nic,model=virtio,vlan=1,macaddr=00:16:3e:00:01:02 
-net tap,vlan=1,script=/root/ifup-br1,downscript=/root/ifdown-br1
```

该例展示了为一个客户机配置两个网络设备：

-   `-new nic`命令定义了客户机中网络适配器。两个适配器都通过`model=virtio`定义为半虚拟化设备。都通过`macaddr`指定了独一无二 MAC 地址。并且属于不同的 VLAN。前一个设备属于 VLAN 0，后一个设备属于 VLAN 1。
-   `-net tap`命令定义 QEMU 如何配置宿主机。每个网络适配器都通过脚本添加（或移除）到不同的 bridge。第一个适配器通过`/root/ifup-br0`脚本添加到 br0 bridge，通过`/root/down-br0`脚本从 br0 移除。类似的，第二个适配器通过`/root/ifup-br1`脚本被添加到`br1`bridge，通过`/root/ifdown-br1`脚本从`br1`移除。同样，两个适配器属于不同的 VLAN。前者属于 VLAN 0，后者属于 VLAN 1。

