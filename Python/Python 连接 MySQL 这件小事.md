# Python 访问 MySQL 这件小事

一般，在代码中连接并访问某个数据库服务，都需要通过一个特定于语言和数据库的驱动（或者叫连接器、binding（绑定））。MySQL 数据库几乎是国内互联网公司的标配。本来用 Python 访问数据库是件很简单的小事，你百度或 Google 就会发现好几种选择，但这些网络博客由于年久失修，会让新手走进坑里。即使不踩坑，用一段时间后，你也会产生疑惑。本文比较全面的盘点了各种 Python 访问 MySQL 的姿势，并给出最佳实践，希望能帮助让你少踩坑，或者从坑里跳出来。

## Python 2 时代的 MySQL-python\MySQLdb

MySQL-python 又叫 MySQLdb，前者是它的包名，后者是它的接口名。是 Python2 时代访问 MySQL 的主流姿势。你现在搜索都能看到很多相关的博文。但几乎都会把你带进坑里。MySQLdb 只适用于 MySQL 3.23~5.5，以及 Python 2.4~2.7，现在已经不能用了。在 Linux 平台可以顺利安装，在 Windows 平台安装比较麻烦，连官方都不愿意写教程，在 Mac OS 上虽然可以安装，但是使用时可能会出现导入包不成功的 bug。

所以现在强烈不建议你使用 MySQL-python，见到它最好绕道走。

## MySQLdb 的 Python3 修复版——mysqlclient

有人看 MySQLdb 年久失修，就 fork 了一份 MySQLdb 代码，改名为 mysqlclient 并添加了 Python 3 支持，并修复了一系列 bug。如果你在 mysqlclient 的项目主页 [mysqlclient-python](https://github.com/PyMySQL/mysqlclient-python) 点击[文档链接](https://mysqlclient.readthedocs.io/)，却发现文档标题却是 “MySQLdb User’s Guide”，千万不要惊讶，因为 mysqlclient 就是 mysqldb 的升级版。mysqlclient 继承了 MySQLdb 的缺点，在 Windows 上安装还是很可能会出现问题，如果你不得不在 Windows 下选择 mysqlclient，新手最好选择去 https://www.lfd.uci.edu/~gohlke/pythonlibs/ 下载别人编译好的 `.whl` 文件。

mysqlclient 一个好处是，提供了比较底层的接口，不过一般人用不着。所以现在也不建议使用 mysqlclient。

安装

```shell
$ pip install mysqlclient
```

连接数据库

还是不建议使用 mysqlclient

## 纯 Python 实现的 PyMySQL

PyMySQL 是纯 Python 代码实现的 Python binding。纯 Python 代码实现的包有个好处就是安装简单，任何支持 Python 的平台都能直接安装，不会出现兼容性问题。mysqlclient 和 MySQLdb 就是因为依赖了 C 库，才导致安装的时候麻烦，特别是在 Windows 上容易出现编译问题。

PyMySQL 支持 Python 2.7 和 Python 3.5 及以上版本。支持的 MySQL  5.5 及以上版本。

安装 

```shell
pip install pymysql
```

## MySQL 官方出品的 mysql-connector-python

这个强烈推荐。mysql-connector-python 由 MySQL 官方出品，有专门的维护团队，能紧跟 MySQL 更新。而且 8.0 版本由纯 Python 代码实现，在各个平台安装都很丝滑。

mysql-connector-python 、Python、MySQL 之间的版本对应关系：

| mysql-connector-python 版本 | MySQL 版本                    | Python 版本                  | 状态                     |
| --------------------------- | ----------------------------- | ---------------------------- | ------------------------ |
| 8.0                         | 8.0, 5.7, 5.6, 5.5            | 3.8, 3.7, 3.6, 3.5, 3.4, 2.7 | 一般可用                 |
| 2.2                         | 5.7, 5.6, 5.5                 | 3.5, 3.4, 2.7                | 里程碑版本，还未正式发布 |
| 2.1                         | 5.7, 5.6, 5.5                 | 3.5, 3.4, 2.7, 2.6           | 一般可用                 |
| 2.0                         | 5.7, 5.6, 5.5                 | 3.5, 3.4, 2.7, 2.6           | 一般可用，已停止更新     |
| 1.2                         | 5.7, 5.6, 5.5 (5.1, 5.0, 4.1) | 3.4, 3.3, 3.2, 3.1, 2.7, 2.6 | 一般可用，已停止更新     |

一般可用就表示可用于生产环境了。现在主流的 Python 版本已经是 3.x 了。主流的 MySQL 版本是 5.7 和 5.6。所以一般直接选 mysql-connector-python 8 就行。如果你必须得使用 Python 2.7，可以选择 mysql-connector-python  2.1 或 2.2。

安装 mysql-connector-python 8

```shell
pip install mysql-connector-python
```

验证

```shell
shell> python
>>> from distutils.sysconfig import get_python_lib
>>> print(get_python_lib())
C:\Users\xxx\AppData\Local\Programs\Python\Python38\Lib\site-packages
```

注意，8.0 之前的版本并不是纯 Python 实现，在 Windows 平台安装可能会有点麻烦。如果要安装 8.0 之前的版本，请按照[官方文档](https://dev.mysql.com/doc/connector-python/en/connector-python-installation.html) 的指示选择使用二进制包安装或者源码编译安装。

## ORM 框架 SQLAlchemy

ORM 是比数据库的语言绑定更先进的工具，直接用 PyMySQL 和 mysql-connector-python 这类 Python binding，你需要手动拼接 SQL 语句，而使用 ORM 框架，你可以只关注 Python 对象，写出的代码更具可读性，而且可以避免很多手写 SQL 的坑。我当初头铁，偏要在实际项目中手动拼接 SQL 语句，结果踩了很多坑，最后修修补补，我发现我实际上在实现一个玩具版的 ORM 框架，最后老老实实换了 SQLAlchemy。

Python 的 ORM 框架有好几个，但考虑到功能齐全，社区活跃度，更新频率，用户数，我只推荐 SQLAlchemy。另外，Djanjo 也比较流行，如果你熟悉 Djanjo，可以把 Djanjo 的 ORM 模块拆出来用。

安装：

```shell
pip install sqlalchemy
```

不过 SQLAlchemy 本身并不没有提供访问数据库的功能，也需要通过 Python binding 来访问数据。所以安装  SQLAlchemy 后，你还需要安装前面介绍的 PyMySQL 或 mysql-connector-python。

连接数据库：

SQLAlchemy + PyMySQL 

```python
engine = create_engine("mysql+pymsql://{user}:{password}@{host}:{port}/{db}?charset=utf8".format(user='chao', password='123456', host='192.168.3.131', port=3306, db='test'),echo=True)
```

SQLAlchemy + mysql-connector-python

```python
engine = create_engine("mysql+mysqlconnector://{user}:{password}@{host}:{port}/{db}?charset=utf8".format(user='chao', password='123456', host='192.168.3.131', port=3306, db='test'),echo=True)
```

SQLAlchemy + mysqlclient（或 MySQLdb）

```python
# 虽然不再建议使用 mysqlclient 或 MySQLdb，但为了完整性，还是列出来
engine = create_engine("mysql+mysqldb://{user}:{password}@{host}:{port}/{db}?charset=utf8".format(user='chao', password='123456', host='192.168.3.131', port=3306, db='test'),echo=True)
```

