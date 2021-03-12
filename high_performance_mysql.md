## High Performance MySQL
---

- **存储引擎和查询优化器是数据库的核心部件**
- **快速、精确、简单，几乎不可能同时满足三者，最多满足其中的两个要求**
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
- **SHOW STATUS和慢查询日志查看SQL实际执行的状态**
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
    - **index** - 按照<u>索引扫描排序</u>而非行扫描全表，相比ALL全表扫描的开销小
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
    - Using index - 使用覆盖索引，避免访问表，数据就在索引中
    - Using where - 存储引擎查找后再进行过滤
    - Using temporary - 使用临时表对查询结果排序
    - Using filesort - 对结果使用文件排序，在内存或磁盘上完成
    - *Range checked for each record* - 没有好用的索引，联接时重新评估

#### 大文件传输
- 根据场景选用
    - 网络带宽足够，直接传输未压缩的文件，节约CPU资源
    - 带宽不够，文件压缩后再传输(多数情况都会选择压缩文件)
- 相比**gzip**，**bzip2**的压缩率更高，时间却更长；**LZO**的压缩速度更快，压缩率却更低
- gzip -> scp -> gunzip
    - 多数时候选用的传输方式，每一步都需要读写磁盘
    - 串行操作，效率不高
- SSH内建的压缩特性
    - gzip -c File1 | ssh user@host gunzip -c - > File2
    - **管道直接读取-传输-写入文件**
- 非加密的传输方法
    - (S)nc -l -p 1234 | gunzip -c - > FILE2
    - (C)gzip -c - FILE1 | nc -q l host 1234
    - (tar压缩方式能保留文件名、文件权限和文件时间)
    - (S)nc -l -p 1234 | tar xvfz -
    - (C)tar cvfz - FILE1 | nc -q l host 1234
- rsync
    - 易于传输镜像文件
    - 支持断点续传
- 基准测试的结果发现rsync < scp < ssh < nc
    - 仅作参考
- **md5sum**
    - 校验文件的完整性
    - 完整扫描的代价比较高，压缩算法本身会做至少一次CRC检验

#### Chapter1
- **事务**
    - 一组原子性的SQL查询，即一个独立的工作单元，**由存储引擎实现，不在数据库服务层**
    - 要么全成功，要么全失败
    - **特性**
        - 原子性
        - 一致性 - *状态转换后保持一致*
        - 隔离性
        - 持久性
    - (需要数据库系统做更多的工作，增加了系统的开销)
    - **隔离级别**
        - 未提交读 - **脏读**，未提交的事务对其他事务是可见的
        - 提交读 - 未提交的事务对其他事务是不可见的
            - 不可重复读，同一事务中执行相同的查询可能得到不同的结果
        - 可重复读(MySQL默认隔离级别)
            - **幻读**，事务前后两次读取范围记录时，会产生幻行，读取间隔另一个事务在该范围内插入了新纪录
            - 多版本并发控制可以解决幻读问题
        - 串行化 - 每一行数据加锁，导致大量的超时和锁争用问题
    - InnoDB可以在事务执行过程中的任何时候加锁，只能在COMMIT或ROLLBACK时才会释放所有的锁
        - *LOCK TABLES/UNLOCK TABLES*在服务器层实现
    - 死锁
        - 死锁检测和死锁超时机制
        - InnoDB，将持有最少行级排他锁的事务回滚(比较简单的**死锁回滚算法**)
        - 事务型系统只有部分或完全*回滚*其中一个事务，才能打破死锁，然后重新执行回滚的事务
    - 事务型存储引擎
        - InnoDB
        - NDB Cluster
- **多版本并发控制**
    - 基于提升并发性能的目标，事务型引擎不是简单地使用行级锁，同时实现了多版本并发控制机制
    - InnoDB-MVCC
        - 每行记录保存2个隐藏的列(空间->时间)，1列保存行记录的创建时间，1列保存行记录的删除时间
        - (“创建时间”、“删除时间”可能不是实际的时间，而是系统版本号作为事务的版本号)
        - 插入，当前系统版本号作为创建时间
        - 删除，当前系统版本号作为删除时间
        - 更新，插入一条新纪录，当前系统版本号作为新行的创建时间，当前系统版本号作为原行的删除时间
        - **查询**，事务的版本号大于或等于行记录的创建版本，小于行的删除版本(或行的删除版本未定义)
    - 只在**提交读**和**可重复读**两个隔离级别下工作，使得大多数读操作都不需要加锁，并且保证读取到符合标准的记录行
- InnoDB
    - 事务型(Commit, Rollback)
    - 自动崩溃恢复
    - 支持数据和索引放在单独的文件中
    - MVCC支持高并发
    - 默认隔离级别-可重复读
    - 间隙锁消除幻行的影响
    - 基于聚簇索引建立，主键查询的性能很高
- MyISAM
    - 不支持事务和行级锁(使用**表锁**)
    - 不支持崩溃后安全恢复
    - 支持全文索引
    - 支持表压缩(对于*只读*的数据进行压缩，可以减少磁盘空间和*磁盘IO*)

#### Chapter5
- 索引基础
    - **由存储引擎实现，不在数据库服务层**
    - 若查询使用了索引，MySQL在索引上按值查找后返回所有包含该值的数据行
    - **索引列**的顺序很重要，MySQL只能高效的使用索引的**最左前缀树**
    - (ORM在多数情况下能够产生符合逻辑、合法的查询，却不一定能产生高效的使用索引的查询)
- 类型
    - B Tree，InnoDB使用的是B+Tree，NDB使用的是T Tree
        - MYISAM根据数据的物理位置引用被索引的行，**InnoDB根据主键引用被索引的行**
        - 适用于**全键值、键值范围、或键前缀(最左前缀)查询**
        - 节点是有序组织的，除了按值查找，索引可以用于查询中的**ORDER BY**操作，只要ORDER BY子句满足索引的查询要求
    - 哈希索引(*Memory引擎*、*NDB集群引擎*支持)
        - 存储索引键的哈希码，哈希表则保存指向数据的指针
        - **只有精确匹配索引列的查询才会生效**
    - 自定义哈希索引(伪哈希索引)
        - 针对很长的数据(如**url**)，计算并存储哈希码，并且在哈希列上创建索引
        - 查询时WHERE同时带上哈希码和数据值，防止哈希冲突
        - CRC32()适用于数据量不大的场合，数据量很大时应考虑实现返回整型的64bit哈希函数
            - SHA1()和MD5()是强校验，为了最大限度消除冲突，计算量大且返回字符串，不宜直接使用
            - ```CONV(RIGHT(MD5('https://www.baidu.com'), 16), 16, 10)```
            - **尽可能地使用MySQL内置的<u>函数和特性</u>**
    - 空间数据索引R Tree
        - 用作地理数据存储
    - 全文索引
        - 不是简单的WHERE条件匹配，类似于搜索引擎的功能
- 优点
    - 减少服务器扫描的数据量(先找索引，再找数据)
    - 避免排序和临时表(B Tree是二叉搜索树，数据是有序的 **ORDER BY, GROUP BY**)
    - 将随机IO变成顺序IO(减少IO次数)
- 高效地选择和使用索引
    - **索引列**不能是表达式的一部分，也不能是MySQL函数的参数
    - 对于**BLOB、TEXT**很长类型的列，MySQL必须使用**前缀索引**，不允许索引完整的列
        - 选择合适的前缀长度，保证较高的**索引选择性=COUNT(DISTINC Column) / COUNT(\*)**
        - 使得索引更小、更快
        - 但是，前缀索引无法做GROUP BY和ORDER BY，也无法做覆盖索引
    - EXPLAIN计划中出现索引合并**index_merge**，应该检查表结构和查询语句
        - **WHERE...AND**相交操作上应该建立多列索引
        - **WHERE...OR**可以优化为UNION ALL
        - **UNION ALL**可能消耗大量CPU和内存资源在缓存、排序和合并操作上，还可能影响查询的并发性
    - 合适的索引列顺序很重要
        - B Tree索引能够按照顺序扫描，从而满足GROUP BY、ORDER BY、DISTINCT等符合列顺序的查询要求
        - 通常将**选择性高**的列放在索引最前列(更重要的是，避免随机IO访问和排序)
- 聚簇索引
    - InnoDB支持(通常是主键-唯一非空索引)，它不是单独的索引类型，而是一种数据存储方式
    - 数据行和相邻的键值紧凑地存储在一起，B Tree索引的数据行存放在索引的叶子页
    - B Tree节点页只包含索引列，叶子页包含了行的全部数据
    - **二级索引叶子节点保存的不是指向行的物理地址的指针，而是行的主键值**
    - **对于IO密集型应用，最好避免随机的聚簇索引，索引值不连续且分布范围很大如uuid将使得聚簇索引的插入变得随机**
        - 插入性能变差(随机IO)
        - 索引占用的空间更大(页分裂和页碎片)
    - 尽可能按主键顺序插入数据，尽可能使用单调递增的聚簇键插入数据行
- MyISAM vs InnoDB
    - MyISAM按照插入顺序存放数据(插入顺序可能不等于主键顺序)，主键索引和二级索引都指向数据行的物理地址
    - InnoDB按照主键顺序存放数据，主键索引包含了数据行，二级索引指向主键索引
- 覆盖索引
    - 索引(叶子节点)包含了需要查询的字段值
    - B Tree索引支持，其他如哈希索引、全文索引、空间索引不支持覆盖索引的实现
    - MySQL存储引擎在索引上只能做**最左前缀匹配的LIKE**比较，无法匹配通配符开头的LIKE查询
- <u>索引</u>扫描做<u>排序</u>
    - 索引顺序与ORDER BY查询条件的顺序一致；执行计划中的type显示为"index"
    - 需要索引能够覆盖查询的全部列，若返回表查询数据行，那么随机IO的查询速度很慢
    - 理论上可以使用索引进行关联排序的，优化器可能调整了关联表的顺序，从而导致无法使用索引
- 锁
    - InnoDB访问数据行会对其加锁(即行锁)，在二级索引使用共享读锁，主键索引使用排他锁
    - InnoDB在存储引擎层利用索引过滤掉数据，返回到服务层后应用WHERE子句对余下的行加锁，再次筛选后会释放锁(有的实现在事务提交后才释放锁)
    - *SELECT...FOR UPDATE*
    - InnoDB若不能有效地利用索引，MySQL会扫描并给全表加锁(表锁)
- 优化排序**ORDER BY + LIMIT**
    - 对于**选择性**非常低的列如**性别**，需要创建特殊的索引优化，index(sex, rating)
    - ```SELECT cols FROM table WHERE sex='M' ORDER BY rating LIMIT 10;```
    - 针对翻页的应用场景，特别是LIMIT偏移量很大的翻页，一般采用反范式化、缓存、预先计算的方式处理
    - 另一个方案是延迟关联，利用覆盖索引快速查找需要的主键
        - ```
            SELECT cols
            FROM table
            INNER JOIN
            (SELECT primary_key FROM table WHERE sex='M' ORDER BY rating LIMIT 10000,10) AS t
            USING (primary_key);
          ```

#### Chapter6
- **库表结构优化 + 索引优化 + 查询优化**
- 慢查询日志会记录查询执行的相关信息，如响应时间、扫描的行数、返回的行数
- MySQL应用WHERE
    - 存储引擎层在索引中使用WHERE条件过滤不满足条件的记录
    - 服务器层使用索引覆盖扫描过滤不满足条件的记录，无须返回表查询记录(Using Index)
    - 服务器层从数据表中返回数据，然后过滤不满足条件的记录(Using Where)
- SQL执行过程
    - **SQL** -> 查询缓存 -> 语法解析/预处理 -> 查询优化器 -> 生成**执行计划** -> 存储引擎执行查询计划
- MySQL客户端和服务端的通信协议是**半双工**的，任一时刻只能有一端传送数据
    - 通常需要等所有的数据都发送给客户端后才能释放本次查询占用的资源
    - 只查询需要的数据，常用LIMIT，少用SELECT *
- MySQL线程/连接的状态
    - SHOW FULL PROCESSLIST
    - Sleep
    - Query - 线程正在执行查询
    - Locked - 线程正在等待表锁
    - Analyzing and statistics - 线程正在收集存储引擎的统计信息，并生成执行计划，一般是在进行**GROUP BY/ORDER BY/UNION ALL**操作
    - Sorting results - 排序
    - Sending data - 生成结果集，或发送数据给客户端
- 查询优化
    - 查询优化过程中的**统计信息(数据和索引)**由存储引擎层实现，数据库的服务器层没有任何统计信息
    - MySQL认为任何一个查询都是“关联的”，即使是单表简单查询也视作“关联查询”
    - MySQL对任何关联都执行嵌套循环关联操作
    - MySQL生成一棵查询指令树，而不是生成查询字节码来执行查询
        - 多表查询关联生成的树不是常用的平衡树，而是一棵**左侧深度优先树**(嵌套循环/回溯的结果)
        - **重新定义关联顺序**，通过评估成本决定多表关联的顺序，选择其中成本最小的关联顺序
    - 排序操作的成本很高
        - 单次传输排序，读取需要的数据行，然后按照给定列排序(顺序IO，数据量大时会占用大量的空间)
        - 两次传输排序，先读取给定列数据排序后，再读取需要的数据行(排序时占用的空间小，二次读取数据时会产生大量的随机IO)
        - ORDER BY子句中所有列来自于关联的第一个表，MySQL在关联处理第一个表时就进行文件排序Using filesort
        - ORDER BY子句的其他情况，先将关联的结果存放在一张临时表中，关联结束后再进行文件排序Using temporary;Using filesort
- 存储引擎根据执行计划生成的指令树逐步执行完成数据查询
    - InnoDB实现了细粒度、并发度更高的行锁
    - 存储引擎共有的特性由服务器层实现，如表锁、时间日期函数、视图、触发器等等
- 关联子查询**IN -> EXISTS**
    - ```SELECT * FROM film WHERE film_id IN (SELECT film_id FROM actor WHERE actor_id=1);```
    - 上面查询可能会被改写为
        - ```SELECT * FROM film WHERE EXISTS (SELECT * FROM actor WHERE actor_id=1 AND actor.film_id=film.film_id);```
        - DEPENDENT SUBQUERY，子查询依赖外层的条件，先对film表执行全表扫描，然后对返回的file_id执行子查询
    - 可以使用JOIN优化```SELECT * FROM film INNER JOIN actor USING(film_id) WHERE actor_id=1;```
- 索引合并优化
    - WHERE包含多个复杂条件时，MySQL能够访问单个表中的多个索引以合并和交叉过滤的方式定位需要查找的行记录
- *MySQL无法利用多核特性并行执行查询*
- *MySQL所有的关联都是嵌套循环关联，原生不支持哈希关联*
- 松散索引扫描(Using index for group-by)，跳跃而非连续的方式扫描索引的方式过滤数据
- 索引条件下推(Index Condition Pushdown)，解决松散索引扫描的限制
    - InnoDB只支持联合索引，因为聚簇索引直接能够获取到数据行
    - 联合索引中第1个索引和后续索引成功匹配后，再回到聚簇索引树中查找数据行，减少扫描聚簇索引树的次数
    - 默认开启，**Using Index Condition**，尽量建立联合索引，而不是多个单列索引
- NULL
    - MySQL支持索引列的NULL查询，包括IS NULL/IS NOT NULL，属于范围查询; 因此如果索引失效，可能是回表的数据量太大导致的
    - 非空约束列的IS NULL查询不会走索引，有更高效的查询方式(What???)
- 最大值与最小值**重写查询**
    - ```SELECT MIN(actor_id) FROM actor WHERE name='shs';```字段name上没有索引，将执行全表扫描
    - ```SELECT actor_id FROM actor USE INDEX(PRIMARY) WHERE name='shs' LIMIT 1;```**主键索引扫描**，查询满足条件的第一条记录就返回，使MySQL尽可能少地扫描数据
- **MySQL不允许对一张表同时进行查询和更新**
    - 通过生成表的形式执行多表关联UPDATE
    - 【Error】```UPDATE actor AS outer SET cnt=(SELECT COUNT(*) FROM actor AS inner WHERE inner.type=outer.type);```
    - ```UPDATE actor INNER JOIN (SELECT type, COUNT(*) AS count FROM actor GROUP BY type) AS der USING(type) SET actor.cnt=der.count;```
- Hints
    - **SQL_NO_CACHE** - 不使用缓存，每次读取数据都会进行磁盘IO操作
    - **STRAIGHT_JOIN** - 指定表的关联顺序
    - **FOR UPDATE/LOCK IN SHARE MODE** - 主动给数据行加锁，可能会造成服务器的锁争用问题
    - **USE INDEX/IGNORE INDEX/FORCE INDEX**
- COUNT() - 聚合函数
    - 指定列或列表达式，统计非NULL的数量**设置列属性非NULL，可以加快不带WHERE下COUNT(col)的查询速度**
    - COUNT(*)统计结果集的行数，并不会扩展成统计所有列的数量
- 关联查询
    - 确保ON或USING子句上的列有索引
    - 一般来说，只需要在关联顺序中第2张表的相应列上创建索引; 比如优化器关联顺序是B-A，不需要在B表的相应列建立索引
    - 确保GROUP BY和ORDER BY的表达式只涉及1张表的列，优化器才可能利用索引优化查询过程
- GROUP BY/DISTINCT
    - 最有效的优化方式是使用索引
    - 如果没有索引可用，MySQL使用临时表或文件排序做分组
    - **SQL_MODE设置为ONLY_FULL_GROUPBY，不允许SELECT中直接使用非分组列**
    - **子查询中创建的临时表是没有任何索引的**
    - **GROUP BY分组后若没有排序的需要，可以使用ORDER BY NULL避免文件排序**
- *GROUP BY WITH ROLLUP*对返回的分组结果再做一次超级聚合
- LIMIT
    - OFFSET非常大时会扫描很多不需要的数据
    - ```SELECT film_id, film_name FROM film ORDER BY title LIMIT 1000, 20```
    - 尽量使用**索引覆盖扫描**，然后关联操作后返回所需要的列，避免查询所有的列
    - ```SELECT film_id, film_name FROM film INNER JOIN (SELECT film_id FROM film ORDER BY title LIMIT 1000, 20) AS lim USING(film_id);```**延迟关联**
    - 连续翻页情况下可以记录上一次翻页的位置，下一次翻页从前一次翻页点开始，可以减少扫描的数据量
        - ```SELECT * FROM rental ORDER BY rental_id DESC LIMIT 20;```
        - ```SELECT * FROM rental WHERE rental_id < 16030 ORDER BY rental_id DESC LIMIT 20;```
    - 还可以使用<u>汇总表、冗余表</u>、<u>缓存、预读取</u>等方式
- UNION
    - 创建并填充临时表的方式执行查询，优化器很难做优化
    - 手动地将WHERE、ORDER BY、LIMIT等子句“下推”到UNION的各个子句中
    - UNION ALL > UNION，UNION默认添加DISTINCT选项，会对整个临时表作唯一性检查
- 用户自定义变量(部分场景中很有用)
    - 使用时无法利用查询缓存
    - 生命周期在一个连接中有效
    - 变量的类型是动态的，赋值时会发生变化
    - **变量的赋值时间和顺序依赖于优化器，实际运行情况可能和预期存在差异**
        - Invalid
          ```
          SELECT actor_id, @cur:=COUNT(*) AS cnt, @rank:=IF(@pre<>@cur, @rank+1, @rank) AS rank, @pre:=@cur AS dummy
          FROM film_actor
          GROUP BY actor_id
          ORDER BY cnt DESC
          LIMIT 10;
          ```
        - Valid 分2步，子查询生成临时表先查询相关信息，然后再对数据进行排名
          ```
          SELECT actor_id, @cur:=cnt AS cnt, @rank:=IF(@pre<>@cur, @rank+1, @rank) AS rank, @pre:=@cur AS dummy
          FROM(
              SELECT actor_id, COUNT(*) AS cnt
              FROM film_actor
              GROUP BY actor_id
              ORDER BY cnt DESC
              LIMIT 10
          ) AS der;
          ```
    - 避免重复查询刚刚更新的数据
        - ```
          UPDATE t SET lastupdate=NOW() WHERE id=123;
          SELECT lastupdate FROM t WHERE id=123;
          ```
        - ```
          UPDATE t SET lastupdate=NOW() WHERE id=123 AND @now:=NOW();
          SELECT @now;
          ```
        - 需要2次查询和2个网络往返，不过第2次查询不需要访问任何表就可以得到结果
    - 案例
        - 统计数量时加上排名操作
        - 查询时计算总数和平均值
    - *INSERT INTO...ON DUPLICATE KEY* 插入重复记录
- 查询条件(列)的类型一定要匹配，不然无法利用索引查询
    - 比如字符串列要加""，否则会被数据库隐式转换为数字
- 操作符!=, <>, NOT IN在大多数情况下不会利用索引，具体实现依赖于存储引擎的优化策略
    - NOT LIKE一定不会使用索引
