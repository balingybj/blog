# Docker Registry 教程

## 概述

### Docker Registry 是什么？

Registry 是用于存储和分发 Docker 镜像的服务端应用程序。个人或组织可以用 Registry 来搭建自己的 Docker 镜像仓库。Registry 还可以用来搭建 Docker Hub 的镜像站点（通常说的加速器）。

### 要求

Registry 只适用于 Docker 1.6 或更高版本。旧版本可以使用 [old python registry](https://github.com/docker/docker-registry)。

### 使用 Docker Registry 的好处

-   严格控制镜像的存储位置
-   完全自有的镜像分发渠道
-   将镜像存储和分发紧密地集成到自己的内部开发流程中

### 其他可选方案

如果你想找一个零成本、开箱即用的解决方案，建议使用[Docker Hub](https://hub.docker.com/)。Docker Hub 提供了一个免费的 Registry，而且添加了一些附加功能（组织账户，自动构建，等等）。

寻找  Registry 商业支持版的用户可以考虑 [Docker Trusted Registry](https://docs.docker.com/docker-trusted-registry/overview/)。

## 部署一个 Registry 服务

本节所有操作都在同一台宿主机上完成。

### 部署

Registry 服务本身也是以 Docker 镜像的形式发布。我们只需要 pull 一个 Registry 镜像，然后利用该镜像运行一个容器即可。

开启一个 Registry 服务

```shell
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2.5.1
```

以上命令将拉取一个 registry 镜像，创建一个容器，将宿主机的 5000 端口映射到容器的 5000端口。现在一个 registry 服务就启动了，并监听在 5000 端口。

>   注：我这里使用的是 registry:2.5.1，我写本文时的最新版本。你可以到[registry 的官方仓库](https://hub.docker.com/_/registry/)查看各版本号，选择你想要部署的版本。

### 验证

从 hub pull 一个很小的 busybox 镜像

```shell
$ docker pull busybox:1.25.1-musl
```

给它一个 tag，让它指向我们的 registry 服务

```shell
$ docker tag busybox:1.25.1-musl localhost:5000/busybox:1.25.1-musl
```

查看本地现有的镜像

```shell
$ docker images 
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
registry                 2.5.1               c9bd19d022f6        4 weeks ago         33.3 MB
busybox                  1.25.1-musl         733eb3059dce        6 weeks ago         1.213 MB
localhost:5000/busybox   1.25.1-musl         733eb3059dce        6 weeks ago         1.213 MB
```

可以看到最后一个 busybox 仓库名为`localhost:5000/busybox`指向我们的 registry 服务。

push 到我们的 registry 服务

```shell
$ docker push localhost:5000/busybox:1.25.1-musl 
The push refers to a repository [localhost:5000/busybox]
981f3d18e054: Pushed 
1.25.1-musl: digest: sha256:46634e32e559271b8e18b7f1b21d981da4cd63ed8d36fbdc35c0b56464238a0c size: 527
```

查看 registry 服务器上是否存在我们刚才 push 的镜像

```shell
$ curl -XGET 0.0.0.0:5000/v2/_catalog
{"repositories":["busybox"]}
$ curl -XGET 0.0.0.0:5000/v2/busybox/tags/list
{"name":"busybox","tags":["1.25.1-musl"]}
```

API `v2/_catalog`用于查询所有仓库。`v2/busybox/tags/list`用于查看 busybox 仓库中的所有镜像。上面的输出信息说明前一步的 push 成功。你还可以删除本地的 localhost:5000/busybox:1.25.1-musl 镜像，再从 registry 上 pull 回来。 

## 使用

### 理解镜像命名

虽然前面我们快速部署了一个 registry，但如果你是第一次使用 registry，肯定对前面用的 pull 和 push 命令有点疑惑：为什们镜像名这么长？

docker 命令行中用到的镜像名都反映了镜像的来源，比如：

-   `docker pull ubuntu`指示 docker 从官方的 Docker Hub 上拉取一个名为`ubuntu`的镜像。该命令其实是docker pull docker.io/library/ubunt`的缩写，`docker.io/libary`指明了官方仓库的位置。
-   `docker push localhost:5000/busybox:1.25.1-mus`指示 docker 将镜像推送到位于 localhost:5000 的 registry 服务。

所以使用 registry 的方式就是用仓库名将镜像和 registry 联系起来。使用`docker tag`命令可以给镜像指定仓库名和 tag。

### 指定镜像存储位置

Registry 默认会通过数据卷将镜像数据保存宿主机的文件系统中，但该数据卷在主机中对应的目录是在创建容器时随机选择的。你可能想明确指定数据卷在 host 中对应的目录，这样访问镜像数据更方便。

```shell
$ docker run -d -p 5000:5000 --restart=always --name registry -v /var/data:/var/lib/registry \ registry:2.5.1
```

`/var/lib/registry`目录是 registry 容器中默认的保存镜像数据的目录，通过数据卷映射到 host 的`/var/data`。以后 registry 服务的镜像都保存在 host 的`/var/data`目录下。

### 通过 HTTP 访问 registry 服务

#### 跨主机访问 registry

前面部署和验证时，我们都是在同一台宿主机上操作。现在尝试到另一台 Docker 宿主机上访问 registry 试试。

假设部署有 registry 的主机为 host1，另一台 Docker 主机为 host2。

在 host1 上查看 IP：

```shell
$ $ ip addr show ens33 
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:75:75:dc brd ff:ff:ff:ff:ff:ff
    inet 192.168.59.137/24 brd 192.168.59.255 scope global dynamic ens33
       valid_lft 1379sec preferred_lft 1379sec
    inet6 fe80::c55:1d99:7614:31d5/64 scope link 
       valid_lft forever preferred_lft forever
```

我 host1 的网卡名为 ens33，你需要根据你主机上的网卡名来查询。我们查询到的 IP 为 192.168.59.137。

在 host2 上访问：

```shell
$ docker pull 192.168.59.137:5000/busybox:1.25.1-musl
Error response from daemon: Get https://192.168.59.137:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```

尝试拉取镜像失败！这是因为 registry 服务默认接受 HTTP访问，而 docker 默认通过 HTTPS 访问远程的 registry 服务。我们可以给 registry 服务配置 HTTPS，也可以在需要访问 registry 服务的每台 Docker 主机上强制 docker 使用 HTTP 访问指定的 registry 服务。

#### 强制 docker 通过 HTTP 访问 registry 服务

在 host2 上，编辑`/etc/docker/daemon.json`，加入

```
{
  "insecure-registries":["192.168.59.137:5000"]
}
```

重启 docker 守护进程（还是在 host2 上）:

```shell
$ sudo service docker restart
```

再次访问 host1 上的 registry 服务：

```shell
$ docker pull 192.168.59.137:5000/busybox:1.25.1-musl
1.25.1-musl: Pulling from busybox
Digest: sha256:46634e32e559271b8e18b7f1b21d981da4cd63ed8d36fbdc35c0b56464238a0c
Status: Downloaded newer image for 192.168.59.137:5000/busybox:1.25.1-musl
```

以后在每台需要访问该 registry 服务的 Docker 主机上都需要这么修改。

## todo

给 registry 配置自签名的 HTTPS。

将 registry 配置成 hub 加速器。