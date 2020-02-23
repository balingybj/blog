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

```python

```

还是不建议使用 mysqlclient

## 纯 Python 实现的 PyMySQL





