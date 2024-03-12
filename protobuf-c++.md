## protobuf消息定义

1. 消息至少由一个字段组成
2. 字段类似C语言的结构
3. 字段有一定的格式: 字段类型① | 数据类型② | 字段名称③ | 字段编码④ | 字段默认值⑤
```
message Request {
  required string query = 1;
  optional int search_id = 2;
  repeated float values = 3;
}
```

### 字段类型(消息字段限定修饰符)
1. required 表示必要字段, 未设置或未识别该字段会引发编解码异常, 导致消息丢弃
2. optional 可选字段
3. repeated
