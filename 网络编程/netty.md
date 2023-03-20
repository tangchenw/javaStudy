## 编解码技术

### java序列化的缺点

- **无法跨语言**。对于跨进程的服务调用，服务提供者可能会使用c++或者其它语言开发，java序列化技术是java语言的内部私有协议，其它语言并不支持。
- **序列化后的码流太大**。JDK序列化机制编码后的二进制数组大小是二进制编码的5倍左右
- **序列化性能太低**。对序列化和二进制编码分别进行性能测试，编码100万次，统计总共耗时，发现jdk耗时1862 ms；二进制编码为280 ms；该编码结果因host性能不同可能存在一定差异性，但是jdk的性能是一定比二进制编码的性能差非常多的。

## 业界主流的编解码框架

### Google的Protobuf介绍

它的数据结构是以.proto文件进行描述，通过代码生成工具可以生成对应的数据结构的pojo对象和protobuf相关的方法和属性。

特点：

- 结构化数据存储格式（XML、JSON等）；
- 高效的编解码性能。
- 语言无关、平台无关、扩展性好
- 官方支持java、c++和python三种语言。

### Facebook的Thrift

Thrift主要由5部分组成。

- 语言系统以及IDL编码器：负责由用户给定的IDL文件生成相应语言的接口代码；
- TProtocol：RPC的协议层，可以选择多种不同的对象序列化方式，如JSON和Binary。
- TTransport:RPC的传输层，同样可以选择不同的传输层实现，如socket、NIO、MemoryBuffer等
- TProcessor:作为协议层和用户提供的服务实现之间的纽带，负责调用服务实现的接口。
- TServer:聚合TProtocol、TTransport和TProcessor等对象。

### JBOSS的Marshalling

JBOSS Marshalling是一个java对象的序列化API包，修正了JDK自带的序列化包的很多问题，同时又兼容了serializable接口。



# 高级篇——Netty多协议开发和应用

## HTTP协议开发应用

