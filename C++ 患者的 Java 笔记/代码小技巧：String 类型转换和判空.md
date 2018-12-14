# 代码小技巧：String 类型转换和判空

写了一个多月 Java 代码，发现代码中总是需要对 String 和 int 类型的参数进行相互转换，对 String 变量进行判空（判断 String 类型变量是否为空指针，字符串里面的内容是不是都是空白字符），类型转换失败后设置默认值。这些操作在很多业务代码中都存在，尽管这些代码自己写也不难，但是每次都自己写也太麻烦了，自己写成库又不太通用，幸好有一些现成通用的库可以来帮我们完成这些琐碎的操作。

## String to int

### 使用 Java 自带的 Integer.parseInt

Java 自身提高的 Integer 类提供了 Sting 转 int 的功能，例如下面将 “123” 转成 123：

```java
int a = Integer.parseInt("123");
System.out.println(a);
```

输出：

```
123
```

对于这种简单的任务还是可以满足的，但是如果转换时传入的参数为空指针、空串、或者非数字字符串呢？

下面试试传入一个空指针：

```java
int a = Integer.parseInt(null);
System.out.println(a);
```

输出：

```
Exception in thread "main" java.lang.NumberFormatException: null
	at java.lang.Integer.parseInt(Integer.java:454)
	at java.lang.Integer.parseInt(Integer.java:527)
	at com.dianping.App.main(App.java:10)
```

可见会抛出一个 java.lang.NumberFormatException。

再试一下空串：

```java
int a = Integer.parseInt("");
System.out.println(a);
```

输出：

```
Exception in thread "main" java.lang.NumberFormatException: For input string: ""
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:504)
	at java.lang.Integer.parseInt(Integer.java:527)
	at com.dianping.App.main(App.java:10)
```

再试试非数字字符串：

```java
int a = Integer.parseInt("");
System.out.println(a);
```

输出:

```
Exception in thread "main" java.lang.NumberFormatException: For input string: "hello"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:492)
	at java.lang.Integer.parseInt(Integer.java:527)
	at com.dianping.App.main(App.java:10)
```

可见，Integer.parseInt 对于传入的非法参数，比如空指针、空串、非数字字符都会抛出异常。如果想在代码中正确使用 Integer.parseInt，你就必须得提前检查参数，或者捕获异常再处理，这样很不方便。

可见使用 Java 语言自带的功能进行 String to int 的转换很麻烦，有时候在实际代码中我们只想简单地在不能转换的时候返回一个默认值，让我们知道失败了或者进行默认处理就行。这个时候就可以借助额外的库，比如 Apache Commons Lang。

### 使用 Apache Commons Lang

Apache Commons Lang 提供了很多增强 Java 语言库的功能。我可以使用它提供的 NumberUtils 来简化我们的 Sting to int 代码。使用 Apache Commons Lang 后我们的代码如下：

```java
int a = NumberUtils.toInt("hello", -1); // 第二个参数表示转换失败时返回的默认值
System.out.println(a);
```

输出：

```
-1
```

简单，无需参数检查，无需异常处理。返回的默认值可以设置为实际业务中参数正确时不可能返回的值，比如订单 id 不可能为 0，你就可以将默认值设为 0，这样失败后就可以根据返回的 0 值知道传参出了问题。

Apache Commots Lang 还有更多好用的东西，参加其[官网](https://commons.apache.org/proper/commons-lang/)。

## Object to String

Object to String 的情况比较简单，毕竟所有变量都可以转成字符串，除了空指针，Java 提供的 String.valuof 、Integer.toString 方法可以用来做这种转换，或者干脆使用 `“” + 123` 这拼接字符串的方式。但是如果传入的参数是空指针呢？下面是测试代码。

```java
String s3 = String.valueOf(null);
System.out.println(s3);
```

输出：

```
Exception in thread "main" java.lang.NullPointerException
	at java.lang.String.<init>(String.java:168)
	at java.lang.String.valueOf(String.java:2863)
	at com.dianping.App.main(App.java:22)
```

上面的代码中没有测试 Integer.toString，因为 Integer.toString 要求的参数为基本类型 int，所以传入空指针在编译阶段就会报错。





## 判空处理

