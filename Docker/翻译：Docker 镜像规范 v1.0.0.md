# Docker 镜像规范 v1.0.0

镜像是关于 root 文件系统变更的有序集合以及相应容器运行时的执行参数。该规范概括了这些文件系统变更的格式以及相应参数，并描述了其在容器运行时和执行工具中的使用方法。

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

每个层在被创建时都会给一个 ID。该 ID 是一个 256 比特的 16 进制的串，例如，`a9561eb1b190625c9adb5a9513e72c4dedafc1cb2d4c5236c9a6957ec7dfd5a9`。Image ID 应该足够随机，以确保是全局唯一的。从 `/dev/urandom` 读取 32 个字节就能满足使用需求。或者，还可以对镜像内容使用加密散列函数，将得到的结果作为镜像的 ID。由实现者决定选择哪种。

### Image Parent（父镜像）

大多数的层元信息结构包含一个`parent` 字段，指向该层基于的另一层。一个镜像包含一个单独的 JSON 元数据文件，和相对于父镜像文件系统的变更集合。除了*Image Parent*， 通常也用术语 *Image Ancestor* 和 *Image Descendant*。

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



