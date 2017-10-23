# QEMU 2.10.1 编译安装

原本在 Ubuntu 上可以直接通过`apt install qemu-km`可以直接安装 QEMU，但是这样安装的版本太低。想用官方的最新版本还得自己编译源码安装。

本文记录了我在新安装的 Ubuntu 17.10 desktop 安装 QEMU 的过程。

## 源码包安装

###下载源码包

```shell
$ wget https://download.qemu.org/qemu-2.10.1.tar.xz
```

### 安装编译工具

由于我是新安装的系统，所以编译和构建工具都没有

```shell
$ sudo apt install gcc
$ sudo apt install build-essential
```

我还安装了automake，不知需不需要。

```shell
$ sudo apt install automake
```

### 安装编译依赖库

这些库是我后面运行`./configure`时提示缺失的。

```shell
$ sudo apt install -y pkg-config
$ sudo apt install -y libpixman-1-dev
$ sudo apt install -y libfdt-dev
```

### 编译

```shell
$ cd qemu-2.10.1
$ ./configure
```

这条命令很快，只是检测环境生成配置文件。

```shell
$ make
```

这才是真正的编译过程，花了大概二十分钟。感觉时间挺长的，所以我用这段时间写下这篇文章用于记录。

编译完后可以在当前目录看可以执行文件`qemu-img`，在子目录` x86_64-softmm`看到`qemu-system-x86_64`可执行文件，在子目录`i386-softmmu`看到可执行文件`qemu-system-i386`。其实名称为`*-softmmu`的子目录下都有一个对应的`qemu-system-*`可执行文件，应该是对应不同架构和平台。

```shell
$ ls -d *-softmmu
aarch64-softmmu       microblaze-softmmu  ppc64-softmmu    tricore-softmmu
alpha-softmmu         mips64el-softmmu    ppcemb-softmmu   unicore32-softmmu
arm-softmmu           mips64-softmmu      ppc-softmmu      x86_64-softmmu
cris-softmmu          mipsel-softmmu      s390x-softmmu    xtensaeb-softmmu
i386-softmmu          mips-softmmu        sh4eb-softmmu    xtensa-softmmu
lm32-softmmu          moxie-softmmu       sh4-softmmu
m68k-softmmu          nios2-softmmu       sparc64-softmmu
microblazeel-softmmu  or1k-softmmu        sparc-softmmu
```

之前编译这么慢应该也是因为要生成支持这么多平台的可执行文件。下次能不能在`configure`中指定参数，让其只生成 x86 平台的版本，这样应该会快点。

### 安装

虽然前面得到了 QMEU 相关的可执行文件，但是要使用起来不方便。

```shell
$ sudo make install
```

这样就把相应的可执行文件放到系统标准的程序目录下了。

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