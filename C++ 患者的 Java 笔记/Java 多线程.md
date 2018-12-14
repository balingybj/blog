# Java 多线程

## 创建线程的两种方式

### 第一种方式：实现 Runnable 接口

这种创建线程的方式分为以下几步：

1. 创建一个类实现 Runnable 接口，并覆盖这个接口的 run 方法。将线程要执行的任务放到 run 方法中。
2. 用上一步创建的 Runnable 类创建一个对象，并用该对象作为参数创建一个 Thread 类对象。
3. 启动线程：调用上一步创建的 Thread 类对象的 start 方法。

### 第二种方式：继承 Thread 类

1. 继承 Thread 类，覆盖其 run 方法。将线程要执行的任务放到 run 方法中。
2. 启动线程，用上一步得到的 Thread 子类创建一个对象，调用其 start 方法。

这种方式和实现 Runnable 方法差不多，内部的机制也一样。实际上查看 Thread 类的源码就会发现 Thread 类也实现了 Runnable 接口。不过一般第一种方式用的比较多。

### 代码实例

两种创建线程的代码实例如下。

```java
public class Main {

    public static void main(String[] args) {
        MyThread thread1 = new MyThread();
        MyThread thread2 = new MyThread();

        thread1.start();
        thread2.start();

        Thread thread3 = new Thread(new RunObject(), "thread3");
        Thread thread4 = new Thread(new RunObject(), "thread4");

        thread3.start();
        thread4.start();

    }
}

class MyThread extends Thread {
    private static int threadID = 0;

    public MyThread() {
        super("ID:" + (++threadID));
    }

    public void run() {
        System.out.println(this.getName());
    }
}

class RunObject implements Runnable {
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
```

输出：

```
ID:1
ID:2
thread3
thread4
```

## 线程的 start 方法和 run 方法的区别

Thread.start 用于启动线程，而 Thread.run 方法是在线程启动后在线程中执行的方法。

如果创建好线程对象中直接调用 run 方法，只是普通的调用 run 方法而已，不会创建新的线程，run 方法中的代码将在当前线程中执行。


