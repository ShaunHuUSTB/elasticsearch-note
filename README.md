# elasticsearch-note
# 全文检索
1. 数据（分类）
    1. 结构化数据：如数据库
    2. 非结构化数据(全文数据)：如email, word
2. 非结构化数据的查询方法
    1. 顺序扫描法: 依次查询每个文档, 如Linux grep命令
    2. 全文检索（Full-text Search）
       * 索引: 信息提取(非结构化->结构化):耗时, 一次构建, 多次使用
       * 检索
3. [索引分类](https://www.infoq.cn/article/3ldzr5clsxq4lwlcc97u)
    * 正排索引:以文档ID为关键字, 表中记录文档每个字的位置信息, 检索效率低
    * 倒排索引: 源于实际应用需要根据属性值来查找文档, 以词为关键字, 表中记录出现该词的文档ID和该词在文档中出现的情况
4. [创建索引](https://juejin.cn/post/6844903697642848263)
    * 文档切词
    * 语言处理
    * 索引组建立索引
        - 创建词典
        - 词典排序
        - 合并词,建立倒排链表
        - 建立索引
5. 检索索引
    * 输入查询语句
    * 语言处理(切词, NLU, NLP), 形成语法树
    * 查询得到符合语法树的文档
    * 相关性排序, 结果展示

# 全文检索引擎

* java Lucene系
    - Lucene
    - Compass
    - Solr
    - ElasticSearch
* c++ Sphinx Search系
    - Sphinx Search
    - Manticore Search

# Lucene
1. 基于倒排索引的Java开源全文检索引擎工具包
2. 官网:[https://lucene.apache.org/](https://lucene.apache.org/)

   源码:[https://github.com/apache/lucene](https://github.com/apache/lucene)

   视频教程:[https://www.bilibili.com/video/BV1c8411c7Kj?p=1](https://www.bilibili.com/video/BV1c8411c7Kj?p=1)

   文档教程:[https://xie.infoq.cn/article/1c97baa5110393fb56daa8fa5](https://xie.infoq.cn/article/1c97baa5110393fb56daa8fa5)
4. 创建索引和搜索流程![](https://static001.geekbang.org/infoq/d2/d2a974af0b9bfcdc9ef8d7dddfb5d26e.png)
5. 相关度排序

## Lucene数据模型
1. Index: 索引, 由很多Document组成
2. Document: 文档, 包含多个Field
3. Field: 域, 由Field Name和Field Value相当于key-value
    * 是否分词(tokenized)
    * 是否索引(indexed)
    * 是否存储(stored)
4. Term: 词条, 英文的单词/中文的词语

## Lucene索引
1. 采集原始数据
    * 搜索引擎: 网页爬虫
    * 电商站内搜索: 数据库访问
    * 本地文件搜索: I/O读取文件
2. 创建文档
    * 搜索引擎: Document->网页, Field: url, title, content, ...
    * 电商站内搜索: Document->商品, Field: name, category, price, ...
    * 本地文件搜索: Document->文件, Field: name, path, size, ...
3. 分析文档(分词): 对Document的Field进行分词和过滤
    * 分词器
4. 创建索引
    * 索引原理
       + [常见词典结构](https://www.cnblogs.com/sessionbest/articles/8689030.html)
       + [常见检索方式](https://dunwu.github.io/waterdrop/pages/346350/#%E7%BA%BF%E6%80%A7%E7%BB%93%E6%9E%84%E6%A3%80%E7%B4%A2)
    * 索引文件结构:倒排索引
        + 词典:
            + 存储数据结构:FST(Finite State Machines)[有限状态机](https://www.cnblogs.com/tech-lee/p/15225276.html)
               - 查询速度快, 复杂度O(len(str))
               - 内存占用小, 内存存放共享前缀(term index), 磁盘存放后缀词块(term dict)(分block保存)
            + 数值类或者多维度: [BKDTree](https://developer.aliyun.com/article/581877)
        + 倒排表(Invert)
           - 数据压缩: [Frame of reference](https://www.cnblogs.com/tech-lee/p/15225276.html)
           - 查找合并: [SkipList跳跃表](https://developer.aliyun.com/article/581877)
        + 正向文件: 行式存储
        + DocValues: [列式存储](https://www.cnblogs.com/sessionbest/articles/8689030.html), 实现返回结果按某个属性进行排序/聚合
5. 索引维护
    * 新增文档
    * 修改索引
    * 删除索引
## Lucene检索
对要搜索的信息创建Query查询对象，Lucene会根据Query查询对象生成最终的查询语法

两种方法:使用Lucene提供Query子类和使用QueryParse解析查询表达式

### 通过Query子类来创建查询对象
1. TermQuery:全匹配
2. BooleanQuery:组合查询
3. RangeQuery: 范围查询

### 通过QueryParser创建查询对象
可指定分词器, 分词器须与创建索引同一分词器
1. QueryParser
2. MultiFieldQueryParser

## 相关度排序
1. 计算词权重term weight
    * TF: 词在同一个文档中出现的频率, TF越高, term weight越大
    * DF: 词在多个文档中出现的频率, DF越高, term weight越小
2. 根据词权重打分
3. 设置boost值影响打分: boost值设置到Field上
    * 创建索引时设置boost
    * 搜索时设置boost
