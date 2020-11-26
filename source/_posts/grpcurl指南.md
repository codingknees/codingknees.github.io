---
title: grpcurl指南
date: 2020-11-26 17:54:07
tags: grpc,protobuf
---


基于protoreflect包实现的，gRPC服务访问，探查工具

有3种指定gRPC服务schema的方式：
<!--more-->
* 服务开启server reflection之后，直接指定服务地址

* 本地提供服务对应的proto源文件
  ```
  -proto value  //指定proto源文件，可以指定多个
  -import-path value //指定import依赖的查找路径，可以指定多个
  ```
* 本地提供服务对应的protoset文件
  ```
  -protoset value //指定protoset，可以指定多个
  ```
  protoset是描述服务信息的`google.protobuf.FileDescriptorSet`对象实例。它以二进制格式存储在protoset文件中。生成的方式如下：
  ```
  protoc --proto_path=. \
    --descriptor_set_out=myservice.protoset \
    --include_imports \
    my/custom/server/service.proto
  ```
  在protoc编译过程中已经指定了import path，并且依赖的proto也都编译到了同一个protoset，因此通过这种方式指定schema无需另外指定import path。并且，不需要每次使用都进行proto解析，性能更好。在脚本多次调用时，使用这种方式更加合适。


1. 服务访问
支持包括streaming method在内的所有类型的gRPC接口访问，支持TLS和plain-text方式访问。对于TLS方式，支持指定各种参数。
```
grpcurl [-d ''] address method
```
其中address支持
```
ip:port		//ip+端口
host:port //主机名+端口
ip:servicename //ip+知名端口
host:servicename //主机名+知名端口
-unix=true /a/path/to/the/domain/socket  //unix套接字
```
连接方式支持plaintext和TLS方式，TLS支持各种参数指定
```
-plaintext  //plaintext模式
//下面都是TLS方式的参数
-insecure   //跳过server certificate和domain verification
-authority string //指定 :authority头,和-servername参数一样，在TLS certificate时，替换server name参数。因此不能和-servername同时提供。推荐使用本参数
-servername string //指定TLS certificate时的server name参数
-cacert string //指定根证书
-cert string   //指定client certificate(public key)
-key string    //指定client private key
```
其中method必须是fully-qualified，即必须是service+method，格式可以是
```
service/method
service.method
```
此外通过-d指定请求结构数据，-d选项必须在address 和 method前面。如果是空的请求，可以不需要-d选项。 `-d @` 指定从标准输入中读取请求数据，这个选项配合`jq`命令进行管道操作有奇效。

2. 服务探查
```
grpcurl [address/proto/protoset] list [symbol]
grpcurl [address/proto/protoset] describe [symbol]
```
这里的symbol都必须是fully-qualified名字。list可以带service；describe可以带service，enum，message。
list 不带symbol时，输出所有的service；带service时，输出该service的所有method；
describe 不带symbol时，也是输出所有的service；带symbol时，输出该symbol的详细信息。

3. 其他选项
```
-H value   //指定额外的header, 格式 'name:value'。可以多个。
-reflect-header value  //指定服务探查时额外的header
-rpc-header value  //指定rpc访问时额外的header
-expand-headers   //上述3个选项中的${NAME}格式的header用环境变量进行展开，目的是避免在命令行中暴露密码等敏感信息

-connect-timeout float   //指定连接超时时间, 默认10s
-keepalive-time float      //连接空闲时，心跳请求发送间隔时间
-max-time float    //单次请求最大处理时间，可以避免grpcurl因为复杂请求或者网络状况差或者错误的stream使用方式而挂住
-max-msg-sz int    //单个rsp的最大长度，默认4,194,304(即4M))

-format string   //指定请求数据格式，可选json和text，默认是json
-format-error    //发生错误时，用-format指定的格式打印错误

-allow-unknown-fields  //用json格式解析req时，忽略未知的字段
-emit-defaults    //用json解析rsp时，忽略默认值
-msg-template    //describe子命令探查message时，打印一个message的模板，方便复制使用
-protoset-out string      //list或者describe时，把protoset写到指定的文件中
-use-reflection     //当指定-proto或者--protoset时，默认是不通过服务reflection进行探查的，通过这个选项可以同时打开本地模式和服务reflection模式

```
