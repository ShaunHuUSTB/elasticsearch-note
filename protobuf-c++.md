## Protocol Buffers
- [GitHub源码](https://github.com/protocolbuffers/protobuf)
- [官方文档](https://developers.google.com/protocol-buffers/)
  - [proto2译文](https://colobu.com/2015/01/07/Protobuf-language-guide/)

## protobuf消息定义
1. 消息至少由一个字段组成
2. 字段类似C语言的结构
3. [字段有一定的格式: 字段类型① | 数据类型② | 字段名称③ | 字段编码④ | 字段默认值⑤][1]
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
1. required 表示必要字段, 未设置或未识别该字段会引发编解码异常, 导致消息丢弃, proto3已删除
2. optional 可选字段
3. repeated 重复字段, 相当于数组/列表
4. map 键值对字段

### 数据类型
1. 基本数据类型: bool, int32, uint32, int64, float, double, bytes, string
2. 枚举类型: message内/message外, 关键字enum
3. 复合数据类型

### 字段名称
  字段命名建议:下划线分割, 生成的c++头文件成员变量名及相应的成员函数名强制将大写字母转小写

### 字段编码(字段索引)
  字段编号唯一(在该消息中不可重复)

字段选项, 处理 JSON 映射字段
