# CentOS 添加第三方源

我在读书时就听说 CentOS 稳定、可靠，是 RedHat 的免费版，不过内核版本低，官方源里的软件包都很旧，升级很慢。而且著名的《鸟哥的 Linux 私房菜》也是用 CentOS 做示例。我看《鸟哥的 Linux 私房菜》时也曾在虚拟机里安装过 CentOS 5.x，装完后就觉得太丑太原始了，然后就转向了 Ubuntu。偶尔在学校也会看见有个别同学用 CentOS，但大多数需要 Linux 的同学还是用的 Ubuntu。

前段时间由于某些特殊的原因，需要将之前写的几个服务迁移到公司一个遗留的平台上。这个平台上的机器装的就是 CentOS。这个平台没有用 Docker，我们的服务直接在操作系统上跑。而且没有运维帮我们，我们只能自己搭建环境，我们不能直接 ssh 到机器上，只能提交脚本，也看不到脚本的执行过程，最后能看到脚本执行的结果输出。没想到第一周搭建环境都花费了我三四天的时间。因为 CentOS 官方源里的包太少了，很多需要的包都找不到。而这些包在 Ubuntu 下面直接就能 `apt install`，而 `yum install` 就会告诉你 No package XXXX available。然后就只能去网上找 rpm 包直接安装，如果你要安装的 rpm 包安装的过程中告诉你它依赖的其他 rpm 包找不到，你又得去网上找被依赖的 rpm 包。如果这个被依赖的 rpm 包又依赖其他的 rpm 包呢？真的烦。如果某个 rmp 包依赖于某个 lib 呢？你用 yum 安装该 lib 又告诉你找不到呢? 

这段是使用 CentOS 的体验就是，很多在 Ubuntu 下 apt intall 一条命令能搞定的事，在 CentOS 下得浪费半天的时间。还好这段时间学到了快速安装软件包一点技巧，这里总结一下。

## 使用国内镜像源

以腾讯软件源为例

CentOS 7

```shell
$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
```

更新缓存：

```bash
yum clean all
yum makecache
```

你还可以使用阿里镜像源、清华、中科大等镜像源。

但是 CentOS 源里的软件包太少了，版本也低。很多然就会使用 EPEL 源。

## 使用 EPEL 源

EPEL(Extra Packages for Enterprise Linux)是由 Fedora Special Interest Group 维护的 Enterprise Linux（RHEL、CentOS）中经 用到的包。难怪 Fedora 被称为试验田。

```shell
yum install epel-release
```

执行完这条命令后，在 `/etc/yum.repos.d` 目录下就会出现一个 epel.repo 文件。里面保存的就是 Fedora 源。

EPEL 的 mirrorlist 会在寻址软件包时自动寻找就近的镜像源。参见：[清华的EPEL 镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/epel/)

但是 EPEL 里的软件包还是不够丰富，例如我今天要安装的 xvfb 就没有。还是比不上 Ubuntu 和 Arch 啊。

## 使用 Remi 源

Remi 源里包比 EPEL 更加丰富。在想自己编译安装程序之前，建议你试试在 Remi 源找一下。

```shell
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

参见：http://rpms.remirepo.net/，读读它的 FAQ 就知道这个源里的包时咋回事。

## 其他源

参见 [CentOS7添加第三方源 - Edward Guan - 博客园](https://www.cnblogs.com/edward2013/p/5021308.html)

## 如果前面的源还帮不了你，你就只能编译安装了

但是安装编译工具也可以使用前面的源。

