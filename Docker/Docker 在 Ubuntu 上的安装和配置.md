# Docker 在 Ubuntu 上的安装和配置

以下安装过程只在 Ubuntu 16.04 x64 上做过测试。我们建议安装的是 Docker 官方维护的版本，而不是 Ubuntu 官方软件仓库中的版本。一般 Ubuntu 官方维护的版本会比 Docker 官方维护的版本低两个版本号。如果你不想使用 Docker 的最新特性，只想快速的尝试一下 Docker，使用`$ sudo apt install docker.io`安装 Ubuntu 官方维护的版本就行。

## 安装

### 1. 修改 APT  源

Docker 的安装过程需要执行`apt update`，使用默认的 apt 源会很慢。所以一般使用国内的镜像源。我电的镜像源长期不稳定，推荐使用清华、阿里、中科大的源。这里用清华源举例。

```shell
$ sudo gedit /etc/apt/sources.list
```

将里面的内容替换为：

`````shell
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
`````

我这里的源对应的是 Ubuntu 16.04 的版本。如果你使用的是其他版本，请参考清华源的镜像[使用帮助](https://mirrors.tuna.tsinghua.edu.cn/)。

### 2. 安装

按照官方文档，Docker 安装过程大概需要几个步骤：

-   安装一些依赖的软件包
-   添加 Docker 官方的 `GPG` key
-   添加 Docker 官方的 APT 源
-   安装

这几个步骤根据不同的 Linux 发行版和具体发行版的不同版本会有一些差别。幸运的是官方提供了自动化安装的脚本，该脚本会检测你的系统环境，帮你做完所有上面的所有事情。但不幸的是，由于我们的网络环境访问起来会特别慢，安装十分耗时，而且能否一次成功全靠运气。但又幸运的是，阿里云提供了这个脚本和所使用资源的镜像源，可以用正常的网速访问。所以我们推荐使用阿里云的镜像源。

#### 2.1 使用阿里云镜像源安装（推荐）

```shell
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

详情参考阿里云的[帮助页面](http://mirrors.aliyun.com/help/docker-engine)。

#### 2.2 使用 Docker 官方源安装

```shell
$ curl -sSL https://get.docker.com/ | sudo sh
```

### 3. 验证

```shell
$ sudo docker info
$ sudo docker version
```

你应该能看到一些输出信息。

## 基本配置

### 将当前用户加入 docker 用户组

docker 命令需要使用`sudo` 权限来运行。每次都输入`sudo`很不方便。讲当前用户加入 docker 用户组就不用这么麻烦了。

```shell
$ sudo usermod -aG docker ${USER}
```

#### 重新登录系统

上面的修改只有重新登录系统后才能生效。所以先 logout 再 loginin。

#### 验证

```shell
$ docker info
```

现在可以不用`sudo`来运行 docker 命令了。

### 使用中科大的 Docker Hub 镜像源加速

每次使用`docker pull`命令 pull 镜像时，docker daemon 都会去 Docker Hub 拉取镜像，网速较慢。我们可以使用中科大的镜像源来加速。

#### 编辑配置文件

```shell
$ sudo gedit /etc/docker/daemon.json
```

填入

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

#### 重启 docker 守护进程

```shell
$ sudo service docker restart
```

#### 验证

```shell
$ docker run hello-world
```

该命令会去 Docker Hub 拉取一个名为 hello-world 的镜像，从该镜像创建并运行一个容器，输出`hello world`。如果没有配置镜像源加速，拉取速度可能会很慢。

```shell
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              c54a2cc56cbb        3 months ago        1.848 kB
```

可以看到，当前系统中存在一个我们刚 pull 的`hello-world`镜像。

你还可以拉取一个 ubuntu 镜像试试

```shell
$ docker pull ubuntu
```