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
2. 字段类似C++的类
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
3. 复合数据类型: 将其他消息类型用作数据类型

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
1. 生成的消息类都继承自protobuf的公共类PROTOBUF_NAMESPACE_ID::Message

## 消息API

### 消息字段(成员函数)API
```
message User {
}
message Request {
  required string query = 1;
  optional int search_id = 2;
  repeated float features = 3;
  map<string, string> extrainfo = 4;
  repeated User users = 5;
}
```
1. required/optional字段
  * Get类方法
    - bool has_query() const; // 查询该字段是否设置
    - const string& query() const; // 获取字段值
  * Set类方法
    - void clear(); // 重置字段为空
    - void set_query(const T value); // 设置字段值
    - string* mutable_query(); // 只针对string或消息字段获取指针, 可以修改字段值
    
2. repeated 字段
  * Get类方法(基本数据类型和消息字段类型一致)
    - int features_size() const; // 获取数组大小
    - float features(int index) const; // 根据索引获取数组元素
    - int users_size() const;
    - const User& users(int index) const; // 根据索引获取数组元素引用
  * Set类方法
    * 基本数据类型
      - void clear_features(); // 清空数组
      - void set_features(int index, float value); // 根据索引设置数组元素
      - void add_features(float value); // 数组末尾添加元素
    * 消息字段类型
      - void clear_users(); // 清空数组
      - User* mutable_users(int index); // 根据索引获取数组元素指针, 可以设置元素值, 等效于set方法
      - User* add_features(); // 数组末尾添加元素
  * 获取容器对象(基本数据类型和消息字段类型一致)
    - RepeatedPtrField<float>* mutable_features(); // 获取数组指针, 可修改容器对象本身
    - const RepeatedPtrField<float>& features() const; //获取数组引用, 不可修改容器对象本身
  * [遍历数组](https://blog.csdn.net/liuxiao723846/article/details/105564742)
    - 索引循环
    ```
    for (int i = 0; i < request.users_size(); i++) {
      const auto& user = request.users(i);
      ...
    };
    ```
    - 迭代器循环
    ```
    for (auto iter = request.users().begin(); iter != request.users().end(); iter++) {
      const auto& user = *iter;
      ...
    }
    ```
    - 范围循环
    ```
    for (const auto& user : request.users()) {
      ...
    };
    ```

3. map字段
  * size, clear
    - int extrainfo_size() const;
    - void clear_extrainfo();
  * 获取容器对象
    - Map<string, string>* mutable_extrainfo(); // 可修改容器对象本身
    - const Map<string, string>& extrainfo() const; // 不可修改容器对象本身
  * 添加数据
    - (*request.mutable_extrainfo())[my_key] = my_value;
  * 遍历map
    - 迭代器循环
    ```
    for (auto iter = request.extrainfo().begin(); iter != request.extrainfo().end(); iter++) {
      const auto& key = iter->first;
      const auto& value = iter->second;
      ...
    }
    ```
    - 范围循环
    ```
    for (const auto& elem : request.extrainfo()) {
      const auto& key = elem.first;
      const auto& value = elem.second;
      ...
    };
    ```
  * 与std::map转换(深拷贝)
  ```
  std::map<string, string> std_extrainfo(request.extrainfo().begin(), request.extrainfo().end());
  google::protobuf::Map<string, string> extrainfo(std_extrainfo.begin(), std_extrainfo.end());
  ```
### 容器类API

1. [RepeatedPtrField等效于std::vector](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.repeated_field/)
  * 拷贝(Copy)
  ```
  RepeatedField(const RepeatedField & other);
  RepeatedField& operator=(const RepeatedField & other);
  ```  
  * 迭代器(Iterators)
  ```
  iterator begin();
  iterator end();
  ```
  * 容量(Capacity)
  ```
  int size() const;
  bool empty() const;
  ```
  * 元素访问(Element access)
  ```
  const T& Get(int index) const;
  T*	Mutable(int index);
  T& operator[](int index);
  const T& at(int index) const;
  T& at(const Key& key);
  ```
  * 修改器(Modifiers)
  ```
  void Set(int index, const T& value);
  void Add(const Element & value);
  Element*	Add();
  void  Add(Iter begin, Iter end);
  void RemoveLast();
  void Clear();
  ```

2. [MAP等效于std::unordered_map](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.map/)
  * 拷贝(Copy)
  ```
  Map(const Map& other);
  Map& operator=(const Map& other);
  ```  
  * 迭代器(Iterators)
  ```
  iterator begin();
  iterator end();
  ```
  * 容量(Capacity)
  ```
  int size() const;
  bool empty() const;
  ```
  * 元素访问(Element access)
  ```
  T& operator[](const Key& key);
  const T& at(const Key& key) const;
  T& at(const Key& key);
  ```
  * 修改器(Modifiers)
  ```
  pair<iterator, bool> insert(const value_type& value);
  template<class InputIt>
  void insert(InputIt first, InputIt last);
  size_type erase(const Key& Key);
  iterator erase(const_iterator pos);
  iterator erase(const_iterator first, const_iterator last);
  void clear();
  ```
  * 查找
  ```
  bool contains(const Key& key) const;
  int count(const Key& key) const;
  const_iterator find(const Key& key) const;
  iterator find(const Key& key);
  ```

### [标准消息类Message](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message/)
  * 构造(Copy)
  ```
  Message(const Message& other);
  Message& operator=(const Message& other);
  Message* Message::New();
  ```  
  * 常用(Basic)
  ```
  void CopyFrom(const Message & from); // 深拷贝
  void Swap(Message* other);
  void MergeFrom(const Message & from);
  bool IsInitialize() const; //检查是否全部required字段都被set
  void Clear(); // 所有项复位到空
  int ByteSize() const; // 消息字节大小
  ```  
  * 调试(Debug)
  ```
  string DebugString() const; //将消息内容以可读的方式输出
  string ShortDebugString() const; //功能类似于，DebugString(),输出时会有较少的空白
  string Utf8DebugString() const; //Like DebugString(), but do not escape UTF-8 byte sequences.
  void PrintDebugString() const;  //Convenience function useful in GDB. Prints DebugString() to stdout.
  ```  
  * 解析和序列化(Parsing and Serialization)
  ```
  bool SerializeToString(string* output) const; // 序列化消息并存储给定字符串中二进制的字节
  bool ParseFromString(const string& data); // 从给定字符串中解析消息
  bool SerializeToArray(void * data, int size) const  //将消息序列化至数组
  bool ParseFromArray(const void * data, int size)    //从数组解析消息
  bool SerializeToOstream(ostream* output) const; //将消息写入到给定的C++ ostream中
  bool ParseFromIstream(istream* input); //从给定的C++ istream解析消息
  ```

  4. [IO](https://protobuf.dev/reference/cpp/api-docs/#google.protobuf.io) 用于I/O的辅助类
  
  5. [Utility](https://protobuf.dev/reference/cpp/api-docs/#google.protobuf.util)实用程序类
      * [google/protobuf/util/json_util.h][3] 用于在protobuf二进制格式和proto3 JSON格式之间转换, 性能一般
        ```
        struct JsonPrintOptions {
          bool add_whitespace = false;
          bool always_print_fields_with_no_presence = false;
          bool always_print_enums_as_ints = false;
          bool preserve_proto_field_names = false;
          bool unquote_int64_if_possible = false;
        };
    
         util::Status util::MessageToJsonString(const Message& message, std::string* output, const JsonPrintOptions& options);
    
         struct JsonPraseOptions{
            bool ignore_unknown_fields = false;
            bool case_insensitive_enum_parsing = false;
         };
         util::Status util::JsonStringToMessage(StringPiece input, Message* message, const JsonParseOptions& options);
         ```
  
  [3]: https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.json_util
    

