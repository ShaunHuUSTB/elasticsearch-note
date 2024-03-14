## Protocol Buffers
- [GitHub源码](https://github.com/protocolbuffers/protobuf)
- [官方文档](https://developers.google.com/protocol-buffers/)
  - [proto2译文](https://colobu.com/2015/01/07/Protobuf-language-guide/)

## protbuf使用
1. 定义.proto文件
   * 定义protobuf版本: syntax = "proto2";
   * 定义包名: package Foo;生成的类的所属命名空间Foo
   * 定义消息/服务
2. 生成类(.pb.h和.pb.cc)

## protobuf消息定义
1. 消息至少由一个字段组成
2. 字段类似C语言的结构
3. [字段有一定的格式][1]: 字段类型① | 数据类型② | 字段名称③ | 字段编码④ | 字段选项⑤
4. 通信双方:相同编码值的字段, 字段类型和数据类型必须相同(字段名称不需要), 才能互相识别
```
message Request {
  required string query = 1;
  optional int search_id = 2;
  repeated float values = 3;
  map<string, string> extrainfo = 4; 
}
```
[1]:https://blog.csdn.net/yxwb1253587469/article/details/78200464

### 字段类型(消息字段限定修饰符)
  1. required 表示必要字段, 未设置或未识别该字段会引发编解码异常, 导致消息丢弃, proto3已废弃
  2. optional 可选字段
  3. repeated 重复字段, 相当于数组/列表
  4. map 键值对字段
  5. reserved 保留值, 针对删除的字段

### 数据类型
1. 基本数据类型: bool, int32, uint32, int64, float, double, bytes, string
2. 枚举类型: message内/message外, 关键字enum
3. 复合数据类型

### 字段名称
  字段命名建议:下划线分割, 生成的c++头文件成员变量名及相应的成员函数名强制将大写字母转小写

### 字段编码(字段索引)
  字段编号唯一(在该消息中不可重复)

### 字段选项
可用选项的完整列表在[/google/protobuf/descriptor.proto][2]中定义
* default: 默认值, proto3已废弃
* json_name: 处理 JSON 映射字段
* packed = true: 在基本数字类型的重复字段上设置为， 则使用更紧凑的编码
* deprecated = true: 表示该字段已弃用,Java生成@Deprecated注释,C++，clang-tidy生成警告

[2]:https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto

## protbuf服务定义
想将消息类型用在RPC系统中, 需要在.proto文件中定义一个服务接口,服务中有一个方法接收请求返回响应
1. optional cc_generic_services = true; 编译器基于服务定义产生抽象服务代码
2. 定义服务
```
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```
3. 注册服务: 实例化抽象服务和方法
```
class ExampleSearchService : public SearchService {
 public:
  void Search(protobuf::RpcController* controller,
              const SearchRequest* request,
              SearchResponse* response,
              protobuf::Closure* done) {
    if (request->query() == "google") {
      response->add_result()->set_url("http://www.google.com");
    } else if (request->query() == "protocol buffers") {
      response->add_result()->set_url("http://protobuf.googlecode.com");
    }
    done->Run();
  }
};
```

### 应用protobuf的RPC框架TODO
https://github.com/protocolbuffers/protobuf/blob/main/docs/third_party.md

## 生成类TODO


## 消息API
