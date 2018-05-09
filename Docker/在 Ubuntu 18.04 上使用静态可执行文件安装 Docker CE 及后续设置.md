# 在 Ubuntu 18.04 上使用静态可执行文件安装 Docker CE 及后续设置



前几天安装了最新的 Ubuntu 18.04 LTS，想在上面安装一个 docker，结果发现 docker 官方的软件源里没有Ubuntu 18.04 的安装包版本，可能是系统太新了，官方还没来得及制作。之前用 Ubuntu 17.04 也是这样，上次我在 17.04 上是在官方的软件源里下载了 16.04 的 .deb 安装包到本地直接安装的，也能正常工作。不过我觉得这种方式有点不靠谱，万一不同版本的安装包有点不兼容怎么办。所以这次我选择使用官方的提供了静态可执行文件安装。这里记录一下，省的下次安装时再去官网看文档，不知道为啥，docker 文档的网页总是被墙。而且官方软件源也访问巨慢，还得寻找国内的镜像源，这里做个笔记，节约下次的安装时间。

如果官方没有为你的目标环境提供安装包，就可以使用这种安装方式。

> 如果你嫌这种安装方式麻烦，可是直接下载官方为 Ubuntu 17.10 提供的 .deb 包安装。理论上为 17.10 提供的 .deb 包也适用于 18.04。不过本人并没完整测试，只是之前在 17.10 上用过 16.04 的 .deb 安装包。
>
> ```shell
> $ wget http://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/dists/artful/pool/stable/amd64/docker-ce_18.03.1~ce-0~ubuntu_amd64.deb
> $ sudo dpkg -i docker-ce_18.03.1~ce-0~ubuntu_amd64.deb
> ```
>
> 这样可以自动让系统的 systemd 工具来管理 docker 后台进程，省去一些设置上的麻烦。

## 获取静态的二进制文件存档

###从官方源下载

```shell
$ wget https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz
```

https://download.docker.com/linux/static 目录下面有不同的版本，比如stable、edge、test三个分支版本，以及不同硬件平台的版本。

###or 从国内镜像源下载

但是我发现这个下载过程特别慢，所以使用阿里的镜像源更快一些。

```shell
$ wget https://mirrors.aliyun.com/docker-ce/linux/static/stable/x86_64/docker-18.03.1-ce.tgz
```

## 提取出二进制文件

```shell
$ tar xzvf docker-18.03.1-ce.tgz
```

这一步会提取出一个 docker 目录，里面包含了使用 Docker 需要的各个可执行文件。

```shell
$ ls docker
docker  docker-containerd  docker-containerd-ctr  docker-containerd-shim  dockerd  docker-init  docker-proxy  docker-runc
```

## copy 二进制文件到相应目录

```shell
$ sudo cp docker/* /usr/bin/
```

现在就可以在里的系统上使用 Docker 了。

## 启动 dockerd

```shell
$ sudo dockerd &
```

可以看到一大堆输出信息。

## 查看 Docker 版本

```shell
$ docker version
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.2
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:12:25 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.37/version: dial unix /var/run/docker.sock: connect: permission denied
```

可看到版本为目前最新的 18.03.1-ce，但是只输出了 client 的版本信息，server 端的信息只有一个 permission denied 错误。这是因为没有使用 sudo 权限。重新用 sudo 执行一遍：

```shell
$ sudo docker version
sudo docker version
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.2
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:12:25 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.03.1-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   9ee9f40
  Built:        Thu Apr 26 07:23:03 2018
  OS/Arch:      linux/amd64
  Experimental: false
```

## 运行一个 hello-world 容器

```shell
~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete 
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

该命令会从官方的仓库里拉取一个 hello-world 镜像，然后创建容器并运行。如果顺利的话，你会看到以上输出。但是笔者这里由于网络原因，拉取镜像的时候超时了，后面换了国内的镜像源才成功。

## 设置开机启动

使用官方的提供为特定系统制作的安装包安装 Docker 时，安装过程中会自动设置开机启动。由于我们是自己使用静态的可执行文件，所以每次得自己运行 `sudo dockerd &`启动 dockerd 进程，为了方便，我们自己设置一下开机启动。Docker 在 Ubuntu 16.04 及以上版本上是利用 Ubuntu 系统的 systemd 初始化系统来管理 dockerd 进程，我们也来模仿一下。本人并不熟悉 systemd，只是依葫芦画瓢。

在进行下面的步骤之前先停掉前面启动的 dockerd 进程。

### 创建 systemd 单元文件

新建一个 docker.service 文件，在里面填入一下内容：

```shell
[Unit]
Description=docker static

[Service]
ExecStart=/usr/bin/dockerd

[Install]
WantedBy=multi-user.target
```

将其 copy 到`/lib/systemd/system/`目录下：

```shell
$ sudo cp docker.service /lib/systemd/system/
```

###重载 systemd 单元配置文件

```shell
$ sudo systemctl daemon-reload 
```

### 使用 systemd 启动 docker 服务

```shell
$ sudo systemctl start docker
```

### 验证

使用 `docker verison `和 `docker info` 验证一下 docker 服务是否启动成功。

```shell
~$ sudo docker version
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.2
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:12:25 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.03.1-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   9ee9f40
  Built:        Thu Apr 26 07:23:03 2018
  OS/Arch:      linux/amd64
  Experimental: false
```

看起来没啥问题。

### 允许开机启动

```shell
$ sudo systemctl enable docker.service 
[sudo] chao 的密码： 
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.
```

这样开机的时候 systemd 就会自动启动 dockerd。笔者重启系统发现 dockerd 真的已经启动了。

## 解决 docker cli 权限问题

前面我们在使用 docker 命令时都是通过 sudo 的方式，这是因为：dockerd 进程启动会绑定到一个 Unix socket，默认情况下，该 Unix socket 被 root 用户所有，其他用户可以通过 sudo 来访问该它。而 dockerd 都是以 root 用户运行，所以 dockerd 可以绑定到该 Unix socket。但普通用户在使用 docker cli 也就是 docker 命令来使用 docker 服务时也需要用的该 Unix socket，docker 命令需要访问该 Unix socket，通过该 Unix socket 将用户的指令发送给 dockerd 进程。所以普通用户在使用执行 docker 命令时需要通过 sudo 的方式。

如果想要避免每次在执行 docker 命令都加一个 sudo，可以创建一个 docker 用户组，并将要使用 docker 的普通用户添加到该组。dockerd 进程启动时会将 Unix socket 的读写权限都赋予给 docker 用户组。这样就不需要使用 sudo 了。

### 创建 docker 用户组

```shell
$ sudo groupadd docker
```

### 添加当前用户到 docker 用户组

```shell
$ sudo usermod -aG docker $USER
```

### 注销当前用户或重启系统

接下来注销当前用户并重新登录，使前面的设置生效。如果是虚拟机环境，你可能需要重启虚拟机系统才行。

### 测试

重新登录系统，执行一条 docker 命令测试一下：

```shell
$ docker run hello-world
```

## 使用 docker hub 国内镜像

Docker 默认从官方的 hub.docker.com 拉取镜像，但是由于网络的原因，在国内拉取镜像巨慢，甚至会超时而失败。推荐使用国内的镜像站。国内有好几个镜像站，但是好像只有 ustc 提供的镜像站不需要注册，其他镜像站都要求用户注册一个账号，然后给用户分配一个唯一的加速地址，个人觉得太麻烦了，还是 ustc 的比较良心。

在 `/etc/docker/daemon.json` 文件（没有就新建一个）添加以下内容：

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

然后拉取镜像就非常快了。