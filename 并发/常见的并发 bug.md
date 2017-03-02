# 常见的并发 bug

在并发编程中，死锁是个经典的 bug。但更常见的是其他类型的 bug。本文尝试总结一下这些并发编程的常见的问题以及其解决方案。也许你会发现，本文其实是《Operating Systems：Three Easy Pieces》一书的读书笔记。

## 并发编程中的 bug 类型

《Operating System：Three Easy Pieces》将并发编程中的 bug 类型分为死锁 bug 和非死锁 bug。并且统计了四大开源软件中这两种类型的 bug 数量。统计到的四大开源软件包括：MySQL（流行的开源数据库系统），Apache（著名的 web server），Mozilla（浏览器），OpenOffice（开源的 Office 套件）。统计数据如下表：

| 软件         | 应用场景      | 非死锁 bug | 死锁 bug |
| ---------- | --------- | ------- | ------ |
| MySQL      | 数据库       | 14      | 9      |
| Apache     | Web 服务器   | 13      | 4      |
| Mozilla    | Web 浏览器   | 41      | 16     |
| OpenOffice | Office 套件 | 6       | 2      |
| 总数         |           | 74      | 31     |

可以看出，虽然死锁 bug 占一定比例，但非死锁 bug 更多。

## 非死锁 bug

非死锁 bug 占据了并发 bug 中大部分。但它们到底是什么样的 bug？怎么发生的？怎么解决？我们将分析非死锁 bug 中最主要的两种类型：原子性破坏 bug、顺序性破坏 bug。

### 原子性破坏 bug

原子性破坏 bug 是指一个线程中的某段代码在逻辑上应该为原子操作，但是执行到一半，切换到另一个线程执行，而另一个线程修改了这段代码的状态，导致原子性被破坏。这么说你可能不明白我在说什么，看下面的例子就一目了然。

```c
# 线程 1
if (thd)
{
  ...
fputs(thd->proc_info, ...);
  ...
}

# 线程 2
thd = NULL;
```

假设线程1 执行完 if 语句中的判断，正准备执行第 5 行的打印语句，访问指针变量 thd 指向的内容。突然发现线程切换，线程 1 休眠，线程 2 开始执行并将指针变量 thd 指向NULL。然后线程 1 恢复执行是就成了访问 NULL 指针指向的内容，于是内存错误，程序奔溃。

这种错误的解决方法也很简单，在 thd 前后加个锁就行了。线程 1 和 线程 2 在访问 thd 前都需要获得锁，这样可以保证 thd 的原子性访问。修改后的代码如下所示：

```c
pthread_mutex_t thd_lock = PTHREAD_MUTEX_INITIALIZER;

# 线程 1
pthread_mutex_lock(&thd_lock);
if (thd)
{
  ...
fputs(thd->proc_info, ...);
  ...
}
pthread_mutex_unlock(&thd_lock);

# 线程 2
pthread_mutex_lock(&thd_lock);
thd = NULL;
pthread_mutex_unlock(&thd_lock);
```

### 顺序性破坏 bug

不多说，直接上代码示例：

```c
# 线程 1
void init()
{
  ...
  mThread = PR_CreateThread(mMain, ...);
  ...
}

# 线程 2
void mMain(...)
{
  ...
  mState = mThread->State;
  ...
}
```

该代码本来的意思是在线程 1 中创建一个线程，并在线程 2 中访问新建线程的状态。但是如果如果线程 1 来不及创建这个新线程，线程 2 就开始执行并访问一个不不存在的线程状态。这样错误就产生了。正确的顺序应该是线程 1 先创建好线程，线程 2 才能访问新建线程的状态。我们需要某种机制来保证这个先后顺序，条件变量正适合干这事。修改后的代码如下：

```c
pthread_mutex_t mtLock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t mtCond = PTHREAD_COND_INITIALIZER;
int mtInit = 0;

# 线程 1
void init()
{
  ...
  mThread = PR_CreateThread(mMain, ...);
  // 通知线程2 新线程已经创建好
  pthread_mutex_lock(&mtLock);
  mtInit = 1;
  pthread_cond_signal(&mtCond);
  pthread_mutex_unlock(&mtLock);
  ...
}

# 线程 2
void mMain(...)
{
  ...
  // 等待新线程创建好
  pthread_mutex_lock(&mtLock);
  while (mtInit == 0)
  {
    pthread_cond_wait(&mtCond, &mtLock);
  }
  pthread_mutex_unlock(&mtLock);
  mState = mThread->State;
  ...
}
```

上面的代码用一个变量 mtInit 表示新线程是否创建完毕，用条件变量 mtCond 来通知线程 2 新线程已创建好，并用一个锁 mtLock 保证对 mtInit 的原子性访问。如果你对条件变量的这种用法不理解，请看我的另一篇关于条件变量的博文。

### 非死锁 bug 小结

我们上面举的例子都很容易理解，但不要以为实际工程中的这类 bug 有这么好找，而且这么容易修复。实际工程中的数据结构往往很复杂，加上模块的封装，这种 bug 不容易发现，而且可能涉及到一大堆数据结构，不容易修复。