# Docker Compose

## 概述

Compose 是专门用来定义和运行多容器应用的工具。使用 Compose ，你只需要用一个 Compose 文件配置服务，然后，只需要单条命令，就能根据配置创建并启动所有服务。更多功能特性请查看[功能特性列表](#功能特性)。

Compose 适用于开发、测试、staging 环境，以及 CI 工作流。

Compose 的使用基本分为三个步骤。

1.  用`Dockerfile`定义应用,这样应用可在任何地方再生。
2.  在`docker-compose.yml`中定义组成你应用的服务，这样它们就能在一个隔离的环境中一起远行。
3.  最后，执行`docker-compose up`命令，Compose 会启动你的整个应用。

一个典型的`docker-compose.yml`如下所示：

```yaml
version: '2'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Compose 文件的更多信息，参考[Compose file reference](https://docs.docker.com/compose/compose-file/)。

Compose 提供了一系列命令，用于管理整个应用的生命周期：

-   启动、停止和重建服务
-   查看正在运行的服务状态
-   流式化所有运行服务的日志输出
-   在某个服务上执行一次性命令

### 功能特性

-   同一个主机上多个隔离环境
-   重建容器时保留卷数据
-   只重新构建有更改的容器
-   变量和在不同环境之间移动应用


#### 同一主机多个隔离环境

Compose 使用项目名来隔离各个项目。项目名可以用在好几种不同的上下文中：

-   在开发主机上，创建同一个环境的多个拷贝（例如，你想为项目的每个特性分支运行一份拷贝）
-   在 CI 系统主机上，将每个项目名设置为唯一的构建编号，以隔离各个构建。
-   在共享主机或开发主机上，避免不同项目中名称相同的服务之间相互干扰。

默认的项目名是项目所在目录名。你可以通过命令行参数`-p`或环境变量`COMPOSE_PROJECT_NAME`设置项目名。

#### 重建容器时保留数据卷

Compose 会保存你所有服务用到的数据卷。每当`docker-compose up`执行时，它会检查容器之前是否运行过，如果之前的容器运行时有数据卷，它会将之前容器中的数据卷拷贝到新的容器中。这保证了你保存在数据卷中的数据不会丢失。

#### 只重建有更改的容器

Compose 会缓存创建容器所用的配置。当你重启一个没有更改的服务时，Compose 重用之前的容器。重用容器意味着你可以快速地修改环境。

#### 变量以及在不同的环境移动应用

Compose 支持在 Compose 文件中使用变量。你可以使用变量为不同的环境或用户定制服务。更多细节参考[Variable substitution](https://docs.docker.com/compose/compose-file/#variable-substitution)。

你可以通过`extends`域扩展 Compose 文件，或创建多个 Compose 文件。参考[extends](https://docs.docker.com/compose/extends/)获取更多细节。

### 常见的使用场景

Compose 可以有不同的使用方式。下面概括几种场景的使用场景。

#### 开发环境

当你在开发软件时，在一个隔离的环境中运行软件并与之交互是特别关键的。Compose 命令行工具可以用来创建环境并与之交互。

[Compose file](https://docs.docker.com/compose/compose-file/)可以提供了一种记录和配置应用所有服务依赖（数据库、消息队列、缓存、web service APIs 等）的方法。通过 Compose 命令行工具，你可以使用单条命令`docker-compose up`为每个依赖的服务启动一个或多个容器。

Compose 所有的这些特性，让开发者可以很方便的开始一个项目。Compose 将多个页面的“开发者入门指南”缩减成单个机器可读的 Compose 文件和几条命令。

#### 自动测试环境

任何持续部署和持续集成过程中很重要的一部分就是自动特性套件。自动化的端到端测试需要一个环境来运行测试。Compose 为测试套件提供了一种便捷的方法来创建和销毁隔离的测试环境。通过在一个[Compose 文件](https://docs.docker.com/compose/compose-file/)定义完整的环境信息，用几条简单命令行就可以创建和销毁测试环境：

```shell
$ docker-compose up -d
$ ./run_tests
$ docker-compose down
```

#### 单机部署环境

之前，Compose 一直将重心放在开发和测试工作流上，但是随着各个版本的发布，我们都在生产型功能上取得进步。里可以使用 Compose 部署到远程的 Docker 引擎上。Docker 引擎可能是通过[Docker Machine](https://docs.docker.com/machine/overview/)部署的单个实例，也可能是整个[Docker Swarm](https://docs.docker.com/swarm/overview/)集群。

关于生产型功能的更多细节，参考文档中的[compose in production](https://docs.docker.com/compose/production/)。

### 发现说明

关于 Docker Compose 当前版本和之前版本变化的详细列表，参考[CHANGELOG](https://github.com/docker/compose/blob/master/CHANGELOG.md)。

## 安装

安装 Docker Compose 之前先确保你已经安装了 Docker Engine。

### 获取安装文件

```shell
$ curl -L "https://github.com/docker/compose/releases/download/1.8.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

该命令会根据操作系统和系统架构选择对应的文件下载，并复制为`/usr/local/bin/docker-compose`。

如果`curl`下载速度过慢，你可以在系统上自己运行`uname -s`和`uname -m`，根据结果组成下载文件地址，粘贴到浏览器里下载，这样会快很多。下载完后再复制到`/usr/local/bin/docker-compose`。

如果提示权限不够，用`sudo sh -c`执行。

### 给与可执行权限

```shell
$ sudo chmod +x /usr/local/bin/docker-compose
```

### 验证

```shell
$ docker-compose version
docker-compose version 1.8.1, build 878cff1
docker-py version: 1.10.3
CPython version: 2.7.9
OpenSSL version: OpenSSL 1.0.1e 11 Feb 2013
```

## 开始使用

本节我们将用 Docker Compose 创建并运行一个简单的 Python web 应用。该应用使用 Flask 框架，并在每次访问时输出已访问的次数，该次数存储在 Redis 中。虽然该示例用到了 Python，但此处展示的概念是十分好理解的，即使你不熟悉 Python。

### Step 1: 设置

1.  创建项目所使用的目录

    ```shell
    $ mkdir composetest
     $ cd composetest
    ```

2.  用你最喜欢的编辑器在项目目录下编辑一个`app.py`文件，添加以下内容：

    ```shell
    from flask import Flask
     from redis import Redis

     app = Flask(__name__)
     redis = Redis(host='redis', port=6379)

     @app.route('/')
     def hello():
         redis.incr('hits')
         return 'Hello World! I have been seen %s times.' % redis.get('hits')

     if __name__ == "__main__":
         app.run(host="0.0.0.0", debug=True)
    ```

3.  再在项目目录下创建一个`requirements.txt`文件，添加以下内容：

    ```shell
    flask
    redis
    ```

    该文件定义了Python 应用的依赖。

4.  创建一个`pip.conf`文件，并在其中添加以下内容：

    ```ini
    [global]
    index-url = https://pypi.tuna.tsinghua.edu.cn/simple
    ```

    熟悉 Python 的同学应该知道，这是一个 PIP 的配置文件，我们在其中配置了 PyPi 源，并将其设为清华源地址。考虑国内访问官方 PyPi 源的网速问题，还是使用国内的源比较好。

### Step 2: 创建 Docker 镜像

在这一步，我们创建一个 Docker 镜像。该镜像包含了 Python 应用所有的依赖，包括 Python 本身。

1.  在项目目录下创建一个`Dockerfile`文件，并添加以下内容：

    ```dockerfile
    FROM python:2.7.12-alpine
    COPY pip.conf /root/.pip/pip.conf
    COPY app.py /code/
    COPY requirements.txt /code/
    WORKDIR /code
    RUN pip install -r requirements.txt
    CMD python app.py
    ```

    这是告诉 Docker：

    -   从 python:2.7.12-alpine 镜像开始构建一个新镜像。(选择 python:2.7.12-alpine 为基础镜像，是因为该镜像比较小)
    -   将当前目录下的`.pip`文件拷贝到容器的 `/root/.pip/pip.conf`，配置 PyPi 源。
    -   将`app.py`、`requirementstxt`拷贝到容器的`/code`目录下。
    -   将工作目录设为`/code`。
    -   安装 Python 依赖。
    -   将容器的默认命令设为`python app.py`。

2.  构建镜像

    ```shell
    $ docker build -t web .
    ```

    该命令根据当前目录的上下文创建了一个名为`web`的镜像。该命令会自动定位`Dockerfile`、`app.py`和`requirement.txt`文件。

### Step 3: 定义服务

使用 `docker-compose.yml`定义一组服务：

在项目目录下创建一个`docker-compose.yml`文件，并添加以下内容：

```ya
version: '2'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
    depends_on:
     - redis
  redis:
    image: redis:3.2.5-alpine
```

这个 Compose 文件定义了两个服务,`web`和`redis`。其中，`web`服务：

-   从当前目录下的`Dockerfile`文件创建。
-   将暴露的 5000 端口转发到宿主机的 5000 端口。
-   将宿主机上的该项目目录挂载到容器的`/code`目录，这样可使得你修改代码后不需要重新构建镜像。
-   将 web 服务链接到 redis 服务。


`redis`服务使用从  Docker Hub 上拉取的最新官方[Redis](https://registry.hub.docker.com/_/redis/)镜像（redis:3.2.5-alpine 较小，所以我用它）。

### Step 4: 构建并运行应用

1.  cd 到项目目录下

    ```shell
    $ docker-compose up
    Building web
    Step 1 : FROM python:2.7.12-alpine
     ---> 055c3b9a9656
    Step 2 : COPY pip.conf /root/.pip/pip.conf
     ---> Using cache
     ---> 623aca7522ad
    Step 3 : COPY app.py /code/
     ---> Using cache
     ---> 68fb433dcae9
    Step 4 : COPY requirements.txt /code/
     ---> Using cache
     ---> 76bef0e9afee
    Step 5 : WORKDIR /code
     ---> Using cache
     ---> 813d783ae0db
    Step 6 : RUN pip install -r requirements.txt
     ---> Running in 3a1bc1fb64b9
    Collecting flask (from -r requirements.txt (line 1))
      # 这里是一大堆 pip install -r requirements.txt 的输出信息
    Removing intermediate container 3a1bc1fb64b9
    Step 7 : CMD python app.py
     ---> Running in 67c702c6f831
     ---> 274f32fda062
    Removing intermediate container 67c702c6f831
    Successfully built 274f32fda062
    WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
    Creating composetest_redis_1
    Creating composetest_web_1
    Attaching to composetest_redis_1, composetest_web_1
    redis_1  | 1:C 28 Nov 11:34:57.711 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
    redis_1  |                 _._                                                  
    redis_1  |            _.-``__ ''-._                                             
    redis_1  |       _.-``    `.  `_.  ''-._           Redis 3.2.5 (00000000/0) 64 bit
    redis_1  |   .-`` .-```.  ```\/    _.,_ ''-._                                   
    redis_1  |  (    '      ,       .-`  | `,    )     Running in standalone mode
    redis_1  |  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
    redis_1  |  |    `-._   `._    /     _.-'    |     PID: 1
    redis_1  |   `-._    `-._  `-./  _.-'    _.-'                                   
    redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
    redis_1  |  |    `-._`-._        _.-'_.-'    |           http://redis.io        
    redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'                                   
    redis_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
    redis_1  |  |    `-._`-._        _.-'_.-'    |                                  
    redis_1  |   `-._    `-._`-.__.-'_.-'    _.-'                                   
    redis_1  |       `-._    `-.__.-'    _.-'                                       
    redis_1  |           `-._        _.-'                                           
    redis_1  |               `-.__.-'                                               
    redis_1  | 
    redis_1  | 1:M 28 Nov 11:34:57.748 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
    redis_1  | 1:M 28 Nov 11:34:57.748 # Server started, Redis version 3.2.5
    redis_1  | 1:M 28 Nov 11:34:57.748 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
    redis_1  | 1:M 28 Nov 11:34:57.748 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
    redis_1  | 1:M 28 Nov 11:34:57.748 * The server is now ready to accept connections on port 6379
    web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    web_1    |  * Restarting with stat
    web_1    |  * Debugger is active!
    web_1    |  * Debugger pin code: 110-719-698

    ```

    从输出信息，我们可以看出：Compose 从项目代码构建一个镜像（实际是重用前面构建的 web 镜像），拉取了一个 Redis 镜像，并启动`docker-compose.yml`中定义的服务。

    想要停止，输入`ctrl c`就行。我输入`ctrl c`后的输出如下：

    ```shell
    ^CGracefully stopping... (press Ctrl+C again to force)
    Stopping composetest_web_1 ... done
    Stopping composetest_redis_1 ... done
    ```

2.  打开本地浏览器，在浏览器中打开网址`http://0.0.0.0:5000/`，验证应用是否运行。

    如果你宿主机用的是 Linux 发行版，该 web 应用应该在 5000 端口监听。如果`http://0.0.0.0:5000/`不启作用，试试`http://localhost:5000`。

    如果你是在 Mac 上使用 Docker Machine，使用`docker-machine ip MACHINE_VM`查询你 Docker 宿主机的 IP。然后在浏览器里打开`http://MACHINE_VM_IP:5000`。

    打开浏览器后，你应该可以在浏览器里看到：

    `Hello World! I have been seen 1 times.`

3.  刷新浏览器

    浏览器中输出信息中访问次数应该会增加。

### Step 5: 试试其他命令

#### 以后台方式启动应用

```shell
~/composetest$ docker-compose up -d
Starting composetest_redis_1
Starting composetest_web_1
~/composetest2$ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                    NAMES
fff0d5d5fe97        composetest2_web     "/bin/sh -c 'python a"   14 minutes ago      Up About a minute   0.0.0.0:5000->5000/tcp   composetest_web_1
8075edb9cc12        redis:3.2.5-alpine   "docker-entrypoint.sh"   14 minutes ago      Up About a minute   6379/tcp                 composetest_redis_1
```

`docker-compose up`加`-d`参数就是后台启动的意思。

#### 执行一次性命令

`ocker-compose run` 命令可以在指定的服务上执行一次性命令。例如，查看`web`服务的环境变量：

```shell
~/composetest$ docker-compose run web env
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=091a25e0b332
TERM=xterm
LANG=C.UTF-8
GPG_KEY=C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF
PYTHON_VERSION=2.7.12
PYTHON_PIP_VERSION=8.1.2
HOME=/root
```

#### 停止服务

前面我用前台方式启动应用的所有服务，用`ctrl c`就能停止所有服务。而这次我们用后台方式启动的服务得用`docker-compose stop`停止。

```shell
~/composetest$ docker-compose stop
Stopping composetest_web_1 ... done
Stopping composetest_redis_1 ... done
```

更多命令，参考`docker-compose --help`。
