# Docker swarm 模式初体验

本教程将介绍 Docker Engine Swarm 模式。先介绍 swarm 模式几个关键的概念，然后通过动手实践带你体验一下 swarm 模式。

## 几个关键概念

### Swarm

Docker Engline 中集成的集群管理和编排功能都是基于 **SwarmKit** 实现。参与到集群中的 Docker Engine 会进入 swarm 模式。比如初始化一个 swarm 或加入已有的 swarm。

一个 **swarm** 就是个一个 Docker Engine 集群，你可以在上面部署**服务**。Docker Engine CLI 提供了用于 swarm 管理的命令，比如添加或移除一个节点。同样还提供了用于部署服务到 swarm 和管理服务编排的命令。

当 Docker Engine 运行在 swarm 模式时，你管理容器。当 Docker Engine 运行在 swarm 模式时，你编排服务。

### 节点

每个参与到 swarm 中的 Docker Engine 都称之为一个节点。

将服务的定义提交到**管理节点**即可将应用部署到 swarm。然后管理节点将叫作**任务**的工作单元分发到工作节点。

为了维持 swarm 的目标状态，管理节点还将承当编排和集群管理的功能。一般有多个管理节点，它们之间会选出一个领导来进行编排任务。

**工作节点**接收并执行来自管理节点分发的任务。默认情况下，管理节点也是工作节点，你可以把它配置成只当管理节点。代理通知管理节点已分配任务的当前状态，以便管理节点维护所需的状态。

### 服务和任务

服务定义了需要在工作节点上执行的任务。它是 swarm 系统的中心结构，也是用户和 swarm 交互的主要根源。

服务包括要使用的容器镜像，以及在容器中执行的指令。

对于**复制服务**，管理节点根据服务的规模和目标状态将一定数量的任务副本分发到各节点上。

对于**全局服务**，每个节点上都将分发一个任务副本。

**任务**包含一个容器和需要在容器中执行的指令。它是 swarm 原子调度单位。管理节点根据服务规模中定义的副本数量将任务分配给工作节点。一旦某个任务被分配到某个节点，就不能再移动到其他节点。它只能在分配的节点上运行或者失败。

### 负载均衡

swarm 管理节点使用 **入口负载均衡** 的方式暴露你想要让外部访问的服务。Swarm 管理节点可以自动将一个服务分配到某个 **发布端口**，或者你可以为服务指定一个**发布端口**。你可以指定任意未使用的端口。如果你没指定端口，swarm 管理节点将为服务指定一个 30000-32757 之间的端口。

外部组件，比如云负载均衡器，可以通过集群中的任意节点访问发布端口上的服务，不管当前节点上是否有服务对应的任务在运行，swarm 中的所有节点都会将进入连接路由到正在运行任务的实例上。

Swarm 模式有一个内置的 DNS 组件，它会自动给 swarm 中的每个服务分配一个 DNS 条目。Swarm 管理器使用**internal load balancing** 基于服务对应的 DNS 名称在服务之间分发请求。

## 开始实践

本教程将指导你完成以下任务：

-   在 Docker Engline swarm 模式下初始化一个集群
-   添加节点到 swarm
-   部署服务到 swarm
-   在一切就绪后管理 swarm

## 准备工作

在开始本教程之前，你需要准备一下几样东西：

-   三台通过网络连接的主机
-   每台主机安装 Docker Engine 1.12 或更新版本
-   充当管理节点的主机 IP
-   主机之间开发下面提到的端口

### 主机之间端口开放

主机之间的以下端口必须是开放。某些环境下，这些端口默认是允许的：

-   TCP 端口 2377 用于集群管理通信（管理节点）
-   TCP 和 UDP 端口 7946 用于节点间通信（所有节点）
-   TCP 和 UDP 端口 4789 用于 overlay 网络流量（所有节点）


如果你的这些端口没有打开，可以用`iptables`命令打开它们：

```shell
iptables -A INPUT -p tcp --dport 2377 -j ACCEPT
iptables -A INPUT -p tcp --dport 7946 -j ACCEPT
iptables -A INPUT -p udp --dport 7946 -j ACCEPT
iptables -A INPUT -p tcp --dport 4789 -j ACCEPT
iptables -A INPUT -p udp --dport 4789 -j ACCEPT
```




## 创建 swarm

### 创建一个 swarm

随意选择一个主机作为管理节点，在上面初始化一个 swarm：

```shell
chao@manager01:~$ docker swarm init --advertise-addr 192.168.59.128
Swarm initialized: current node (7ik7wqhe5wcag8k5tp816c7ck) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-0p0p5f96e1w4xblhw2eeookrv46spwf4yx7qmve2srxe9wec5g-ellbnyt4cwwvvdkssaj0cbtus \
    192.168.59.128:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

`--advertise-addr` 标志配置了管理节点的 IP 地址。如果你的机器只有一个 IP 地址，可以省略`--adbertise-addr`选项，docker 会自动选择正确的 IP。上输出信息说明了怎样加入新的工作节点。也说明了执行`docker swarm join-token manager` 可以查询怎样加入新的管理节点。

### 执行`docker info`命令查看`swarm`的当前状态

```shell
~$ docker info
...
Swarm: active
 NodeID: 7ik7wqhe5wcag8k5tp816c7ck
 Is Manager: true
 ClusterID: 2scd04fv8c9mua1jiaq6n0370
 Managers: 1
 Nodes: 1
 ...
```

### 执行 `docker node ls` 命令查看节点信息

```shell
$ docker node ls
ID                           HOSTNAME   STATUS  AVAILABILITY  MANAGER STATUS
7ik7wqhe5wcag8k5tp816c7ck *  manager01  Ready   Active        Leader
```

节点 id 后面的`*`表示你当前连接到了该节点。

## 添加节点到 swarm

### 添加两个工作节点到`swarm`

在第二台主机上，执行前面创建 `swarm` 时 `docker swarm init` 输出信息中命令创建工作节点并加入到 `swarm`

```shell
chao@worker01:~$ docker swarm join \
     --token SWMTKN-1-0p0p5f96e1w4xblhw2eeookrv46spwf4yx7qmve2srxe9wec5g-ellbnyt4cwwvvdkssaj0cbtus \
     192.168.59.128:2377
This node joined a swarm as a worker.
```

输出信息表示当前节点已是 `swarm` 中的一个工作节点了。如果你忘记了该命令，可以在管理节点上执行 `docker swarm join-token worker ` 查询怎么加入。

```shell
~$ docker swarm join-token worker 
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3wi7tszkolocsbc7vopv1tfx2r2h1owtqegwevdqqdk3fj195u-ejpeq0afjvfmujlvzboux9zjs \
    192.168.59.128:2377
```

到第三台主机上，继续讲第三台主机加入到 `swarm` 中

```shell
chao@worker02:~$ docker swarm join \
     --token SWMTKN-1-0p0p5f96e1w4xblhw2eeookrv46spwf4yx7qmve2srxe9wec5g-ellbnyt4cwwvvdkssaj0cbtus \
     192.168.59.128:2377
This node joined a swarm as a worker.
```

### 查看所有节点的状态

在管理节点上

```shell
chao@manager01:~$ docker node ls
ID                           HOSTNAME   STATUS  AVAILABILITY  MANAGER STATUS
4rkvaeauhftolat98u5ty01iz    worker02   Ready   Active        
7ik7wqhe5wcag8k5tp816c7ck *  manager01  Ready   Active        Leader
cashcvy4qzcq3lvtnix4trqu7    worker01   Ready   Active
```

输出信息第二行 id 后面的`*`表示当前连接到了该节点。`HOSTNAME` 栏输出节点的 hostname。`MANAGER` 用于指示 `swarm`中的管理节点，该栏值为 `Leader` 表示为管理节点，空值表示为工作节点。

## 部署一个服务到 `swarm`

### 创建服务

在管理节点上创建一个服务，每隔三秒输出一个 "hello world"：

```shell
chao@manager01:~$ docker service create --replicas 1 --name helloworld busybox:1.25.1-musl /bin/sh -c "while true; do echo hello world; sleep 3; done"
04a3iqg8zlhba84kpi2tatssf
```

-   `docker service create` 命令创建服务。
-   `--name` 标志将服务命名为`helloworld`。
-   `--replicas` 标志指定了期望状态为 1 个运行示例。
-   参数 `busybox:1.25.1-musl /bin/sh -c "while true; do echo hello world; sleep 3; done` 将服务定义为使用镜像`busybox:1.25.1-musl` 创建容器，并在里面执行 `/bin/sh -c "while true; do echo hello world; sleep 3; done`。

### 查看服务列表

还是在管理节点上

```shell
chao@manager01:~$ docker service ls
ID            NAME        REPLICAS  IMAGE                COMMAND
04a3iqg8zlhb  helloworld  0/1       busybox:1.25.1-musl  /bin/sh -c while true; do echo hello world; sleep 3; done
```

## 查看服务的详细信息

### 查看服务的详细信息

在管理节点上。

```shell
$ docker service inspect --pretty helloworld 
ID:		04a3iqg8zlhba84kpi2tatssf
Name:		helloworld
Mode:		Replicated
 Replicas:	1
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
ContainerSpec:
 Image:		busybox:1.25.1-musl
 Args:		/bin/sh -c while true; do echo hello world; sleep 3; done
Resources:
```

参数`--pretty`表示以可读性良好的格式输出。如果想输出详细的 json 格式信息，去掉`--pretty`参数即可。

```shell
$ docker service inspect  helloworld 
[
    {
        "ID": "04a3iqg8zlhba84kpi2tatssf",
        "Version": {
            "Index": 23
        },
        "CreatedAt": "2016-12-22T15:00:03.780379494Z",
        "UpdatedAt": "2016-12-22T15:00:03.780379494Z",
        "Spec": {
            "Name": "helloworld",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "busybox:1.25.1-musl",
                    "Args": [
                        "/bin/sh",
                        "-c",
                        "while true; do echo hello world; sleep 3; done"
                    ]
                },
                "Resources": {
                    "Limits": {},
                    "Reservations": {}
                },
                "RestartPolicy": {
                    "Condition": "any",
                    "MaxAttempts": 0
                },
                "Placement": {}
            },
            "Mode": {
                "Replicated": {
                    "Replicas": 1
                }
            },
            "UpdateConfig": {
                "Parallelism": 1,
                "FailureAction": "pause"
            },
            "EndpointSpec": {
                "Mode": "vip"
            }
        },
        "Endpoint": {
            "Spec": {}
        },
        "UpdateStatus": {
            "StartedAt": "0001-01-01T00:00:00Z",
            "CompletedAt": "0001-01-01T00:00:00Z"
        }
    }
]
```

### 查看哪个节点在运行该服务

还是在管理节点上：

```shell
$ docker service ps helloworld 
ID                         NAME          IMAGE                NODE       DESIRED STATE  CURRENT STATE          ERROR
3mmy1k19z3hdoa62wz63pkmjt  helloworld.1  busybox:1.25.1-musl  manager01  Running        Running 4 minutes ago 
```

输出信息表明，`helloworld` 服务的一个实例在 `manager01` 节点上执行。这是因为，默认情况下管理节点是工作节点。

`DESIRED STATE`和`CURRENT STATE`表示服务的期望状态和当前状态，你可以对比它们，判断服务是否想期望的那样运行。这里的`Running`和`Running 4 minutes ago`说明服务运行正常。

### 在执行任务的节点上使用`docker ps`命令查看相关容器的详细信息

```shell
$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS               NAMES
a471b58c133a        busybox:1.25.1-musl   "/bin/sh -c 'while tr"   6 minutes ago       Up 6 minutes                            helloworld.1.3mmy1k19z3hdoa62wz63pkmjt
```

## 伸缩服务

### 伸缩服务的任务数量

在管理节点上

```shell
$ docker service scale helloworld=5
helloworld scaled to 5
```

我们将服务的数量伸缩到5。

### 查看服务列表

在管理节点上：

```shell
$ docker service ps helloworld 
ID                         NAME          IMAGE                NODE       DESIRED STATE  CURRENT STATE            ERROR
3mmy1k19z3hdoa62wz63pkmjt  helloworld.1  busybox:1.25.1-musl  manager01  Running        Running 9 minutes ago    
7s417360vd8ja49xfdt8egbcx  helloworld.2  busybox:1.25.1-musl  worker02   Running        Preparing 9 seconds ago  
bmvcbb1basbbka0kjqplm862v  helloworld.3  busybox:1.25.1-musl  worker01   Running        Preparing 9 seconds ago  
5aj7nyio2zo62xswl19kyg8o8  helloworld.4  busybox:1.25.1-musl  worker01   Running        Preparing 9 seconds ago  
bwb8vie6mxtz8a55qo26up0mo  helloworld.5  busybox:1.25.1-musl  manager01  Running        Preparing 9 seconds ago  
```

可以看到 9 秒前又创建了 4 个任务，现在一共有 5 个任务了。有两个在 `manager01` 上执行。

### 在各节点上查看服务

首先是管理节点：

```shell
$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED              STATUS              PORTS               NAMES
98da817cca74        busybox:1.25.1-musl   "/bin/sh -c 'while tr"   About a minute ago   Up About a minute                       helloworld.5.bwb8vie6mxtz8a55qo26up0mo
a471b58c133a        busybox:1.25.1-musl   "/bin/sh -c 'while tr"   11 minutes ago       Up 11 minutes                           helloworld.1.3mmy1k19z3hdoa62wz63pkmjt
```

再是工作节点1

```shell
$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS               NAMES
16f0b076a2f6        busybox:1.25.1-musl   "/bin/sh -c 'while tr"   55 seconds ago      Up 54 seconds                           helloworld.3.bmvcbb1basbbka0kjqplm862v
00fb48c39eb1        busybox:1.25.1-musl   "/bin/sh -c 'while tr"   56 seconds ago      Up 55 seconds                           helloworld.4.5aj7nyio2zo62xswl19kyg8o8
```

再是工作节点2

```shell
$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS               NAMES
68846b348c62        busybox:1.25.1-musl   "/bin/sh -c 'while tr"   2 minutes ago       Up 2 minutes                            helloworld.2.7s417360vd8ja49xfdt8egbcx
```

可以看到：管理节点上运行了两个任务，工作节点1运行了两个任务，工作节点2运行了1个任务。

## 删除 `swarm` 上运行的服务

### 删除

在管理节点上

```shell
$ docker service rm helloworld 
helloworld
```

### 确认

在管理节点上

```shell
$ docker service ls
ID  NAME  REPLICAS  IMAGE  COMMAND
```

看不到任何服务了。

在工作节点上：

```shell
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

可以看到，服务被删除后，所有节点上都没有了和任务相关的容器。

## 滚动更新

在这部分，我们将部署一个基于 Rddis 3.07 镜像的服务。然后使用滚动更新将服务更新到使用 Redis 3.2.5 镜像。

### 在`swarm`中部署 Redis 3.0.7，并配置 10 秒更新延迟

```shell
$ docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.7-alpine
c3hrsui7svtmzpxxars48tmtx
```

我们在部署服务指定滚动更新策略。`--update-delay` 表示更新服务对应的任务或一组任务之间的时间间隔。时间间隔用数字和时间单位表示，m 表示分，h 表示时，所以 10m30s 表示 10 分 30 秒的延时。

默认情况下，调度器一次更新一个任务。你可以使用 `--update-parallelism` 标志配置调度器每次同时更新的最大任务数量。

默认情况下，如果更新某个任务返回了`RUNNING`状态，调度器会转去更新另一个任务，直到所有任务都更新完成。如果在更新某个任务的任意时刻返回了`FAILED`，调度器暂停更新。我们可以在执行 `docker service create` 命令和 `docker service update` 命令时使用 `--update-failure-action` 标志来覆盖这种默认行为。

### 查看`redis`服务

```shell
$ docker service inspect --pretty redis
ID:		c3hrsui7svtmzpxxars48tmtx
Name:		redis
Mode:		Replicated
 Replicas:	3
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
ContainerSpec:
 Image:		redis:3.0.7-alpine
Resources:
```

### 更新`redis`服务使用的容器镜像

`swarm`管理节点会根据`UpdateConfig`策略来更新节点

```shell
$ docker service update --image redis:3.2.5-alpine redis 
redis
```

调度器根据下面默认的策略来应用滚动更新：

-   停止第一个任务。
-   为停止的任务应用更新。
-   为更新的任务启动容器。
-   如果更新任务时返回`RUNNING`，等待一个指定的延时后停止下一个任务。
-   如果，在更新的任意时刻，某个任务返回`FAILED`，暂停更新。

### 查看新镜像的状态

```shell
$ docker service inspect --pretty redis
ID:		c3hrsui7svtmzpxxars48tmtx
Name:		redis
Mode:		Replicated
 Replicas:	3
Update status:
 State:		updating
 Started:	29 seconds ago
 Message:	update in progress
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
ContainerSpec:
 Image:		redis:3.2.5-alpine
Resources:
```

可以看到 redis 服务的镜像已经变成了`redis:3.2.5-alpine`。不过状态还在更新中。过一会再查看：

```shell
$ docker service inspect --pretty redis
ID:		c3hrsui7svtmzpxxars48tmtx
Name:		redis
Mode:		Replicated
 Replicas:	3
Update status:
 State:		completed
 Started:	6 minutes ago
 Completed:	3 minutes ago
 Message:	update completed
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
ContainerSpec:
 Image:		redis:3.2.5-alpine
Resources:
```

可以看到更新完成。

### 查看滚动更新后服务的状态

```shell
$ docker service ps redis 
ID                         NAME         IMAGE               NODE       DESIRED STATE  CURRENT STATE                ERROR
9b89vrug0wz1nw28840rv5r57  redis.1      redis:3.2.5-alpine  manager01  Running        Running about a minute ago   
8bqo8nni0pi2o3c3nog5r0u7v   \_ redis.1  redis:3.0.7-alpine  manager01  Shutdown       Shutdown about a minute ago  
3x6m9vmrw20kaittw28slwtmj  redis.2      redis:3.2.5-alpine  worker02   Running        Running about a minute ago   
cb4b3iowzhmi8bk2izok4t2e6   \_ redis.2  redis:3.0.7-alpine  worker02   Shutdown       Shutdown about a minute ago  
cozd08m25sg3eip9cimsc6bc7  redis.3      redis:3.2.5-alpine  manager01  Running        Running 42 seconds ago       
eb8f2m36c00h3kxpjfhctre32   \_ redis.3  redis:3.0.7-alpine  worker01   Shutdown       Shutdown 56 seconds ago
```

可以看到有三个镜像为`redis:3.0.7-alpine`的任务状态为`Shutdown`，三个镜像为`redis:3.2.5-alpine`的任务状态为`Running`。说明滚动更新已完成。

再看看容器信息

```shell
$ docker ps -a
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                     PORTS               NAMES
7ff070b012cb        redis:3.2.5-alpine   "docker-entrypoint.sh"   9 minutes ago       Up 9 minutes               6379/tcp            redis.1.0bjgpgzrenco91wkmp6tgd2pj
439020be1806        redis:3.0.7-alpine   "docker-entrypoint.sh"   30 minutes ago      Exited (0) 9 minutes ago                       redis.3.26t3etvukx440tht3kw5wib77
```

可以看出更新后，`swarm`并没有删除旧的容器。

## 下线某个节点

在前面的步骤中，所有的节点都处于运行状态且可用性为`ACTIVE`。swarm 管理器可以将任务分配给任何可用性为 `ACTIVE` 的节点，所以到目前为止，所有节点都可以接收任务。

有时候，比如到了计划的维护时间，你需要将节点的可用性设为`DRAIN`。可用性为`DRAIN`的节点不会从 swarm 接收任何新任务。同时，管理器将停止运行在该节点上的任务，并在另外可用性为 `ACTIVE` 的节点上启动相应的任务副本。

### 确认所有节点都是活跃可用的

```shell
$ docker node ls
ID                           HOSTNAME   STATUS  AVAILABILITY  MANAGER STATUS
4rkvaeauhftolat98u5ty01iz    worker02   Ready   Active        
7ik7wqhe5wcag8k5tp816c7ck *  manager01  Ready   Active        Leader
cashcvy4qzcq3lvtnix4trqu7    worker01   Ready   Active  
```

### 启动前面提到的 `redis` 服务（如果该服务没有运行）

我这边 `redis` 服务一直处于运行状态，所以不需要重新启动。如果你的没有在运行，那就重新启动它吧：

```shell
$ docker service create --replicas 3 --name redis --update-delay 10s redis:3.2.5-alpine
```

### 查看任务在各节点的分配情况

```shell
$ docker service ps redis 
ID                         NAME         IMAGE               NODE       DESIRED STATE  CURRENT STATE           ERROR
9b89vrug0wz1nw28840rv5r57  redis.1      redis:3.2.5-alpine  manager01  Running        Running 4 minutes ago   
8bqo8nni0pi2o3c3nog5r0u7v   \_ redis.1  redis:3.0.7-alpine  manager01  Shutdown       Shutdown 4 minutes ago  
3x6m9vmrw20kaittw28slwtmj  redis.2      redis:3.2.5-alpine  worker02   Running        Running 3 minutes ago   
cb4b3iowzhmi8bk2izok4t2e6   \_ redis.2  redis:3.0.7-alpine  worker02   Shutdown       Shutdown 4 minutes ago  
cozd08m25sg3eip9cimsc6bc7  redis.3      redis:3.2.5-alpine  manager01  Running        Running 3 minutes ago   
eb8f2m36c00h3kxpjfhctre32   \_ redis.3  redis:3.0.7-alpine  worker01   Shutdown       Shutdown 3 minutes ago  
```

可以看到有两个任务在管理节点上运行，有一个任务在其中一个工作节点上运行。这种分配不是固定的，你的实验结果很可能和我的不同。

>   注：上输出信息中，状态为 `Shtudown` 的任务时前面滚动更新时遗留下来任务。不要疑惑。

### 下线一个被分配了任务的节点

将分配了任务的工作节点 `worker02` 下线：

```shell
$ docker node update --availability drain worker02
worker02
```

### 查看被下线节点的详细信息

```shell
$ docker node ls
ID                           HOSTNAME   STATUS  AVAILABILITY  MANAGER STATUS
4rkvaeauhftolat98u5ty01iz    worker02   Ready   Drain         
7ik7wqhe5wcag8k5tp816c7ck *  manager01  Ready   Active        Leader
cashcvy4qzcq3lvtnix4trqu7    worker01   Ready   Active     
$ docker node inspect --pretty worker02
ID:			4rkvaeauhftolat98u5ty01iz
Hostname:		worker02
Joined at:		2016-12-22 14:51:02.806948804 +0000 utc
Status:
 State:			Ready
 Availability:		Drain
Platform:
 Operating System:	linux
 Architecture:		x86_64
Resources:
 CPUs:			2
 Memory:		1.937 GiB
Plugins:
  Network:		bridge, host, null, overlay
  Volume:		local
Engine Version:		1.12.3
```

可以看到该节点的状态为`Ready`，但可用性为`Drain`。

### 再次查看任务的分配情况

```shell
$ docker service ps redis 
ID                         NAME         IMAGE               NODE       DESIRED STATE  CURRENT STATE                ERROR
9b89vrug0wz1nw28840rv5r57  redis.1      redis:3.2.5-alpine  manager01  Running        Running 8 minutes ago        
8bqo8nni0pi2o3c3nog5r0u7v   \_ redis.1  redis:3.0.7-alpine  manager01  Shutdown       Shutdown 9 minutes ago       
1lwslgvlq631v14urjuo2vge2  redis.2      redis:3.2.5-alpine  worker01   Running        Running about a minute ago   
3x6m9vmrw20kaittw28slwtmj   \_ redis.2  redis:3.2.5-alpine  worker02   Shutdown       Shutdown about a minute ago  
cb4b3iowzhmi8bk2izok4t2e6   \_ redis.2  redis:3.0.7-alpine  worker02   Shutdown       Shutdown 8 minutes ago       
cozd08m25sg3eip9cimsc6bc7  redis.3      redis:3.2.5-alpine  manager01  Running        Running 8 minutes ago        
eb8f2m36c00h3kxpjfhctre32   \_ redis.3  redis:3.0.7-alpine  worker01   Shutdown       Shutdown 8 minutes ago 
```

可以看到，swarm 管理器停止了`workd02`工作节点上的任务，并在 `work01` 上创建了一个新任务。

### 再次将被下线的节点重置为活动状态

```shell
$ docker node update --availability active worker02
worker02
```

### 确认该节点的新状态

```shell
$ docker node ls
ID                           HOSTNAME   STATUS  AVAILABILITY  MANAGER STATUS
4rkvaeauhftolat98u5ty01iz    worker02   Ready   Active        
7ik7wqhe5wcag8k5tp816c7ck *  manager01  Ready   Active        Leader
cashcvy4qzcq3lvtnix4trqu7    worker01   Ready   Active

$ docker node inspect --pretty worker02
ID:			4rkvaeauhftolat98u5ty01iz
Hostname:		worker02
Joined at:		2016-12-22 14:51:02.806948804 +0000 utc
Status:
 State:			Ready
 Availability:		Active
...
```

说明现在该节点又可以重新接收任务了。

再看看 `worker02` 节点上是否被分配了任务：

```shell
$ docker service ps -f desired-state=running redis 
ID                         NAME     IMAGE               NODE       DESIRED STATE  CURRENT STATE           ERROR
9b89vrug0wz1nw28840rv5r57  redis.1  redis:3.2.5-alpine  manager01  Running        Running 20 minutes ago  
1lwslgvlq631v14urjuo2vge2  redis.2  redis:3.2.5-alpine  worker01   Running        Running 13 minutes ago  
cozd08m25sg3eip9cimsc6bc7  redis.3  redis:3.2.5-alpine  manager01  Running        Running 20 minutes ago  
```

`desired-state` 表示只列出处于活动状态的任务。说明 `worker02` 虽然可用，但没被分配任务。

一个可用性为`Active`的节点在以下情况下可以接收到新任务：

-   当一个服务在伸缩规模时
-   滚动更新时
-   当你把其他某个节点的可用性设为 `Drain` 时
-   当某个任务在另外某个 `Active` 节点上启动失败时


我们扩大一下服务的规模，看是否有新任务被分配到 `worker02` 上：

```shell
$ docker service scale redis=5
redis scaled to 5
```

在查看一个任务分配情况：

```shell
$ docker service ps -f desired-state=running  redis 
ID                         NAME     IMAGE               NODE       DESIRED STATE  CURRENT STATE               ERROR
9b89vrug0wz1nw28840rv5r57  redis.1  redis:3.2.5-alpine  manager01  Running        Running 26 minutes ago      
1lwslgvlq631v14urjuo2vge2  redis.2  redis:3.2.5-alpine  worker01   Running        Running 19 minutes ago      
cozd08m25sg3eip9cimsc6bc7  redis.3  redis:3.2.5-alpine  manager01  Running        Running 25 minutes ago      
0s11xwnvusv0vorgnjielg7ef  redis.4  redis:3.2.5-alpine  worker02   Running        Running about a minute ago  
035fd3l4xz6bvlz5wi4s04b9k  redis.5  redis:3.2.5-alpine  worker02   Running        Running about a minute ago 
```

可以看到，`worker02` 被新分配了两个任务。