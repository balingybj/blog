# 编译 Nginx-rtmp for ARM 并打包到容器中

利用 Nginx 和 nginx-rtmp 模块在树莓派上搭建流媒体服务器成功。但不知是不是树莓派硬件配置的原因，还是程序的问题。推流一段时间后树莓派就卡死，得重启才行。为了排除程序的问题，也为了方便以后的安装，决定将这个流媒体服务程序打包到 Docker 容器中。

## 编译 rtmp

编译 Nginx 和 nginx-rtmp 模块时用到的配置脚本如下：

```shell
#!/bin/bash

ROOT=/home/pi
INSTALL_PATH=$ROOT/rtmp-static-install-1.8.1

PCRE_PATH=$ROOT/pcre-8.39/
ZLIB_PATH=$ROOT/zlib-1.2.8/
OPENSSL_PATH=$ROOT/openssl-1.0.1u/
RTMP_PATH=$ROOT/nginx-rtmp-module-1.1.10/

./configure --prefix=$INSTALL_PATH --with-zlib=$ZLIB_PATH --with-pcre=$PCRE_PATH --with-openssl=$OPENSSL_PATH --add-module=$RTMP_PATH --with-cc-opt="-static -static-libgcc" --with-ld-opt="-static"
```

为了尽量不依赖基础镜像提供的环境，我选择的是静态编译。然后 `$ ./myconfigure.sh && make && make install`。大概十分钟能编译完，环境是树莓派 3b，Raspbian-jessie。

Nginx 用到的配置文件如下：

```
user  nginx;
worker_processes  1;

events {
    worker_connections  1024;
}

rtmp {
    server {
        listen 1935;

        application myapp {
            live on;
        }
    }
}
```

## 构建镜像

rtmp 服务程序编译出来了，在宿主机上测试成功。现在打包成 Docker 镜像。由于是在 ARM 架构的树莓派上运行，能选择的基础镜像很少。就选择体积最小的 `armhf/busybox:glibc`。完整的 `Dockerfile`如下：

```yaml
FROM armhf/busybox:glibc
COPY rtmp-static-install-1.8.1 ./rtmp
WORKDIR /rtmp
CMD ["./sbin/nginx", "-p", "./", "-g", "daemon off;"]
EXPOSE 1935
```

构建：

```shell
$ docker build -t rtmp:nginx-1.8.1 .
```

## 踩坑

镜像打包完成后，貌似一切正常。运行试试：

```shell
$ docker run --rm -it rtmp:nginx-1.8.1
nginx: [emerg] getpwnam("nginx") failed (2: No such file or directory) in ./conf/nginx.conf:1
```

为什么在宿主机上可以，在容器里就不行呢？上网搜了一下，并看了 Nginx 的官网文档。可能是因为 nginx 的工作进程需要以另外一个普通用户的身份运行。所以得为 Nginx 创建一个用户。我看 Docker 上 nginx 的官方镜像确实也另外创建了一个用户和用户组。那我也创建一个吧。在宿主机上运行时没创建，可能是因为编译的时候就创建了吧。OK，修改后的 `Dockerfile` 如下：

```yaml
FROM armhf/busybox:glibc
COPY arm-linux-gnueabihf /lib/
RUN addgroup -S nginx && adduser -D -S -H -G nginx nginx
WORKDIR /rtmp
CMD ["./sbin/nginx", "-p", "./", "-g", "daemon off;"]
EXPOSE 1935
```

重新构建后再运行试试：

```shell
$ docker run --rm -it rtmp:nginx-1.8.1
nginx: [emerg] getpwnam("nginx") failed (2: No such file or directory) in ./conf/nginx.conf:1
```

x，又是同样的错误。是不是我用户创建的有问题。于是我用 bash 命令运行镜像，进去看看用户和用户组有没有创建成功。

```shell
$ docker run --rm -it rtmp:nginx-1.8.1 /bin/sh
```

进去确认后发现用户和用户组创建成功，没毛病。百度到的结果都是在说创建用户的问题。我后来将基础镜像换成 alpine 也没用。我好像遇到了很多网友没经历过的大坑。这时候，我的那个弱鸡 VPN 好像可以用了，我必须的 Google 一下。用关键词 docker + 报错信息，终于找到了答案。

Google 到的这三个个页面：

[HowTo: Put nginx and PHP to jail in Debian 8 - blog.dornea.nu](http://blog.dornea.nu/2016/01/15/howto-put-nginx-and-php-to-jail-in-debian-8/)

[Slim application containers (using Docker) | fosiki](http://fosiki.com/blog/2015/04/28/slim-application-containers-using-docker/)

[How to create the smallest possible docker container of any image | Xebia Blog]( http://blog.xebia.com/how-to-create-the-smallest-possible-docker-container-of-any-image/)

都提到了打包镜像时遇到了同样的问题。原来是 nginx 需要获取相关用户信息，而读取信息需要用到一些库。这些库在我的基础镜像中没有，我静态编译的 nginx-rtmp 中也没有。所以访问用户信息失败，而报错信息又没说清楚。所以找到这些库并一起打包到镜像中就可以了。

>   如果你用的比较大的镜像，比如 Ubuntu，可能就不会遇到这问题，当然我没验证过。

## 解决

Nginx 访问用户信息时需要的库在 x86 平台上为一般为`/lib/i386-linux-gnu/libnss*` 或者 `/lib/x86-64-linux-gnu/libnss*`。在我的树莓派上则为 `/lib/arm-linux-gnueabihf/libnss*`。如果你用的交叉编译工具编译，则这些库在你的交叉编译工具下也有。

在 `Dockerfile` 所在的目录下创建一个目录，并将这些库文件全部复制到该目录下，方便后面打包到镜像中。

```shell
$ mkdir arm-linux-gnueabihf
$ cp /lib/arm-linux-gnueabihf/libnss* arm-linux-gnueabihf
```

`Dockerfile` 的内容也得跟着变一下：

```yaml
FROM armhf/busybox:glibc
COPY arm-linux-gnueabihf /lib/
COPY rtmp-static-install-1.8.1 ./rtmp
RUN addgroup -S nginx && adduser -D -S -H -G nginx nginx
WORKDIR /rtmp
CMD ["./sbin/nginx", "-p", "./", "-g", "daemon off;"]
EXPOSE 1935
```

重新构建后再运行一下试试：

```shell
$ docker run  -d rtmp:nginx-1.8.1
eda045811d571d2377fe6019b452050b31623cd16a5b915f1f84608aaf5f5712
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
eda045811d57        rtmp:nginx-1.8.1    "./sbin/nginx -p ./ -"   9 seconds ago       Up 5 seconds        1935/tcp            high_panini
$ docker top high_panini
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                30156               30125               0                   09:45               ?                   00:00:00            nginx: master process ./sbin/nginx -p ./ -g daemon off;
systemd+            30340               30156               0                   09:45               ?                   00:00:00            nginx: worker process
```

果然可以了。

## TODO

问题解决了。下一步把这个镜像上传到仓库保存起来。

但，还是有个小疑问：既然我 Nginx 是静态编译的，为什么访问用户信息时还需要加载额外的库呢？Nginx 是以何种方式调用访问用户信息的这种功能的呢？或者根本不是 Nginx 发起的调用，而是系统在验证？

Google 到的那三个页面都提到了用 `strace` 或 `inotifywait` 来跟踪程序的行为。学习一下，便于以后排查这种类似的问题。