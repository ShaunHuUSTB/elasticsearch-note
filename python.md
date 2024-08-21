## 类型注解
python是动态类型语言，不支持静态语言的类型检查

### 版本演进
- python 3.5引入typing模块，支持类型注解: 如List[int], Dict[str, float]
- python 3.9支持直接使用内建类型和泛型类型注解：如list[int], dict[str, float]

### 语法
- 函数参数添加类型注解 (param: type)
- 函数返回值添加类型注解 -> type:

### 分类
- 基本类型注解：int, float, str, bool
- 容器类型注解：List/list, Tuple/tuple, Dict/dict

### 作用
- 借助工具静态类型检查
- 增加代码可读性和可维护性
- 支持IDE的智能提示和自动补全

### 注意事项
类型注解不会在运行时进行类型检查或引发异常
