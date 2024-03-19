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
