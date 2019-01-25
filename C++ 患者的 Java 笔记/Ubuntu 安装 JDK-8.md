# Ubuntu 安装 JDK-8

## 两种安装方式的选择

可以选择安装 OpenJDK 或者 Oracle JDK。这里选择 openJDK。

1. 通过 apt 命令从软件源安装。
2. 通过官网下载压缩包解压缩后安装。

第一种方式是全自动的安装。第二种方式需要自己指定目录、配置环境变量，设置系统默认的 JDK 也比较麻烦。但是比较灵活。这里选择第一种安装方式。

## 安装

更新源

```shell
$ sudo apt update
```

安装

```shell
$ sudo apt install openjdk-8-jdk
```

## 验证

```shell
$ java -version
$ javac -version
```

## 设置系统默认的 JDK 版本

如果你安装了不同版本的 JDK，则需要走这步。

### 查看系统当前的默认 JDK 版本

```shell
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```

或者

```shell
$ sudo update-java-alternatives -l
```

### 设置系统默认的 JDK 版本

先安装一个必须的东西

```shell
$ sudo apt install icedtea-8-plugin 
```

设置

```shell
$ sudo update-java-alternatives -s java-1.8.0-openjdk-amd64 
```

参考：sudo update-java-alternatives -s java-1.8.0-openjdk-amd64 

http://openjdk.java.net/projects/icedtea/