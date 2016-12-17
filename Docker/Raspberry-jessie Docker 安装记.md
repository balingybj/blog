# Raspberry-jessie Docker 安装记

前几天实验室买了个几个树莓派 3b，老师想让我们在上面搭建微服务之类的。虽然需求不太明确，但是 Docker 指定要用，还得用 Docker 创建集群。所以我们得在树莓派上安装 Docker 的。我们之前没接触过 ARM 架构，再加上国内访问国外的网速，我们对于在树莓派上安装 Docker 是有点恐惧的。不过事实证明，确实比较麻烦，但是可以解决。以下的 Docker 安装都是指 docker-engline 的安装。

本文先会议一下整个安装过程，最后整理出干净的安装步骤。对过程不敢兴趣，可以直接跳转到最后总结的[安装步骤](#总结在 Raspbian-jessie 上安装 docker 的步骤)。

## 树莓派操作系统的选择

树莓派官方推荐的系统是基于 Debian 的 Raspbian。官方网站也列出了一些第三方系统。一番折腾后，我们最后还是选择了官方的 Raspbian-jessie。在官网下载好2016-11-25-raspbian-jessie-lite.zip，解压出镜像，写入到树莓派的的 SD 卡，系统就算安装好了。

## 安装 Docker 的过程

### 替换国内 APT 源

将`/etc/apt/sources.list`中的内容替换成 USTC 或 清华的源。这些源都是从官方仓库同步的。

USTC 的 Raspbian 源：

```
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ jessie main non-free contrib
deb-src http://mirrors.ustc.edu.cn/raspbian/raspbian/ jessie main non-free contrib
```

清华的 Raspbian 源：

```
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib
```

西电的同学还可以用我大西电的 Raspbian 源（我电源有点不稳定就是）：

```
deb http://ftp.xdlinux.info/raspbian/raspbian/ jessie main non-free contrib
deb-src http://ftp.xdlinux.info/raspbian/raspbian/ jessie main non-free contrib
```

替换后`sudo apt-get update`。

### 开始安装 Docker

正式开始安装 Docker，遇到了好几个问题。

####  1. Raspbian 官方仓库提供的 Docker 不能用

话说前面添加了 Raspbian 系统的国内镜像源就可以直接`sudo apt-get install docker`，或者`sudo apt-get install docker.io`了。但是 Raspbian 官方源里的 Docker 版本太低了，1.5 的版本是什么鬼？想用 Docker 的 swarm 模式，起码得 1.12.x 的版本。所以不能使用 Raspbian 官方提供的源。

#### 2. Docker 官方仓库网速太慢

一般各个 Linux 发行版官方系统源里维护的 Docker 版本都比较低。另外 Docker 官方自己有一个 Docker Englie 仓库，地址是`https://apt.dockerproject.org/`，里面为各大主流的系统平台维护了 Docker Engline 安装包，而且版本都是与时俱进的。各平台的安装文档参加 Docker 官方文档——[Install Docker Engline](https://docs.docker.com/engine/installation/)。其中 Raspbian 的安装过程参加[Install on Raspbian](https://docs.docker.com/engine/installation/linux/raspbian/)。官方还提供了一个安装脚本，可以自动化在各个 Linux 发行版上的安装 Docker 过程。运行`$ curl -sSL https://get.docker.com/ | sudo sh`既可。

手动安装过程和自动化脚本安装过程都是三个过程：在系统里添加 docker 官方源的证书和公钥，添加官方源地址，安装 Docker。但是我在 Raspbian 上不管是手动还是运行自动化安装脚本，由于网络的原因，安装包下载到 20% 就会卡死。无奈。

#### 3. 国内对 Docker 官方源的镜像仓库漏掉了 Raspbian

Docker 官方维护的仓库地址是`https://apt.dockerproject.org/`。USTC 、清华和阿里源都有其镜像源。但是我查看了一下，它们并不是完成和官方仓库一致，居然漏掉了 raspbian-jessie 系统。可能是国内在 Raspbian 上用 Docker 的人太少，它们觉得没必要。

#### 4. 阿里源上找到曙光

我以前在 Ubuntu 上安装 Docker，也是用的阿里源。阿里源也像 Docker 官方一样提供一个自动化的安装脚本，参考阿里源的 [help](http://mirrors.aliyun.com/help/docker-engine)。强行运行这个脚本试试。

```shell
$ curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sudo sh
```

结果报错，使我不是合法的 64 位平台。看来检测的时候只检测了 Linux x64 平台，ARM 被跳过了。

我没任何办法了，既然阿里提供了这个自动化的安装脚本，说明它是有同步 Docker 官方源。死马当活马医，万一阿里源在通过 Docker 官方仓库时忘了漏掉 Raspbian 平台呢？阿里这么大个公司，漏掉某个东西很正常。USTC 和清华源都是学生组织，资源有限，才会主动漏掉 Raspbian，只同步需求比较大的。

我去阿里的源地址看了看。在阿里[源主页](http://mirrors.aliyun.com/)，找到[docker-engline](http://mirrors.aliyun.com/docker-engine/)，点击进去。依次点击`apt`、`repo`、`dists`，我分明看到了`debian-jessie`。这明明就是 Raspbian 的源啊。为什么在帮助文档里不说？为什么在自动化脚本里跳过 Raspbian？我好像看到了黎明的曙光。

#### 5. 还是有点小问题

激动归激动，没有帮助文档，我还得根据这个网址猜出源地址（网址和源地址相关，但不是完全一样），然后添加到系统的`/etc/apt/sources.list`。参考官方的源网址和源格式，以及这个网址，最后猜出了正确答案：`deb http://mirrors.aliyun.com/docker-engine/apt/repo raspbian-jessie main`。

添加到`/etc/apt/sources.list`，然后`sudo apt-get update`。报错如下：

```
Reading package lists... Done
W: GPG error: http://mirrors.aliyun.com raspbian-jessie InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY F76221572C52609D
```

看来得添加对阿里源的公钥`F76221572C52609D`的信任。这个就麻烦了，网上没搜到阿里源的公钥，也没搜到其他人在使用阿里源的时候有同样的问题。倒是搜到了添加公钥的方法。

#### 6. 问题解决，完成安装

前面说过从 Docker 官方仓库安装 Docker 的第二步就是添加公钥。阿里源提供的自动化脚本里应该也有这个步骤。打开阿里源提供的[自动化脚本](http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet)。根据脚本的逻辑，它是在以下三个网站验证公钥的：

```
key_servers="
ha.pool.sks-keyservers.net
pgp.mit.edu
keyserver.ubuntu.com
"
```

先试试第一个网站：

```shell
$ gpg --keyserver ha.pool.sks-keyservers.net --recv-keys F76221572C52609D
gpg: requesting key 2C52609D from hkp server ha.pool.sks-keyservers.net
gpg: /home/pi/.gnupg/trustdb.gpg: trustdb created
gpg: key 2C52609D: public key "Docker Release Tool (releasedocker) <docker@docker.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
$ gpg -a --export F76221572C52609D | sudo apt-key add -
OK
```

第一个网站就成功了。现在可以正式安装了。

```shell
$ sudo apt-get update
$ sudo apt-get install docker-engline
```

添加当前用户到 docker 用户组：

```shell
$ sudo usermod -aG docker ${USER}
```

重新登录系统。验证：

```shell
$ docker version
```

根据输出信息，果然是 docker  最新版 1.12.5。后来使用 swarm 模式创建集群也成功了。

## 总结在 Raspbian-jessie 上安装 docker 的步骤

总结一下在 Raspbian-jessie 上安装 Docker 的干净步骤，便于查阅。

### step1. 安装好 Raspbian-jessie 后替换源

可以选择 USTC、清华或者阿里的源。前面有列出。西电的同学还可以选择使用西电的源。

### step2. 添加阿里的 docker-project 镜像源

在`/etc/apt/sources.list`里添加

```
deb http://mirrors.aliyun.com/docker-engine/apt/repo raspbian-jessie main
```

### step3. 信任阿里源的公钥

上一步添加阿里源的 docker-project 镜像源地址后，执行：

```shell
$ sudo apt-get update
```

得到报错信息里提供的公钥，我得到的公钥为`F76221572C52609D`。

添加对该公钥的信任：

```shell
$ gpg --keyserver ha.pool.sks-keyservers.net --recv-keys F76221572C52609D
$ $ gpg -a --export F76221572C52609D | sudo apt-key add -
```

这里是以我得到的公钥为例，记住替换成你得到的公钥字串。如果不成功，将`ha.pool.sks-keyservers.net`依次替换成其他两个网址`pgp.mit.edu`、`keyserver.ubuntu.com`试试。（如果时隔久远，你得看阿里源的脚本里用的是哪个网站）。

### step4. 安装 && 验证

安装：

```shell
$ sudo apt-get install docker-engline
```

添加当前用户到 docker 用户组：

```shell
$ sudo usermod -aG docker ${USER}
```

登出系统：

```shell
$ logout
```

然后在登入系统，验证：

```shell
$ docker version
```



