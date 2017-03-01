# Protocol Buffers

## 简介

Protocol Buffers 是一种序列化结构化数据的语言无关，平台无关，可扩展机制。

```
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;
}
```

```java
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

```c++
Person john;
fstream input(argv[1],
    ios::in | ios::binary);
john.ParseFromIstream(&input);
id = john.id();
name = john.name();
email = john.email();
```

### protocol buffer 是什么？

Protocol Buffers 是 Google 的一种序列化结构化数据的语言无关的，平台无关的，可扩展机制——类似 XML，但是更小，更快，更简单。一旦你定义了数据的结果化方式，就可以使用特别生成的源代码，很容易的读取你的结果化数据，并组成各种各样的数据流，并且可以使用不同的编程语言。

### 语言支持

Protocol Buffers 目前可以生成 Java，Python，C++ 代码。新的 proto3 语言版本可以使用 Go，JavaNano，Ruby，和 C#，更多的语言正在加入支持。

### 开始入门

1.  [下载](https://github.com/google/protobuf)并安装 Protocol Buffers 编译器
2.  阅读[概述](https://developers.google.com/protocol-buffers/docs/overview)
3.  选择一个编程语言对应的[教程](https://developers.google.com/protocol-buffers/docs/tutorials)

