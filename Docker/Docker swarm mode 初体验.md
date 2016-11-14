# Docker swarm mode 初体验

本教程将介绍 Docker Engine Swarm 模式。在开始之前，你可能需要先熟悉一下关于 Docker Swarm mode 中的[几个关键概念](https://docs.docker.com/engine/swarm/key-concepts/)。

本教程将指导你完成一下任务：

-   在 Docker Engline swarm 模式下初始化一个集群
-   添加节点到 swarm
-   部署服务到 swarm
-   在一切就绪后管理 swarm

## 准备工作

在开始本教程之前，你需要准备一下几样东西：

-   三台通过网络连接的主机
-   Docker Engine 1.12 或更新版本
-   充当管理节点的主机 IP
-   主机之间端口相互开放

### 主机之间端口开放

主机之间的以下端口必须是开放。某些环境下，这些端口默认是允许的：

-   TCP 端口 2377 用于集群管理通信
-   TCP 和 UDP 端口 7946 用于节点间通信
-   TCP 和 UDP 端口 4789 用于 overlay 网络流量



## 创建 swarm

### 创建一个 swarm

```shell
~$ docker swarm init --advertise-addr 192.168.59.128
Swarm initialized: current node (9d4x1kfnqqoor4pb2dsnsonl2) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3wi7tszkolocsbc7vopv1tfx2r2h1owtqegwevdqqdk3fj195u-ejpeq0afjvfmujlvzboux9zjs \
    192.168.59.128:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

`--advertise-addr`标志配置了管理节点的 IP 地址。输出信息说明了怎样将新节点加入到 swarm 中。

### 执行`docker info`命令查看`swarm`的当前状态

```shell
~$ docker info
...
Swarm: active
 NodeID: 9d4x1kfnqqoor4pb2dsnsonl2
 Is Manager: true
 ClusterID: 1ffzpqr0mbeduc02hg885108b
 Managers: 1
 Nodes: 1
 ...
```

### 执行`docker node ls`命令查看节点信息

```shell
~$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
9d4x1kfnqqoor4pb2dsnsonl2 *  chao-v    Ready   Active        Leader
```

节点 id 后面的`*`表示你当前连接到了该节点。

## 添加节点到 swarm

### 添加两个工作节点到`swarm`

在第二台主机上，执行前面创建`swarm`时`docker swarm init`输出信息中命令创建工作节点并加入到`swarm`

```shell
~$ docker swarm join \
  --token SWMTKN-1-3wi7tszkolocsbc7vopv1tfx2r2h1owtqegwevdqqdk3fj195u-ejpeq0afjvfmujlvzboux9zjs   \   192.168.59.128:2377
This node joined a swarm as a worker.

```

输出信息表示当前节点已是`swarm`中的一个工作节点了。如果你忘记了该命令，可以在管理节点上执行`docker swarm join-token worker `查询

```shell
~$ docker swarm join-token worker 
To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3wi7tszkolocsbc7vopv1tfx2r2h1owtqegwevdqqdk3fj195u-ejpeq0afjvfmujlvzboux9zjs \
    192.168.59.128:2377
```

到第三台主机上，继续讲第三台主机加入到`swarm`中

```shell
$ docker swarm join \
     --token SWMTKN-1-3wi7tszkolocsbc7vopv1tfx2r2h1owtqegwevdqqdk3fj195u-ejpeq0afjvfmujlvzboux9zjs \    192.168.59.128:2377
This node joined a swarm as a worker.
```

### 查看所有节点的状态

在管理节点上

```shell
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
97f30o9pu20kpazecn54qn00n    chao-v    Ready   Active        
9d4x1kfnqqoor4pb2dsnsonl2 *  chao-v    Ready   Active        Leader
9mu0c1319pmytgeuak8874zz9    chao-v    Ready   Active    
```

输出信息第二行 id 后面的`*`表示当前连接到了该节点。`HOSTNAME`栏输出节点的 hostname，我三个节点的 hostname 都是`chao-v`，别问我为什么，因为我用的虚拟机，后两个节点都是拷贝自第一个节点，而且没改 hostname。`MANAGER`用于指示`swarm`中的管理节点，该栏值为`Leader`表示为管理节点，空值表示为工作节点。

## 部署一个服务到`swarm`

### 创建服务

在管理节点上

```shell
$ docker service create --replicas 1 --name helloworld alpine:3.4 ping baidu.com
d717uhdl48tntpfu7zdcgo6se
```

-   `docker service create`命令创建服务。
-   `--name`标志将服务命名为`helloworld`。
-   `--replicas`标志指定了期望状态为 1 个运行示例。
-   参数`alpline:3.4 ping baidu.com`将服务定义为一个 Alpine Linux 容器执行`ping baidu.com`命令。

### 查看服务列表

还是在管理节点上

```shell
$ docker service ls
ID            NAME        REPLICAS  IMAGE       COMMAND
d717uhdl48tn  helloworld  1/1       alpine:3.4  ping baidu.com
```

## 查看服务的详细信息

### 查看服务的详细信息

在管理节点上。

```shell
$ docker service inspect --pretty helloworld 
ID:		d717uhdl48tntpfu7zdcgo6se
Name:		helloworld
Mode:		Replicated
 Replicas:	1
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
ContainerSpec:
 Image:		alpine:3.4
 Args:		ping baidu.com
Resources:
```

参数`--pretty`表示以可读性良好的格式输出。如果想输出详细的 json 格式信息，去掉`--pretty`参数即可。

```shell
$ docker service inspect helloworld 
[
    {
        "ID": "d717uhdl48tntpfu7zdcgo6se",
        "Version": {
            "Index": 22
        },
        "CreatedAt": "2016-11-14T01:49:27.989653523Z",
        "UpdatedAt": "2016-11-14T01:49:27.989653523Z",
        "Spec": {
            "Name": "helloworld",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "alpine:3.4",
                    "Args": [
                        "ping",
                        "baidu.com"
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

还是在管理节点上

```shell
$ docker service ps helloworld 
ID                         NAME          IMAGE       NODE    DESIRED STATE  CURRENT STATE           ERROR
ezqjxoewzlv6bgoz70rk85xxp  helloworld.1  alpine:3.4  chao-v  Running        Running 14 minutes ago  
```

输出信息表明，`helloworld`服务的一个实例在`chao-v`节点上执行。但我的三个节点的主机名都是`chao-v`，无法分辨服务具体是在哪个节点上。所以我们再次运行该命令，并用参数`--no-resolve`表明输出节点的 id。

```shell
$ docker service ps --no-resolve helloworld 
ID                         NAME                         IMAGE       NODE                       DESIRED STATE  CURRENT STATE           ERROR
ezqjxoewzlv6bgoz70rk85xxp  d717uhdl48tntpfu7zdcgo6se.1  alpine:3.4  9d4x1kfnqqoor4pb2dsnsonl2  Running        Running 19 minutes ago  
```

我们看到 `NODE`栏输出了一串 id。我们再查看一下`swarm`中所有节点的 id，对比一下就知道是哪个节点了。

```shell
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
97f30o9pu20kpazecn54qn00n    chao-v    Ready   Active        
9d4x1kfnqqoor4pb2dsnsonl2 *  chao-v    Ready   Active        Leader
9mu0c1319pmytgeuak8874zz9    chao-v    Ready   Active        
```

对比可以发现，服务居然是在管理节点上运行。这是因为，默认情况下管理节点也可以像工作节点一样执行任务。

`DESIRED STATE`和`CURRENT STATE`表示服务的期望状态和当前状态，你可以对比它们，判断服务是否想期望的那样运行。这里的`Running`和`Running 14 minutes ago`说明服务运行正常。

### 在执行任务的节点上使用`docker ps`命令查看相关容器的详细信息

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1f2572bd690e        alpine:3.4          "ping baidu.com"    28 minutes ago      Up 28 minutes                           helloworld.1.ezqjxoewzlv6bgoz70rk85xxp
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

```shell
$ docker service ps --no-resolve helloworld 
ID                         NAME                         IMAGE       NODE                       DESIRED STATE  CURRENT STATE            ERROR
ezqjxoewzlv6bgoz70rk85xxp  d717uhdl48tntpfu7zdcgo6se.1  alpine:3.4  9d4x1kfnqqoor4pb2dsnsonl2  Running        Running 41 minutes ago   
76ntgdarnpyine9cgir7e5oug  d717uhdl48tntpfu7zdcgo6se.2  alpine:3.4  97f30o9pu20kpazecn54qn00n  Running        Preparing 7 seconds ago  
apetz2thnl2gos3iiapssdea8  d717uhdl48tntpfu7zdcgo6se.3  alpine:3.4  97f30o9pu20kpazecn54qn00n  Running        Preparing 7 seconds ago  
0orapmj2jt8dhrmrlgndxbxbp  d717uhdl48tntpfu7zdcgo6se.4  alpine:3.4  9d4x1kfnqqoor4pb2dsnsonl2  Running        Preparing 7 seconds ago  
298kwpykwyw8e7y6wb3rbf3dq  d717uhdl48tntpfu7zdcgo6se.5  alpine:3.4  9mu0c1319pmytgeuak8874zz9  Running        Preparing 7 seconds ago  
```

可以看到 7 秒前又创建了 4 个任务，现在一共有 5 个任务了。有两个在管理节点上执行。

### 在各节点上查看服务

首先是管理节点

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
849a91e6516a        alpine:3.4          "ping baidu.com"    2 minutes ago       Up 2 minutes                            helloworld.4.0orapmj2jt8dhrmrlgndxbxbp
1f2572bd690e        alpine:3.4          "ping baidu.com"    44 minutes ago      Up 44 minutes                           helloworld.1.ezqjxoewzlv6bgoz70rk85xxp
```

再是工作节点1

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1b0b9a5b8be9        alpine:3.4          "ping baidu.com"    3 minutes ago       Up 3 minutes                            helloworld.2.76ntgdarnpyine9cgir7e5oug
c43e64fa487e        alpine:3.4          "ping baidu.com"    3 minutes ago       Up 3 minutes                            helloworld.3.apetz2thnl2gos3iiapssdea8
```

再是工作节点2

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8f404250c1a8        alpine:3.4          "ping baidu.com"    4 minutes ago       Up 4 minutes                            helloworld.5.298kwpykwyw8e7y6wb3rbf3dq
```

可以看到：管理节点上运行了两个任务，工作节点1运行了一个任务，工作节点2运行了一个任务。

## 删除`swarm`上运行的服务

### 删除

在管理节点上

```shell
$ docker service rm helloworld 
helloworld
```

### 确认

在管理节点上

```shell
$ docker service inspect helloworld
[]
Error: no such service: helloworld
```

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
f63c8523e148        registry:latest     "/entrypoint.sh /etc/"   4 days ago          Exited (0) 47 hours ago                       registry
```

在工作节点上

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

我们在部署服务指定滚动更新策略。`--update-delay`表示更新服务对应的任务或一组任务之间的时间间隔。时间间隔用数字和时间单位表示，m 表示分，h 表示时，所以 10m30s 表示 10 分 30 秒的延时。

默认情况下，调度器一次更新一个任务。你可以使用`--update-parallelism`标志配置调度器每次同时更新的最大任务数量。

默认情况下，如果更新某个任务返回了`RUNNING`状态，调度器会转去更新另一个任务，直到所有任务都更新完成。如果在更新某个任务的任意时刻返回了`FAILED`，调度器暂停更新。我们可以在执行`docker service create`命令和`docker service update`命令时使用`--update-failure-action`标志来覆盖这种默认行为。

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
ID                         NAME         IMAGE               NODE    DESIRED STATE  CURRENT STATE           ERROR
0bjgpgzrenco91wkmp6tgd2pj  redis.1      redis:3.2.5-alpine  chao-v  Running        Running 5 minutes ago   
19iia6u70zsa1ri58f5nkvzov   \_ redis.1  redis:3.0.7-alpine  chao-v  Shutdown       Shutdown 7 minutes ago  
9a822ab1dc9fc9cr3kv0x5osi  redis.2      redis:3.2.5-alpine  chao-v  Running        Running 4 minutes ago   
641ad3zd7m7j9o6rmncpq61ih   \_ redis.2  redis:3.0.7-alpine  chao-v  Shutdown       Shutdown 4 minutes ago  
bys277rw7e9cqz3hwf62nn44b  redis.3      redis:3.2.5-alpine  chao-v  Running        Running 4 minutes ago   
26t3etvukx440tht3kw5wib77   \_ redis.3  redis:3.0.7-alpine  chao-v  Shutdown       Shutdown 4 minutes ago  
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

在前面的步骤中，所有的节点都处于运行状态且可用性为`ACTIVE`。swarm 管理器可以将任务分配给任何`ACTIVE`节点，所以到目前为止，所有节点都可以接收任务。

有时候，比如计划的维护时间，你需要将节点的可用性设为`DRAIN`。可用性为`DRAIN`的节点不会从 swarm 接收任何新任务。同时，管理器将停止运行在该节点上的任务，并在另外可用性为`ACTIVE`的节点上启动相应的任务副本。

### 确认所有节点都是活跃可用的

```shell
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
97f30o9pu20kpazecn54qn00n    chao-v    Ready   Active        
9d4x1kfnqqoor4pb2dsnsonl2 *  chao-v    Ready   Active        Leader
9mu0c1319pmytgeuak8874zz9    chao-v    Ready   Active 
```

### 启动前面提到的`redis`服务（如果该服务没有运行）

我这边`redis`服务一直处于运行状态，所以不需要重新启动。如果你的没有在运行，那就重新启动它吧：

```shell
$ docker service create --replicas 3 --name redis --update-delay 10s redis:3.2.5-alpine
```

### 查看任务在各节点的分配情况

```shell
$ docker service ps --no-resolve redis 
ID                         NAME                             IMAGE               NODE                       DESIRED STATE  CURRENT STATE         ERROR
0bjgpgzrenco91wkmp6tgd2pj  c3hrsui7svtmzpxxars48tmtx.1      redis:3.2.5-alpine  9d4x1kfnqqoor4pb2dsnsonl2  Running        Running 2 hours ago   
9a822ab1dc9fc9cr3kv0x5osi  c3hrsui7svtmzpxxars48tmtx.2      redis:3.2.5-alpine  9mu0c1319pmytgeuak8874zz9  Running        Running 2 hours ago   
bys277rw7e9cqz3hwf62nn44b  c3hrsui7svtmzpxxars48tmtx.3      redis:3.2.5-alpine  9mu0c1319pmytgeuak8874zz9  Running        Running 2 hours ago   
```

可以看到有一个任务在管理节点上运行，有两个任务在其中一个工作节点上运行。这种分配不是固定的，你的实验结果很可能和我的不同。

>   注：我把输出信息中状态为`Shutdown`的旧版本`redis`任务，对这里没影响。另外，我用`--no-resolve`参数输出节点的 id，是因为我的三个节点的 hostname 都一样，只能根据 id 区分。如果你的各节点 hostname 不同，可以不加`--no-resolve`参数，直接使用节点的 hostname 区分各节点。

### 下线一个被分配了任务的节点

根据前面的输出找到一个分配了任务的工作节点。将其下线：

```shell
$ docker node update --availability drain 9mu0c
9mu0c
```

`9mu0c`是一个工作节点的 id 前五位。在 Docker 的世界里，只要不造成误会，id 值可以用前几位代替。

### 查看被下线节点的详细信息

```shell
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
97f30o9pu20kpazecn54qn00n    chao-v    Ready   Active        
9d4x1kfnqqoor4pb2dsnsonl2 *  chao-v    Ready   Active        Leader
9mu0c1319pmytgeuak8874zz9    chao-v    Ready   Drain     
$ docker node inspect --pretty 9mu0c
ID:			9mu0c1319pmytgeuak8874zz9
Hostname:		chao-v
Joined at:		2016-11-14 01:16:35.232165074 +0000 utc
Status:
 State:			Ready
 Availability:		Drain
...
```

可以看到该节点的状态为`Ready`，但可用性为`Drain`。

### 再次查看任务的分配情况

```shell
$ docker service ps  redis 
ID                         NAME         IMAGE               NODE    DESIRED STATE  CURRENT STATE           ERROR
0bjgpgzrenco91wkmp6tgd2pj  redis.1      redis:3.2.5-alpine  chao-v  Running        Running 2 hours ago       
eq1vr0emw8guf5dmy1uy1uo6y  redis.2      redis:3.2.5-alpine  chao-v  Running        Running 9 minutes ago   
9a822ab1dc9fc9cr3kv0x5osi   \_ redis.2  redis:3.2.5-alpine  chao-v  Shutdown       Shutdown 9 minutes ago   
4xgnj54ih2ed4aaii3sf16xk7  redis.3      redis:3.2.5-alpine  chao-v  Running        Running 9 minutes ago   
bys277rw7e9cqz3hwf62nn44b   \_ redis.3  redis:3.2.5-alpine  chao-v  Shutdown       Shutdown 9 minutes ago  
```

可以看到，swarm 管理器停止了`Drain`工作节点上的两个任务，并在另外一个工作节点上两个新任务。

### 再次将被下线的节点重置为活动状态

```shell
$ docker node update --availability active 9mu0c
9mu0c
```

### 确认该节点的新状态

```shell
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
97f30o9pu20kpazecn54qn00n    chao-v    Ready   Active        
9d4x1kfnqqoor4pb2dsnsonl2 *  chao-v    Ready   Active        Leader
9mu0c1319pmytgeuak8874zz9    chao-v    Ready   Active
$ docker node inspect --pretty 9mu0c
ID:			9mu0c1319pmytgeuak8874zz9
Hostname:		chao-v
Joined at:		2016-11-14 01:16:35.232165074 +0000 utc
Status:
 State:			Ready
 Availability:		Active
...
```

说明现在该节点又可以重新接收任务了。

一个可用性为`Active`的节点在以下情况下可以接收到新任务：

-   当一个服务在伸缩规模时
-   滚动更新时
-   当你把其他某个节点的可用性设为`Drain`时
-   当某个任务在另外某个`Active`节点上启动失败时



























