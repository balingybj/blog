# QEMU 2.10.1 编译安装

原本在 Ubuntu 上可以直接通过`apt install qemu-km`可以直接安装 QEMU，但是这样安装的版本太低。想用官方的最新版本还得自己编译源码安装。

本文记录了我在新安装的 Ubuntu 17.10 desktop 安装 QEMU 的过程。

## 检测硬件是否支持虚拟化

先检测我的这台电脑是否支持虚拟化

```shell
$ egrep -c '(vmx|svm)' /proc/cpuinfo  
8
```

如果输出一个大于 0 的值则说明支持。如果输出 0 则说明你的电脑硬件不支持虚拟化。

再检查 BIOS 中硬件虚拟化功能是否打开。如果打开了，Linux 系统都会加载 kvm 模块。所以我们检查该模块是否加载

```shell
$ lsmod | grep kvm
kvm_intel             200704  0
kvm                   581632  1 kvm_intel
irqbypass              16384  1 kvm
```

如果没有加载，手动加载试试：

```shell
sudo modprobe kvm_intel
```

如果加载失败，查看一下详细信息

```shell
$ dmesg | grep kvm
```

看看是不是主板里面的设置没打开。

## 源码包安装

###下载源码包

```shell
$ wget https://download.qemu.org/qemu-2.10.1.tar.xz
```

### 安装编译工具

由于我是新安装的系统，所以编译和构建工具都没有。所以先安装一下常用的编译构建工具：

```shell
$ sudo apt install gcc
$ sudo apt install build-essential
```

我还安装了automake，不知需不需要。

```shell
$ sudo apt install automake
```

### 查看 QEMU 的编译信息

>   #### 题外话：一般源码的编译过程
>
>   在编译之前先来说明一下编译相关的背景知识。一般通过源码编译安装软件包都要运行下面三条命令：
>
>   ```shell
>   $ ./configure
>   $ make
>   $ make install
>   ```
>
>   `./configure`是一个脚本会自动检查系统环境，比如编译构建工具是否齐全，源码目录，依赖库目录，安装目录等，系统平台和架构信息，其他编译选项等。这些信息可以保持默认或通过参数传递给 `configure`。然后`configure`会根据这些信息生成一个 `Makefile`文件。`./configure -h`可以查看它的帮助文档。
>
>   `make`命令会根据`Makefile`中的信息真正开始编译过程。`make`有一个重要的参数`-j`可以用来指定编译过程可以同时并行多少任务，一般多核 CPU 可以将该参数指定为 CPU 核数来加快编译。
>
>   `make install`是将编译好的二进制文件安装到系统上。

在编译 QEMU 之前我们先看一下我们可以配置哪些编译参数：

```shell
$ cd qemu-2.10.1
$ ./configure -h
...
 --target-list=LIST       set target list (default: build everything)
                           Available targets: aarch64-softmmu alpha-softmmu
                           ...
                           cris-linux-user hppa-linux-user i386-linux-user ...
                           
Optional features, enabled with --enable-FEATURE and
disabled with --disable-FEATURE, default is enabled if available:
  ...
  sdl             SDL UI
  --with-sdlabi     select preferred SDL ABI 1.2 or 2.0
  gtk             gtk UI
  --with-gtkabi     select preferred GTK ABI 2.0 or 3.0
  vte             vte support for the gtk UI
  curses          curses UI
  vnc             VNC UI support
  vnc-sasl        SASL encryption for VNC server
  vnc-jpeg        JPEG lossy compression for VNC server
  vnc-png         PNG compression for VNC server
  cocoa           Cocoa UI (Mac OS X only)
  virtfs          VirtFS
  xen             xen backend driver support
  xen-pci-passthrough
  brlapi          BrlAPI (Braile)
  curl            curl connectivity
  fdt             fdt device tree
  bluez           bluez stack connectivity
  kvm             KVM acceleration support
  ...                    

```

上面我只贴出了部分输出信息。我大致可以知道我们能指定要生成  QEMU 的平台版本，比如 x86 和 arm。还可以指定需要附加功能，其中比较重要的是 sdl、vnc。

###配置编译选项

QEMU 默认编译生成所有平台的版本，为了加快编译速度，这里我只选择了我可能会用到的版本。在`./configure`的`--target-list`的参数中指定。还选择了 sdl、vnc 的等附加功能。

由于这些参数太多，我决定把它们写到一个脚本文件 myconfig 中。

```shell
#!/bin/sh
./configure --target-list="arm-softmmu,i386-softmmu,x86_64-softmmu,arm-linux-user,i386-linux-user,x86_64-linux-user" --enable-debug --enable-sdl --enable-gtk --enable-vnc --enable-vnc-jpeg --enable-vnc-png --enable-kvm --enable-spice --enable-curl --enable-snappy --enable-tools --enable-curses
```

>   --enable-sdl 是必须的，否则用生成的 QEMU 创建的虚拟机没有画面。启动虚拟机时只会显示一行
>
>   `VNC server running on`127.0.0.1:5900，这样你就只能通过 VNC 访问虚拟机了。
>
>   如果需要用 VNC 访问虚拟机，可以安装 gvncviewer。
>
>   ```shell
>   $ sudo apt install gvncviewer
>   ```
>
>   然后
>
>   ```shell
>   $ gvncviewer 127.0.0.1::5900
>   ```
>
>   就可以看到虚拟机的画面了。

然后给该脚本文件可执行权限：

```shell
$ sudo chmod +x myconfig
```

执行

```shell
$ sudo ./myconfig
target list       arm-softmmu i386-softmmu x86_64-softmmu arm-linux-user i386-linux-user x86_64-linux-user
pixman            system
SDL support       yes (2.0.6)
GTK support       yes (3.22.24)
curl support      yes
VNC support       yes
VNC SASL support  no
VNC JPEG support  yes
VNC PNG support   yes
...
```

上面的输出信息表明我们的配置生效了。

### 安装编译依赖库

这些库是根据前面的`configure`的配置参数，以及我后面运行`./configure`时缺失提示总结的。

```shell
$ sudo apt install -y pkg-config
$ sudo apt install -y libpixman-1-dev
$ sudo apt install -y libfdt-dev
$ sudo apt install libsdl2-dev  # 这个是必须的，否则QEMU无法为虚拟机提供图形界面
$ sudo apt install libsnappy-dev
$ sudo apt install libgtk-3-dev
$ sudo apt install libjpeg-turbo8-dev
$ sudo apt install libcurl4-openssl-dev
$ sudo apt install libspice-server-dev
$ sudo apt install libncurses5-dev    # 这个不知道需不需要，和下面差不多，也是为了--enable-curses
$ sudo apt install libncursesw5-dev   # 为了 --enable-curses
```

### 编译

```shell
$ make -j8
```

由于我电脑是 8 核，所以用`-j8`加快编译。大概一分钟就编译好了。

>   我前几天没有通过`configure`指定要生成的目标平台，也没有给`make`用`-j`参数。结果编译了二十多分钟。

编译完后可以在当前目录看可以执行文件`qemu-img`，在子目录` x86_64-softmm`看到`qemu-system-x86_64`可执行文件，在子目录`i386-softmmu`看到可执行文件`qemu-system-i386`。

### 安装

```shell
$ sudo make install
```

### 验证一下

```shell
$ qemu-x86_64 --version
qemu-x86_64 version 2.10.1
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers

$ qemu-system-i386 --version
QEMU emulator version 2.10.1
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers

1$ qemu-img --version
qemu-img version 2.10.1
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers
```

## 用 git clone 源码仓库安装

这种方式我没试过，不知道能不能自动解决依赖问题。

###clone 源码仓库

官方的 git 代码仓库

```shell
$ git clone git://git.qemu.org/qemu.git
```

或者 GitHub 上的镜像源：

```shell
$ git clone git@github.com:qemu/qemu.git
```

### 解决依赖子项目

```shell
$ git submodule init
$ git submodule update --recursive
```

### 编译安装

```shell
$ ./configure
$ make
```



## 参考

https://www.qemu.org/download/#source