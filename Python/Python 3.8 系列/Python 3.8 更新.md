# Python 3.8 更新

Python 3.8.0 已于 2019 年 10 月 14 日发布。Python 3.8.0 是一个大版本更新，包含很多新功能和优化。

## 3.8 在 3.7 基础上的更新

-   [PEP 572](https://www.python.org/dev/peps/pep-0572/)，赋值表达式
-   [PEP 570](https://www.python.org/dev/peps/pep-0570/)，仅位置参数
-   [PEP 587](https://www.python.org/dev/peps/pep-0587/)，Python 初始化配置（改进的嵌入）
-   [PEP 590](https://www.python.org/dev/peps/pep-0590/)，矢量调用：CPython 快速调用协议
-   [PEP 578](https://www.python.org/dev/peps/pep-0578)，运行时审计 hook
-   [PEP 574](https://www.python.org/dev/peps/pep-0574)，允许带外数据的序列化协议 5
-   注解相关：[PEP 591](https://www.python.org/dev/peps/pep-0591)(Final qualifier)，[PEP 586](https://www.python.org/dev/peps/pep-0586)(文字类型)，以及[PEP 589](https://www.python.org/dev/peps/pep-0589)(TypedDict)
-   用于编译字节码的并行文件缓存
-   调试构建与发布构建共享的 ABI
-   f-strings 支持 `=` 说明符以便调试
-   现在 `continue` 可以合法的出现在 `finally:` 语句块中
-   Windows 平台上的 `asyncio` 事件循环改为 `ProactorEventLoop`
-   macOS 平台上， `multiprocessing` 默认使用 spawn start 方法
-   `multiprocessing` 现在可以在进程间使用共享内存段，避免序列化消耗
-   `typed_ast` 合并回 CPython
-   `LOAD_GLOBAL` 速度提升 40%
-   `pickle` 现在默认使用协议 4，性能有所提高

还有很多其他更新这里没有列出，完整更新请参考官方文档中的 “What's New” 页面。

