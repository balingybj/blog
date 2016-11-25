#  Registry 配置



Registry 使用 YAML 格式的配置文件。但 Registry 也提供了开箱即用的完整的默认值，在部署到生产环境之前，你最好检查一下这些默认值。

## 覆盖特定的配置选项

从官方镜像运行 Registry 时，你可以使用环境变量来指定特定的选项：在`docker run`命令中使用`-e`参数，或者在 Dockerfile 长使用`ENV`指令。

覆盖特定配置的环境变量命名规则为`REGISTRY_variable`，其中`variable`是配置选项的名称，`_`表示缩进级别。例如，你可以设置`filesystem`存储后端的`rootdirectory`：

```
storage:
  filesystem:
    rootdirectory: /var/lib/registry
```

要覆盖该配置选项，创建如下一个环境变量：

```
REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/somewhere
```

该变量就将配置中的`/var/lib/registry`覆盖为`/somewhere`。注意`_`和配置缩进对应关系。

## 覆盖整个配置文件

如果默认的配置基本不适合，或者你不方便使用环境变量，你可以指定一个 YAML 配置文件，将其作为一个数据卷挂载到容器中，来覆盖默认配置。

从头开始创建一个配置文件，假设文件名为`config.yml`，然后：

```shell
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/config.yml:/etc/docker/registry/config.yml \
  registry:2.5.1
```

你参考[这个配置文件示例](https://github.com/docker/distribution/blob/master/cmd/registry/config-example.yml)。

## 配置选项列表

参考官方文档中的[配置选项列表](https://docs.docker.com/registry/configuration/#/list-of-configuration-options)，这里列表中列出了全部选项，而且有些选项是相互排斥的，目的是为了展示出所有的配置选项。

### version

```yaml
version: 0.1
```

`version`选项是必须得。用于指定配置文件的版本。一般放在最顶层，在其他选项被解析之前需要检查该版本号。

### log

`log`段用于配置日志系统的行为。日志系统的所有信息都输出到标准输出中。你可以配置日志输出的粒度和格式。

```yaml
log:
  level: debug
  formatter: text
  fields:
    service: registry
    environment: staging
```

| 参数        | 是否必须 | 描述                                       |
| :-------- | :--- | :--------------------------------------- |
| level     | 否    | 设置日志输出的级别。值可以为 error、warn，info 和 debug。默认为 info |
| formatter | 否    | 日志的输出格式。主要决定每行日志的键值如何编码。可选`test`,`json`或`logstash`。默认是`text`。 |
| fields    | 否    | 字段名到值的映射。这些字段会添加到每行日志输出的上下文。主要用于和其他日志系统混合使用时的识别。 |

### hooks

```yaml
hooks:
  - type: mail
    levels:
      - panic
    options:
      smtp:
        addr: smtp.sendhost.com:25
        username: sendername
        password: password
        insecure: true
      from: name@sendhost.com
      to:
        - name@receivehost.com
```

`hooks`段配置日志 hook 的行为。包含一个序列的处理器，例如，指定一个发送邮件的处理器。

### storage

```yaml
storage:
  filesystem:
    rootdirectory: /var/lib/registry
  # ... 太长了，不列出，参考[官方文档](https://docs.docker.com/registry/configuration/#/storage)
  inmemory:
  delete:
    enabled: false
  cache:
    blobdescriptor: inmemory
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
  redirect:
    disable: false
```

`storage`是必须的，用于指定存储后端。你必须且只能指定一个存储后端。指定多了，registry 会报错。你可以在下面的存储驱动中任选一个：

| 存储驱动         | 描述                                       |
| ------------ | ---------------------------------------- |
| `filesystem` | 使用本地磁盘来存储 registry 文件。适用于开发和某些小规模的生产环境。参考[driver's reference documetation](https://docs.docker.com/registry/storage-drivers/filesystem/) |
| `azure`      | 使用微软的 Azure Blob Storage。参考[driver's reference documentation](https://docs.docker.com/registry/storage-drivers/azure/) |
| `gcs`        | 使用 Google 的 云存储。参考[driver's reference documentation](https://docs.docker.com/registry/storage-drivers/gcs/) |
| `s3`         | 使用亚马逊的 S3。参考[driver's reference documentation](https://docs.docker.com/registry/storage-drivers/s3/) |
| `swift`      | 使用 OpenStack 的对象存储。参考[driver's reference documentation](https://docs.docker.com/registry/storage-drivers/swift/) |
| `oss`        | 使用阿里云OSS 来存储对象。参考[driver's reference documentation](https://docs.docker.com/registry/storage-drivers/oss/) |

如果是纯粹的测试，你可以使用`inmemory`存储驱动。如果你想在易失性存储器上运行 registry，在 ramdisk 上使用`filesystem`驱动即可。

#### Maintenance

目前能使用的维护功能只有上传清除和只读模式。

#### Upload Purging

上传清除是一个后台进程，定时地从上传目录中删除孤立的文件。上传清除默认是启用的。要配置上传清除，必须设置以下参数：

| 参数         | 是否必须 | 描述                         |
| ---------- | :--- | -------------------------- |
| `enable`   | 是    | true 表示启动，反之不启用            |
| `age`      | 是    | 存在时间超过该期限的将被删除。默认是168h(一周) |
| `interval` | 是    | 清除的间隔时间。默认 24h。            |
| `dryrun`   | 是    | 是否获取将要被删除的目录汇总，默认为 false。  |

注意：`age`和`interval`是字符串，包含一个数字和可选的小数部分，加单位后缀，例如：15m，2h10m，168h。

#### Read-only mode

如果`maintenance`下`readonly`段的`enabled`设置为`true`，客户端将不能上传数据到 registry。这个模式主要用于禁止写存储后端，以便执行垃圾回收。在运行垃圾回收之前，应该将`readonly`的`enabled`设置为 true，然后重启 registry。垃圾回收完成后，将`readonly`移除，并再次重启 registry ，

#### delete

`delete`段用于允许进过融合后的镜像二进制对象和清单的删除。默认为 false，可以通过下面的方式将其设置为允许：

```yaml
delete:
  enabled: true
```

#### cache

`cache`用于配置对存储后端数据访问的缓存。目前，只能缓存层的元数据。使用`blobdescriptor`字段来设置。

`blobdescriptor`字段可以设置为`redis`或`inmemory`。`redis`使用 Redis 池缓存层元数据。`inmemory`使用内存字典。

>   `blobdescriptor`就是以前的`layerinfo`。两者是等价的。后者已启用，使用前者吧。

#### redirect

`redirect`段用于管理内容后端的重定向行为。对于支持重定义的后端，重定向是默认允许的。某些特定的部署场景可能更希望由自身 registry 路由所有数据，而不是重定向到后端。对于后端和 registry 不在同地，或 registry 做了积极的缓存，这样可能更高效。

将`redirect`段下的`disable`字段值设置为`true`即可禁止重定向。

```yaml
redirect:
  disable: true
```

### auth

```ya
auth:
  silly:
    realm: silly-realm
    service: silly-service
  token:
    realm: token-realm
    service: token-service
    issuer: registry-token-issuer
    rootcertbundle: /root/certs/bundle
  htpasswd:
    realm: basic-realm
    path: /path/to/htpasswd
```

`auth`是可选的。目前可以三个身份验证服务可用，`silly`、`toker`和`htpasswd`。不过你只能选择其中一个。

#### silly

`silly`只能用于开发场景。它只是简单地检查 HTTP 请求中`Authorization`首部是否存在，而不管该首部的值。如果该首部不存在，`silly`返回一个响应，输出被拒绝访问的领域、服务和范围。

下面的值用于配置响应：

| 参数        | 是否必须 | 描述               |
| --------- | ---- | ---------------- |
| `realm`   | 是    | registry 服务认证的领域 |
| `service` | 是    | 认证的服务。           |

#### token

基于令牌的身份认证允许认证系统和 registry 解耦。这是一个良好的身份验证模式，具备高度的安全性。

| 参数               | 是否必须 | 描述                                      |
| ---------------- | ---- | --------------------------------------- |
| `realm`          | 是    | 被认证的领域                                  |
| `service`        | 是    | 被认证的服务                                  |
| `issuer`         | 是    | 令牌发行人的名字。发行人将它插入到令牌中，所以它必须和为该发行人配置的值匹配。 |
| `rootcertbundle` | 是    | 根证书包的绝对路径。这个包包含了用于签证令牌的证书 public 部分。    |

更多关于基于令牌的身份认证配置信息，参考[说明](https://docs.docker.com/registry/spec/auth/token/)。

#### htpasswd

这种身份认证方式允许用户使用[Apach htpasswd file](https://httpd.apache.org/docs/2.4/programs/htpasswd.html)配置基本的认证。只支持`bcrypt`格式的 password。其他哈希类型的条目将被忽略。htpasswd 文件在启动时加载。如果该文件无效，registry 将输出错误信息并将不启动。

>   警告：这种身份认证方案只能在使用 TLS 的前提下使用，因为基本的身份验证方案将 password 作为 http 首部发送。

| 参数      | 是否必须 | 描述                            |
| ------- | ---- | ----------------------------- |
| `realm` | 是    | 被 registry 服务认证的领域            |
| `path`  | 是    | registry 启动时 htpasswd 文件的加载路径 |

### middleware

#### cloudfront

#### redirect

### reporting

#### bugsnag

#### newrelic

### http

该段用于配置 registry 的 HTTP 服务。

| 参数             | 是否必须 | 描述                                       |
| -------------- | ---- | ---------------------------------------- |
| `addr`         | 是    | 服务监听的地址                                  |
| `net`          | 否    | 创建监听 socket 所使用的网络。`unix`或`tcp`。默认为 tcp  |
| `prefix`       | 否    | 如果服务不是在根目录运行，使用该值指定前缀。根目录是 url 中 v2 之前的部分。必须包含前后斜杠，例如`/path/`。 |
| `host`         | 否    | registry 的外部访问地址，一个完全限定的 URL。如果存在，在创建生成 URLs 将使用该值，否则，这些 url 从客户请求中生成。 |
| `secret`       | 是    |                                          |
| `relativeurls` | 否    |                                          |

#### tls

该段是可选的。该项用于给服务配置 TLS。如果你是用 Nginx 或 Apache 给registry 做服务代理，你可能更喜欢给代理服务器配置 TLS，而 registry 不适用 TLS。

| 参数            | 是否必须 | 描述                 |
| ------------- | ---- | ------------------ |
| `certificate` | 是    | x509 证书文件的绝对路径     |
| `key`         | 是    | x509 私钥文件的绝对路径     |
| `clientcas`   | 是    | 一组 x509 CA 文件的绝对路径 |

#### letsencrypt

`tls`的该子段是可选的，用于配置[Let's Encrypt](https://letsencrypt.org/how-it-works/)提供的证书。

| 参数          | 是否必须 | 描述                           |
| ----------- | ---- | ---------------------------- |
| `cachefile` | 是    | Let's Encrypt 代理缓存数据文件的绝对路径  |
| `email`     | 是    | 在 Let's Encrypt 注册的 Email 地址 |

#### debug

#### headers

### notifications

#### endpoints

### redis

#### poll

### health

#### storagedriver

#### file

#### http

#### tcp

### Proxy

### Compatibility

#### Schemal

### Example: Development configuration

### Example: Middleware configuration

