protobuf golang的版本历史问题
---

v1版
模块：github.com/golang/protobuf
仓库地址：https://github.com/golang/protobuf
protoc-gen-go包：github.com/golang/protobuf/protoc-gen-go
首次release时间：2010.4.20
依赖：从1.4.0开始，依赖v2实现

v2版
模块：google.golang.org/protobuf
仓库地址：https://github.com/protocolbuffers/protobuf-go
protoc-gen-go包：google.golang.org/protobuf/cmd/protoc-gen-go
首次release时间：2020.3.2
依赖：从go.mod看对v1有依赖，但是只在文件internal/weakdeps/weakdeps.go找到如下依赖代码:
```
import _ "github.com/golang/protobuf/proto"
```
没搞懂这种循环依赖为什么没问题？


根据[官方说明](https://blog.golang.org/protobuf-apiv2)，v2具有更好的性能，更清晰的接口以及对反射的一手支持。
v1从1.4.0开始，底层其实是对v2的封装，提供v1的接口。
v2使用不同的包名，这样兼顾了性能上的收益和使用上的正确性，可以让开发者进行平稳过渡。
但是，两个版本由于原理不同，他们各自工具生成的.pb.go只能搭配自己的版本的代码使用。

对于为何不遵循语义化版本的惯例，官方并没有直接说出来。只是说明，v1版的版本号一定不会升到1.20.0，所以可以区分v1和v2。但是又声明会对v1版提供无限期的支持。
这里感觉就有点矛盾了？无限期支持，怎么保证不升到1.20.0？

参考文档
v1：
https://blog.golang.org/protobuf
https://github.com/golang/protobuf/blob/master/go.mod

v2：
https://blog.golang.org/protobuf-apiv2
https://github.com/protocolbuffers/protobuf-go/blob/master/go.mod



