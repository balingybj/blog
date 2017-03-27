# Docker 镜像规范 v1.0.0

本文翻译自[Docker Image Specification](https://github.com/docker/docker/blob/master/image/spec/v1.md)。本文翻译时，该规范为 v1.0.0 版本。

镜像是关于 root 文件系统变更集的有序集合以及相应容器运行时的执行参数。该规范概括了这些文件系统变更集的格式以及相应参数，并描述了创建它们的方法以及在容器运行时和执行工具中如何使用它们。

## 术语

该规范使用了以下术语：

### Layer（层）

镜像由层组成。镜像层是一个一般性的术语，用来表示下面一组或多组信息：

-   层的元信息，JSON 格式。
-   文件系统在该层的变化。

前者可以使用术语 `Layer JSON` 或 `Layer Metadata`。后者可以使用术语 `Image Filesystem Changeset`(镜像文件系统变更集合) 或`Image Diff`。

### Image JSON

每个层都有一个相应的 JSON 结构的描述信息，描述了关于镜像的基本信息，比如创建日期、作者、父镜像的 ID，以及执行/运行时的配置，比如入口、默认参数、CPU/内存共享、网络、卷。

### Image Filesystem Changeset

每一层（Layer）其实就是一个信息归档。描述了该层相对于父层有哪些文件上的变更，比如在父层基础上添加了哪些文件、修改了哪些文件、或删除了哪些文件。使用基于层的或者联合文件系统（例如 AUFS），或者通过计算文件系统快照之间的差异，都可以用来表示层的概念，就好像一个连续的文件系统。

### Image ID

每个层在被创建时都会给一个 ID。该 ID 是一个 256 比特的 16 进制的串，例如，`a9561eb1b190625c9adb5a9513e72c4dedafc1cb2d4c5236c9a6957ec7dfd5a9`。Image ID 应该足够随机，以确保全局唯一。从 `/dev/urandom` 读取 32 个字节就能满足使用需求。或者，还可以对镜像内容使用加密散列函数，将得到的散列结果作为镜像的 ID。由实现者决定选择哪种。

### Image Parent（父镜像）

大多数层的元信息结构都包含一个`parent` 字段，指向该镜像层直接父镜像。一个镜像包含一个单独的 JSON 元数据文件，和相对于父镜像文件系统的变更集合。除了*Image Parent*， 通常也用术语 *Image Ancestor* 和 *Image Descendant*。

### Image Checksum（镜像校验和）

层元信息结构中包含一个关于该层文件系统变更集的 hash 值。尽管这些变更集合只是一个简单的 Tar 归档文件，两个不同的归档文件，即使里面的文件名和内容都相同，如果某个条目的最后访问或最后修改时间不同，产生的 SHA 摘要也不同。所以，镜像校验和通过 TarSum 算法产生，该算法根据文件内容和选择的头信息生成一个加密散列。该算法的具体细节参见 [TarSum 规范](‘https://github.com/docker/docker/blob/master/pkg/tarsum/tarsum_spec.md’) 。

### Tag（标签）

标签用于将用户指定的、具有描述性的名称对应到镜像 ID。镜像名称的后缀（名称中`:`后面的部分）也通常被叫作标签，尽管标签的严格意义是镜像名称。标签前缀可以接收的值由实现特定，但**应该**限制为字母数字字符 `[a-zA-Z0-9]`，标点符号 `[._-]`，以及必须不包含 `:`字符。

### Repository（仓库）

仓库指一个公共前缀（名称中`:`前的部分）下的标签集合。例如，一个镜像标签为 `my-app:3.1.4`，`my-app` 就是名称的 `Repository` 部分。仓库名能接受的值也是由实现特定的，当必须限制为字母数字 `[a-zA-Z0-9]`，和标点符号 `[.-_]`，也可能包含额外的 `/` 和 `:` 字符用来表示组织名，这样的话，只有最后的一个 `:` 字符才会被当做仓库名和标签后缀的分割点。而前面的 `:` 字符之前的字符表示组织名（某个公司，某个团队...）。

## Image JSON  描述

下面是一个镜像 JSON 文件示例：

```json
{  
    "id": "a9561eb1b190625c9adb5a9513e72c4dedafc1cb2d4c5236c9a6957ec7dfd5a9",
    "parent": "c6e3cedcda2e3982a1a6760e178355e8e65f7b80e4e5248743fa3549d284e024",
    "checksum": "tarsum.v1+sha256:e58fcf7418d2390dec8e8fb69d88c06ec07039d651fedc3aa72af9972e7d046b",
    "created": "2014-10-13T21:19:18.674353812Z",
    "author": "Alyssa P. Hacker &ltalyspdev@example.com&gt",
    "architecture": "amd64",
    "os": "linux",
    "Size": 271828,
    "config": {
        "User": "alice",
        "Memory": 2048,
        "MemorySwap": 4096,
        "CpuShares": 8,
        "ExposedPorts": {  
            "8080/tcp": {}
        },
        "Env": [  
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "FOO=docker_is_a_really",
            "BAR=great_tool_you_know"
        ],
        "Entrypoint": [
            "/bin/my-app-binary"
        ],
        "Cmd": [
            "--foreground",
            "--config",
            "/etc/my-app.d/default.cfg"
        ],
        "Volumes": {
            "/var/job-result-data": {},
            "/var/log/my-app-logs": {},
        },
        "WorkingDir": "/home/alice",
    }
}
```

### Image JSON 字段描述

#### id `string`

随机生成、256-bit、16 进制编码，唯一标识一个镜像。

#### parent `string`

父镜像 ID。如果没父镜像，该字段可以省略。可能多个镜像共用相同的祖先层。镜像的组织结构是严格意义上的树，任何层有零个或一个父层，有 0 个或多个后继层，没有环。实现者必须小心避免创建环，或在一个环中无限遍历下去。

#### create `string`

ISO-9601 格式，镜像被创建的日期和时间。

#### author `string`

给出创建并负责维护该镜像的人员或实体的名字或 email 地址。

#### architecture `string`

镜像中二进制文件在其上运行的 CPU 架构。可能的值包括：

-   i386
-   amd64
-   arm

将来可能支持更多的值，其中一些可能不会被特定的容器运行时实现支持。

#### os `string`

镜像在其上运行的操作系统名称。可能的值包括：

-   darwin
-   freebsd
-   linux

将来可能支持更多的值，其中一些可能不会被特定的容器运行时实现支持。

#### checksum `string`

该镜像层代表的文件系统变更集的检验和，即镜像校验和。

#### Size `integer`

镜像层的尺寸，单位为字节。

#### config `struct`

从该镜像运行的容器应使用的基本执行参数。该字段可能为 `null`，这种情况下，任何执行参数都必须在创建容器时指定。

##### 容器 RunConfig（运行配置）字段描述

###### User `string`

容器中的程序执行时应使用的 username 或 UID。当创建容器没指定该参数时，该值作为默认参数使用。

可接受的值为：

-   `user`
-   `uid`
-   `user:group`
-   `uid:uid`
-   `uid:group`
-   `user:gid`

如果没指定 `group`/`gid`，`user`/`uid` 默认的用户组为和补充组由容器中 `/etc/passwd` 文件提供。

###### Memory `integer`

内存限制（单位为字节）。当创建容器没指定该参数，该值作为默认参数。

###### MemorySwap `integer`

总内存使用限制（memory + swap）。设为 `-1` 则表示禁用 swap。当创建容器没指定该参数时，该值作为默认参数。

###### CpuShare `integer`

CPU 份额（相对权重 vs 其他容器）。当创建容器没指定该参数时，该值作为默认参数。

###### ExposedPorts `struct`

容器运行时暴露的端口号集合。该 JSON 结构值比较特殊，它是 go 语言中 `map[string]struct{}` 类型的直接序列化，在 JSON 中表示一个对象将它的键映射到一个空对象。这里是一个例子：

```json
"8080": {},
"53/udp": {},
"2356/tcp": {}
```

键值可以为下面几种格式：

-   `"port/tcp"`
-   `"port/udp"`
-   `"prot"`

如果不指定，默认的协议为 `tcp`。

这些值作为默认参数，和创建容器时指定的参数值合并。

###### Env `array of strings`

条目格式为 `VARNAME="var value"`。这些值作为默认值，和创建容器时指定的参数值合并。

###### Entrypoint `array of strings`

容器启动时当做命令执行的参数列表。该值为默认参数，可以被创建容器时指定的参数覆盖。

###### Cmd `array of strings`

容器 entry point 点的默认参数。该参数为默认参数，可以被创建容器时指定的参数覆盖。如果 `Entrypoint` 没有指定，`Cmd` 中的第一个条目被当做命令执行。

###### Volumes `struct`

目录集合，在该镜像上运行的容器中被创建为数据卷。该 JSON 结构值比较特殊，它是 Go 语言中 `map[string]struct{}` 类型的直接序列化，在 JSON 中表示一个对象将其键值映射到一个空对象。下面是例子：

```json
"/var/my-app-data/": {},
"/etc/some-config.d/": {},
```

###### WorkingDir `string`

入口程序（`Entrypoint` 指定）的当前工作目录。该值为默认参数，可以被创建容器时指定的参数覆盖。



Image JSON 结构的额外字段被当任务时实现特定的，特定实现预定无法解释的额外字段时应该忽略它们。

## 创建镜像文件系统变更集

下面是一个创建镜像文件系统变更集的例子。

一个镜像的根文件系统最初创建为一个空目录，目录名为镜像的 ID。这里有一个初始的空目录结构，表示 ID 为 ``c3167915dc9d`` （真正的 ID 比这长很多，这里为了简便只列出了一截）的镜像对应的变更集。（具体的实现不一定要这样命名根文件系统，但这种方式可方便地保存大量镜像层记录）：

```
c3167915dc9d/
```

然后创建下列目录和文件：

```
c3167915dc9d/
    etc/
        my-app-config
    bin/
        my-app-binary
        my-app-tools
```

`c3167915dc9d` 目录然后被打包成一个普通的 Tar 归档文件，里面的文件条目如下：

```
etc/my-app-config
bin/my-app-binary
bin/my-app-tools
```

现在发生一个变更，添加了一个配置目录在 `/etc/my-app.d` ，该目录包含一个默认配置文件。 二进制文件 `my-app-tools` 为了处理配置文件的变化也发生了变化。现在 `f60c56784b83` 目录的情况如下：

```
f60c56784b83/
    etc/
        my-app.d/
            default.cfg
    bin/
        my-app-binary
        my-app-tools
```

也就是 `/etc/my-app-config` 被删除，创建了一个新目录和文件 `/etc/my-app.d/default.cfg`。`/bin/my-app-tools` 也被替换为一个新版本。因为有父镜像，在将该目录提交为变更集之前，先跟目录树的父快照 `c3167915dc9d` 对比，查找文件和目录的添加、修改和删除情况。下列变更集被找到：

```
Added:      /etc/my-app.d/default.cfg
Modified:   /bin/my-app-tools
Deleted:    /etc/my-app-config
```

然后一个归档文件被创建，该归档文件只包含这些变更集：新添加和被修改的文件都有一个对应的条目，每个被删除的文件或目录对应一个路径相同的空文件，文件名为被删除文件的 basename 加一个 `.wh.` 前缀。文件名中有 `.wh.` 前缀的文件被称为 "whiteout" 文件。注意：这也意味着，不能创建一个镜像，它的根文件系统中包含某个文件或目录的名称以 `.wh.` 开头。`f60c56784b83` 产生的归档文件中的条目如下：

```
/etc/my-app.d/default.cfg
/bin/my-app-tools
/etc/.wh.my-app-config
```

任何镜像都有可能有数个这样的镜像文件系统变更集归档文件组成。

## 结合 Image JSON 和文件系统变更集

还有一个包含镜像所有信息的单个归档文件，包括：

-   仓库名/标签
-   所有的镜像层 JSON 文件。
-   每层的文件系统变更集对应的归档文件

例如，下面是 `library/busybox` 的完整归档文件（以树形展示）：

```
.
├── 5785b62b697b99a5af6cd5d0aabc804d5748abbb6d3d07da5d1d3795f2dcc83e
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── a7b8b41220991bfc754d7ad445ad27b7f272ab8b4a2c175b9512b97471d02a8a
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── a936027c5ca8bf8f517923169a233e391cbb38469a75de8383b5228dc2d26ceb
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── f60c56784b832dd990022afc120b8136ab3da9528094752ae13fe63a2d28dc8c
│   ├── VERSION
│   ├── json
│   └── layer.tar
└── repositories
```

在完整的镜像中有一个或多个以 ID 命名的目录，每个目录对应一层。每个目录包含以下三个文件：

-   `VERSION` - `json` 文件的语法版本。
-   `json` - 镜像层的 JSON 格式元数据。
-   `layer.tar` - 镜像层的文件系统变更集归档文件

`VERSION` 文件中内容是个简单的数字，表示 JSON 元数据的语法版本：

```
1.0
```

`repositories` 是另外一个 JSON 文件，用于描述名称/标签：

```
{  
    "busybox":{  
        "latest":"5785b62b697b99a5af6cd5d0aabc804d5748abbb6d3d07da5d1d3795f2dcc83e"
    }
}
```

该对象中每个键值代表仓库名，并映射到一个标签后缀集合。每个标签映射到标识对应镜像的 ID。

## 加载镜像文件系统变更集

拆包一个镜像打包文件中的所有层 JSON 文件以及对应的文件系统变更集，可以使用下面一系列的步骤：

1.  跟着父镜像 ID 一路找到根镜像（一个镜像没有父镜像 ID）。
2.  对于每个镜像层，从根开始往下遍历，提取每层文件系统变更集的归档文件到一个目录中，该目录将被用作容器的根文件系统。
    -   提取所有归档文件内容。
    -   再次遍历目录树，移除所有前缀为 `.wh.` 的文件及其对应的文件或目录。

## 实现

该规范诚然是一个关于一个未完全理解的问题的不完美描述。Docker 项目是实现该规范的一种尝试。我们的目标和工作将随着时间的推移而演变，但在该规范和我们的实现中，我们主要关注的是兼容性和互操作性。