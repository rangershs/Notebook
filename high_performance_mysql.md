## High Performance MySQL
---

#### Explain
- 使用方法
    - EXPLAIN + SELECT + \G
    - 近似结果，当作优化的参考
    - 涉及子查询时，MySQL会执行子查询将结果放在临时表中，然后完成外层查询优化
    - *(MySQL5.6允许匿名临时表尽可能晚的具体化，而不是在每次优化和执行此临时表部分时再创建使用；
        因此，新版EXPLAIN带子查询的查询语句并不需要先执行子查询)*
- EXPLAIN PARTITION
    - 显示查询访问的分区
- EXPLAIN EXTENDED
    - “逆向编译”执行计划为SELECT语句
- id
    - 值越大，执行优先级越高
    - 值相同，按照顺序执行
- select_type
    - **SIMPLE** - 不包含子查询和UNION
    - **SUBQUERY** - 在SELECT列表中的子查询
    - **DERIVED** - FROM子句中的子查询，子查询的结果放在临时表中
    - **UNION** - UNION之后的SELECT被标记为UNION
    - **UNION RESULT** - 标记UNION匿名临时表中的SELECT结果
    - **DEPENDENT** - SELECT依赖于外层查询的数据
- table
    - 查询计划总是选择**左侧深度优先树**
    - FROM film INNER JOIN film_actor INNER JOIN actor ==> actor,film_actor,film
- type
    - **ALL** - 全表扫描
    - **index** - 按照<u>索引次序</u>而非行扫描全表，相比ALL全表扫描的开销小
        - 若是<u>随机访问</u>，开销会比较大
        - 若Extra=“Using index”，表示使用**覆盖索引**
    - **range** - 范围的索引扫描，如BETWEEN或WHERE+>，开销与index相当
    - **ref** - 索引查找，与值作比较
        - 返回所有匹配值的行，即会找到多个符合条件的行
        - 非唯一索引，或唯一索引的非唯一前缀
    - **eq_ref**
        - 最多只返回一条符合条件的记录
        - 主键，或唯一索引
    - **const,system**
        - MySQL优化将查询条件转换为常量
    - **NULL**
        - MySQL在优化阶段分解查询语句，执行阶段可以不用再访问表就得到结果
- key_len
    - 单位是byte，显示索引字段可能的最大长度，不是实际使用的长度
    - MySQL在**utf8**字符集下，每个字符最多为**3字节**长
- **Extra**
    - Using index - 使用覆盖索引，避免访问表
    - Using where - 存储引擎检索后再进行过滤
    - Using temporary - 使用临时表对查询结果排序
    - Using filesort - 对结果使用文件排序，在内存或磁盘上完成
    - *Range checked for each record* - 没有好用的索引，联接时重新评估

#### Chapter1

