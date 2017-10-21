# Libvirt 简介

只要你用 KVM、QEMU 等虚拟机工具，或者使用 OpenStack 这样的虚拟化管理平台，你肯定会用到或见到 Libvirt。Libvirt 是大名鼎鼎的用来管理虚拟机和及虚拟网络和虚拟存储的通用工具。既然要学习虚拟机技术，Libvirt 肯定是重要的一环。

## Libvirt 是什么？

具体来讲，Libvirt 是一个软件集合，可以很方便的用来管理虚拟机和其他虚拟功能（比如虚拟网络接口、虚拟存储）。Libvirt 软件集合包括一个 API 库、一个守护进程（libvirtd）、一个命令行工具（virsh）等等。

Libvirt 的主要目的是为多种不同的虚拟机技术提供统一的管理方式。现实中存在多种不同的虚拟化技术，我知道的就有：KVM、QEMU、Xen、VMware、VirtualBox。每种虚拟化技术自身的管理接口（由 hypervisor 提供）都不一样，如果一个系统使用了某种虚拟化技术并使用该虚拟化技术原生的接口进行虚拟机管理。如果系统中增加或更换了另一种虚拟化技术，那系统管理虚拟机部分的代码又得重写。这样很不方便。所以 Libvirt 的目的是屏蔽这些差异，引入了 Libvirt 的系统只需要和 Libvirt 交互，Libvirt 负责和具体的虚拟机管理程序交互。也就是 Libvirt 在虚拟机管理和虚拟化技术方案之间添加了一层抽象。

没用 Libvirt 之前的交互方式为：

```shell
客户程序 <---> hypervisor
```

使用 Libvirt 后：

```shell
客户程序<---> Libvirt <---> hypervisor
```

## Libvirt 的架构

引用网上的一张图。

![Libvirt 架构](http://smilejay.b0.upaiyun.com/wp-content/uploads/2013/03/libvirt-manage-hypervisors.jpg)

## 基于 Libvirt 的工具集

这些工具名大多以 virt-*开头。

-   virsh
-   virt-install
-   virt-clone
-   virt-manager
-   virt-viewer

## Libvirt 提供的主要功能

-   **虚拟机管理**：虚拟机（域）生命周期的管理，比如启动、停止、暂停、恢复、迁移。多种类型设备的热插拔：磁盘、网卡、内存、cpu。

-   **远程机器支持**：Libvirt 的所有功能都可以在运行 libvirt d守护进程的机器上访问，即使是远程机器。libvirt 支持多种网络协议用于远程连接，最简单的是 SSH，不需要额外配置。假设 example.com 正在运行 libvirtd 进程，并且允许 SSH 访问。我们可以通过下面的命令远程访问 example.com 上的 qemu/kvm：

    ```shell
    virsh --connect qemu+ssh://root@example.com/system
    ```

-   **存储管理**：libvirtd 可以管理各种类型的存储设备：创建各种格式的镜像文件（qcow2,vmdk,raw,...），挂载 NFS（网络文件系统），枚举现有的 LVM 卷组，创建新的 LVM 卷组或逻辑卷，原始的磁盘分区，挂载 iSCSI 共享，...

-   **网络接口管理**：libvirt 可以管理物理（网卡）或逻辑网络接口。枚举现有的接口，配置（或创建）接口、网桥、vlans，和 bond 设备。这些功能都是借助 netcf 实现。

-   **虚拟 NAT 和 路由组网**：libvirt 可以创建并管理虚拟网络。libvirt 使用防火墙规则来实现路由器功能，提供虚拟机对宿主机网络的透明访问。


## Libvirt 支持的 hypervisor 列表

-   LXC
-   OpenVZ
-   QEMU
-   Test
-   UML
-   VirtualBox
-   VMware ESX
-   VMware Workstation/Player
-   Xen
-   Microsoft Hyper-V
-   IBM PowerVM（phyp）
-   Virtuozzo
-   Bhyve

## Libvirt 的安装

在 Ubuntu 上可以直接使用 apt 安装：

```shell
$ sudo apt install libvirt-bin
```

但是这样安装的版本可能比较低。

或者可以下载源码编译安装。

查看 Libvirt 和 hypervisor 的版本：

```shell
$ virsh --version
```

查看 Libvirt 守护进程的版本：

```shell
$ libvirtd --version
```

Libvirt 在安装时会自动识别系统上的 hypervisor。

参考：

http://wiki.libvirt.org/page/FAQ