# Docker 镜像规范 v1.2.0

>   本文翻译自[Docker Image Specification v1.2.0](https://github.com/moby/moby/blob/master/image/spec/v1.2.md)。

镜像是一个有序的根文件系统的变更集合加上容器运行时相应的执行参数。本规范概述了这些文件系统变更的格式和对应参数，描述了如何在容器运行时和使用执行工具时创建和使用它们。

该版本的镜像规范从 Docker 1.12 开始使用。

## 术语

该规范使用了如下术语：

### Layer(层)

镜像由层组成。每一层表示一个文件系统的变更集合。层没有表示配置的元参数，例如环境变量或默认参数——这些是镜像的整体属性，而不是层的属性。

###Image JSON（镜像 JSON）

每个镜像都有一个相关的 JSON 结构，该 JSON 结构描述了镜像的基本信息，例如创建容器、作者、父镜像的 ID，以及执行/运行时配置，例如默认参数、CPU/内存份额、网络、卷。JSON 结构还引用了镜像中每一层所使用的加密散列，并提供了这些层的历史信息。该 JSON 被认为是不可变的，改变它将改变计算出的 imageID。改变它就意味着创建一个新的派生镜像，而不是改变现有镜像。

### 镜像文件系统变更集

每一层都有一个存档，该存档描述相对于父层哪些文件被添加了、哪些文件被更改了、哪些文件被删除了。通过使用一个基于层的或联合文件系统，例如 AUFS，或通过计算文件系统快照之间的差异，文件系统的变更集可以用来呈现一系列的镜像层，它们的表现就像是一个内聚的文件系统一样。

### Layer DiffID（层 DiffID）

层可以通过一个 DiffID 引用，该 DiffID 是层序列化表示后得到的加密散列。该加密散列是一个 tar 存档文件的 SHA256 摘要，用于传输层，表示为一个 256 比特的十六进制编码序列，例如`sha256:a9561eb1b190625c9adb5a9513e72c4dedafc1cb2d4c5236c9a6957ec7dfd5a9`。为了避免更改层 ID，层必须是可以被重新打包和解包的，例如通过使用 tar-split 来保存 tar 头。注意，用作镜像 ID 的数字序列是在未压缩的 tar 文件版本上得到的。

### Layer ChainID（层链 ID）

为了方便，有时候通过一个标识符引用一组层是非常有用的。该标识符被称为`ChainID`。对于只有一个层的镜像（或栈最底部的那层），`ChainID`等于层的`DiffID`。否则，`ChainID`由如下公式得到：`ChainID(layerN) = SHA256hex(ChainID(layerN-1) + " " + DiffID(layerN))`。

###ImageID（镜像 ID）

镜像 ID 是其 JSON 配置文件的 SHA256 哈希。表示为一个 256 比特的十六进制编码序列，例如        `sha256:a9561eb1b190625c9adb5a9513e72c4dedafc1cb2d4c5236c9a6957ec7dfd5a9`。因为 JSON 配置文件中包含了每层的哈序列，所以 ImageID 的这种构成方式让镜像可内容寻址。（译者注：就是 ID 关联其内容呗）

###Tag（标签）

Tag 用来关联镜像的名称（用户给与的）、描述，Tag 中字符的限定范围为`[a-zA-Z0-9_.-]`，并且首字母不能为`.`或`-`。Tag 最长为 128 个字符。

### 仓库

一个按通用前缀（`:`前名称部门）分组的 tag 集合。例如，一个名称 tag 为`my-app:3.14`镜像，`my-app`就是仓库名。仓库名由斜线分割的名称组成，前面有一个可选的 DNS 主机名。主机名必须符合标准的 DNS 规则，但不能包含`_`字符。如果仓库名中包含主机名，该仓库名后面可跟一个端口数字，格式为`:8080`。名称部分可能包含小写字母、数字、分隔符。分隔符为一个句号，一个或两个下划线，一个或多个划线。但名称部分不能以分隔符开头。

##镜像 JSON 描述

这里有一个镜像 JSON 配置文件示例：

```json
{  
    "created": "2015-10-31T22:22:56.015925234Z",
    "author": "Alyssa P. Hacker &ltalyspdev@example.com&gt",
    "architecture": "amd64",
    "os": "linux",
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
    },
    "rootfs": {
      "diff_ids": [
        "sha256:c6f988f4874bb0add23a778f753c65efe992244e148a1d2ec2a8b664fb66bbd1",
        "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef"
      ],
      "type": "layers"
    },
    "history": [
      {
        "created": "2015-10-31T22:22:54.690851953Z",
        "created_by": "/bin/sh -c #(nop) ADD file:a3bc1e842b69636f9df5256c49c5374fb4eef1e281fe3f282c65fb853ee171c5 in /"
      },
      {
        "created": "2015-10-31T22:22:55.613815829Z",
        "created_by": "/bin/sh -c #(nop) CMD [\"sh\"]",
        "empty_layer": true
      }
    ]
}
```

注意，Docker 产生的镜像 JSON 文件不包含格式化的空白字符。这里添加格式化的空白字符是为了清晰展示。

### 镜像的 JSON 字段描述

####create `string`

镜像的创建时间，ISO-8601 格式。

####author `string`

创建并维护该镜像的个人或组织的名称及邮箱。

####architecture `string`

该镜像中二进制文件构建为运行的 CPU 架构。可能的值为：

-   386
-   amd64
-   arm

将来可能会支持更多值，并且这其中的某些值可能不被某个特定的容器运行时实现支持。

####os `string`

该镜像构建运行的目标操作系统。可能的值包括：

-   darwin
-   freebsd
-   linux

将来可能会支持更多的值，并且其中某些值可能不被某个特定的容器运行时实现支持。

#### config `struct`

执行参数是从该镜像创建的容器的运行时的基础参数。这些字段可以为`null`，这样的话，任何运行参数都必须在创建容器时指定。

#####容器运行参数字段描述

###### User `string`

容器中进程执行使用的 username 或 UID。如果创建容器时未指定相应参数，该值将作为默认参数。

合法的值如下：

-   `user`
-   `uid`
-   `user:group`
-   `uid:gid`
-   `uid:group`
-   `user:gid`

如果`group/gid`未指定，默认为容器中`/etc/passwd` 中`user/uid` 对应的 group 和 supplementary groups。

###### Memory `integer`

内存限制（单位为字节）。如果创建容器时未指定对应参数，该值将作为默认参数。

###### MemorySwap `integer`

总内存限制（memory + swap）。设为`-1`表示禁用 swap。如果创建容器时未指定对应参数，该值将作为默认参数。

###### CpuShares `integer`

CPU 份额（相对权重 vs 其他容器）。如果创建容器时未指定对应参数，该值将作为默认参数。

###### ExposePorts `struct`

该容器运行时暴露出的端口集合。该 JSON 结构比较特别，它是 Go 语言中`map[string]struct{}` 类型的直接序列化，该类型表示一个对象将其键值映射到一个空对象。下面是一个示例：

```json
{
    "8080": {},
    "53/udp": {},
    "2356/tcp": {}
}
```

它的键值可能的格式如下：

-   “port/tcp”
-   "port/udp"
-   "port"

默认的协议为`"tcp"`。

这些值将作为默认值和创建容器时指定的参数合并。

###### Env `array of strings`

条目的格式`VARNAME="var value"`。这些值作为默认值和创建容器时指定的参数合并。

###### Entrypoint `array of strings`

容器启动时作为命令执行的参数列表（入口点）。该值为默认参数，创建容器时指定的 entrypoint 参数将替换该值。

###### Cmd `array of strings`

传递给容器入口点的默认参数。如果创建容器时未指定对应参数，该值将作为默认参数。如果没有指定`Entrypoint`，`Cmd`列表的第一个条目将作为可执行命令，称为入口点。

######Healthcheck`struct`

一个测试，用于判断容器是否健康。例如：

```json
{
  "Test": [
      "CMD-SHELL",
      "/usr/bin/check-health localhost"
  ],
  "Interval": 30000000000,
  "Timeout": 10000000000,
  "Retries": 3
}
```

该对象的字段如下：

-   Test `array of strings`

    用于执行检查容器是否健康的测试。

    选项如下：

    -   `[]`从基础镜像继承 healthcheck
    -   `["NONE"]`：不需要 healthcheck
    -   `["CMD", arg1, arg2, ...]`：直接执行参数
    -   `["CMD-SHELL", command]`：使用系统默认的 shell 执行命令。

    测试命令退出时必须返回一个状态值，0 表示容器健康，1 表示不健康。

-   Interval `integer`

    在探查之前需要等待的纳秒数。

-   Timeout `integer`

    在考虑挂起检查程序直接需要等待的纳秒数。

-   Retries `integer`

    经过多少次连续的失败后，判断容器不健康。

每个字段都可以省略，表示从基础镜像继承该值。

这些值作为默认值，和创建容器时指定的参数值合并。

###### Volumes `struct`

一组目录，在从该镜像运行的容器中被创建，作为数据卷。该 JSON 结构值比较特殊，因为它是 Go `map[string]struct{}`类型的直接 JSON 序列化，在 JSON 中表示一个对象将其键值映射到一个空对象。下面是一个示例：

```json
{
    "/var/my-app-data/": {},
    "/etc/some-config.d/": {},
}
```

###### WorkingDir `string`

为容器的入口点程序设置的当前工作目录。这是一个默认值，可以被创建容器时指定的工作目录替换。

###### rootfs `struct`

rootfs 引用了该镜像中层的内容地址。这使得镜像配置文件的散列依赖于文件系统的散列。rootfs 有两个子键：

-   `type` 通常被设为`layers`
-   `diff_ids`一个组层内容的散列（`DiffIDs`），排序方式为从最底层到最顶层。

下面是一个 rootfs 部分的示例：

```json
"rootfs": {
  "diff_ids": [
    "sha256:c6f988f4874bb0add23a778f753c65efe992244e148a1d2ec2a8b664fb66bbd1",
    "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef",
    "sha256:13f53e08df5a220ab6d13c58b2bf83a59cbdc2e04d0a3f041ddf4b0ba4112d49"
  ],
  "type": "layers"
}
```

###### history `struct`

一组对象，描述每个层的历史。层的排序方式为从最底层到最顶层。对象的字段如下：

-   `created`：创建时间，用 ISO-8601 格式表示的日期和时间。
-   `author`：该构建点的作者
-   `created_by`：创建该层的命令
-   `comment`：创建该层时添加的自定义消息
-   `empty_layer`：该字段是一个标记，用来表示该 history 是否创建了文件系统差异。如果设置为 true，表示该history 条目并不对应一个实际存在的层。（例如，一条类似 ENV 这样的命令并不会改变文件系统）

下面是一个 history 部分示例：

```json
"history": [
  {
    "created": "2015-10-31T22:22:54.690851953Z",
    "created_by": "/bin/sh -c #(nop) ADD file:a3bc1e842b69636f9df5256c49c5374fb4eef1e281fe3f282c65fb853ee171c5 in /"
  },
  {
    "created": "2015-10-31T22:22:55.613815829Z",
    "created_by": "/bin/sh -c #(nop) CMD [\"sh\"]",
    "empty_layer": true
  }
]
```



任何出现在 Image JSON 结构中的其他字段都被认为是实现特定的字段，可以被任何不能解释它们的具体实现忽略。

## 创建镜像文件系统变更集

下面是一个创建文件系统变更集的示例。

首先创建一个空目录表示镜像的根文件系统。下面是一个变更集的初始空目录结构，使用随机生成目录名 `c3167915dc9d`（[actual layer DiffIDs aregenerated based on the content](https://github.com/moby/moby/blob/master/image/spec/v1.2.md#id_desc)）。

```shell
c3167915dc9d/
```

然后创建文件和子目录：

```shell
c3167915dc9d/
    etc/
        my-app-config
    bin/
        my-app-binary
        my-app-tools
```

然后`c3167915dc9d`目录作为一个普通的 Tar 存档被提交，里面的条目如下：

```
etc/my-app-config
bin/my-app-binary
bin/my-app-tools
```

要在该容器镜像的文件系统上做变更，创建一个新的目录，例如`f60c56784b83`，并用父镜像的根文件系统的快照初始化它，所以该目录和`c3167915dc9d`是一样的。注意：该操作在 copy-on-write（写时复制）或联合文件系统上是非常高效的：

```
f60c56784b83/
    etc/
        my-app-config
    bin/
        my-app-binary
        my-app-tools
```

我们的示例变更是要添加一个配置文件目录`/etc/my-app.d`，它包含一个默认的配置文件。同样，二进制文件`my-app-tools`升级了，以便处理配置文件布局的变化。现在`f60c56784b83`目录变成了这样：

```
f60c56784b83/
    etc/
        my-app.d/
            default.cfg
    bin/
        my-app-binary
        my-app-tools
```

反映了`/etc/my-app-config`文件的删除，`/etc/my-app.d/default.cfg`文件和目录的创建。`/bin/my-app-tools`也被替换为一个更新的版本。在将该目录提交到变更集之前，因为它还有个父镜像，所以需要先和父镜像快照的目录结构进行对比，查找被添加、修改、删除的文件和目录。于是下列变更被找到：

```
Added:      /etc/my-app.d/default.cfg
Modified:   /bin/my-app-tools
Deleted:    /etc/my-app-config
```

然后一个 tar 存档文件被创建，该存档只包含这些更改集：新添加和修改的文件和目录在存档文件中对应一个条目，对于被删掉的文件或目录。存档中在相同位置有一个对应的条目，但文件名或目录名是原来的名称加上一个`.wh.`前缀。文件名中带`.wh.`前缀的文件被称为“空白”文件。注意：由于这个机制的原因，你不可能创建一个镜像，它的更文件系统包含一个名称以`.wh.`开头的文件或目录。为`f60c56784b83`目录生成的 tar 存档中的条目如下：

```
/etc/my-app.d/default.cfg
/bin/my-app-tools
/etc/.wh.my-app-config
```

## 组合镜像 JSON + 文件系统变更集格式

整个镜像的完整信息都包含在一个单一的存档文件中，里面包含了：

-   仓库名/标签
-   镜像的 JSON 配置文件
-   每一层的文件系统变更集对应的 tar 存档

例如，下面是`library/busybox` 内容的所有存档（以树形展示：）

```
.
├── 47bcc53f74dc94b1920f0b34f6036096526296767650f223433fe65c35f149eb.json
├── 5f29f704785248ddb9d06b90a11b5ea36c534865e9035e4022bb2e71d4ecbb9a
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── a65da33792c5187473faa80fa3e1b975acba06712852d1dea860692ccddf3198
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── manifest.json
└── repositories
```

该镜像中的每一层都对应一个目录。每个目录都以 64 个十六进制的字符命名，该名称由层信息产生。这些名称不一定是层 DiffID 或 ChainID。每个目录包含以下 3 个文件：

-   `VSERSION` - `json`文件的语法版本
-   `json` - 镜像层遗留的 JSON 元数据。现版本的镜像规范中，层不再需要 JSON 元数据，但是在[版本1](https://github.com/moby/moby/blob/master/image/spec/v1.md)中有。该文件的存在只是为了向后兼容。
-   `layer.tar` - 这一层对应的文件系统变更集的 tar 存档。

注意，这个目录布局对于向后兼容性特别重要。当前的实现使用在`mainifest.json`中指定的路径。

`VERSION` 文件中的内容仅仅标识了 JSON 元数据模式的语法版本：

```
1.0
```

`repositories` 是另一个 JSON 文件，用于描述名称/标签：

```
{  
    "busybox":{  
        "latest":"5f29f704785248ddb9d06b90a11b5ea36c534865e9035e4022bb2e71d4ecbb9a"
    }
}
```

该对象中每一个键值都是一个仓库名，映射到一组标签（tag）后缀。而每个 tag 又映射到该 tag 所代表的镜像的 ID。该文件只是为了向后兼容，现在的版本转而使用`manifest.json`。

`manifest.json` 文件提供镜像的顶层配置 JSON，并可选地提供父镜像 JSON。它包含一组元数据条目：

```json
[
  {
    "Config": "47bcc53f74dc94b1920f0b34f6036096526296767650f223433fe65c35f149eb.json",
    "RepoTags": ["busybox:latest"],
    "Layers": [
      "a65da33792c5187473faa80fa3e1b975acba06712852d1dea860692ccddf3198/layer.tar",
      "5f29f704785248ddb9d06b90a11b5ea36c534865e9035e4022bb2e71d4ecbb9a/layer.tar"
    ]
  }
]
```



该数组中每个镜像都有一个对应的条目。

`Config` 字段引用另 tar 存档中的另一个文件，该文件为该镜像的 JSON 配置文件。

`RepoTags` 字段列出指向该镜像的引用。

`Layers` 字段指向文件系统更改集存档。

另一个可选的 `Parent ` 字段为父镜像的 imageID。This parent must be part of the same `manifest.json` file.

不要将该文件和用于拉取和推送镜像的分布清单（manifest）混淆。

通常，支持该版本规范的实现将使用`manifest.json`（如果可用的话），旧的实现将继续使用遗留的 `*/json` 文件和 `repositories`。

