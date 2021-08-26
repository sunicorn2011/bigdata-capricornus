[toc]



***
# 0、Hive基本概念
## 0.1 
- 1）基本概念：解决海量`结构化`日志的数据`统计工具`，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。
- 2）Hive本质：将HQL转化成MapReduce程序
![image](8D26EDD42D30417F82CBAC185FB1B599)

## 0.2 Hive的优缺点
- 优点：
    - 1、操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）。
    - 2、避免了去写MapReduce，减少开发人员的学习成本
    - 3、Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合
    - 4、Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。
    - 5、Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数
    -
- 缺点：
    - 1、Hive的HQL表达能力有限
        - 迭代式算法无法表达
        - 数据挖掘方面不擅长，由于MapReduce数据处理流程的限制，效率更高的算法却无法实现。
        - 
    - 2、Hive的效率比较低
        - Hive自动生成的MapReduce作业，通常情况下不够智能化
        - Hive调优比较困难，粒度较粗
        - 

## 0.3 Hive的基础架构
- 基础架构图
- 
![image](9E8A906D97FB460E98735C2837DC25E4)
****
- 用户角色阐述：
    - 用户接口：Client
        - CLI（command-line interface）、JDBC/ODBC(jdbc访问hive)、WEBUI（浏览器访问hive） 
        - 
    - 元数据：Metastore
        - 元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；
        - 默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore
        - 
    - Hadoop
        - 使用HDFS进行存储，使用MapReduce进行计算
        - 
    - 驱动器：Driver
        - 1、解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST
        - 2、编译器（Physical Plan）：将AST编译生成逻辑执行计划
        - 3、优化器（Query Optimizer）： 对逻辑执行计划进行优化。
        - 4、执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。
        - 

## 0.4 Hive运行机制
- 运行机制图
![image](74F2264252E542AAB583B0E7A3F911F9)
- 
- 执行过程：
    - Hive通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，
    - 使用自己的Driver，结合元数据(MetaStore)，将这些指令翻译成MapReduce，
    - 提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口。 
    - 

# 1、DDL数据定义
## 1.1 创建表
```md
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] 
[COMMENT table_comment] 
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
[CLUSTERED BY (col_name, col_name, ...) 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
[ROW FORMAT row_format] 
[STORED AS file_format] 
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]
```
- 关键字说明
    - EXTERNAL: 创建一个外部表，默认缺失，则创建的表为内部表（即管理表）。
        - `内部表和外部表区别`：
            - 创建时：当使用load data加载数据时候，数据只保存一份（即执行了类似`-mv` 命令），而外部表执行了类似`-cp`命令
            - 删除时：当删除表时候，内部表会表结构和数据一起删除。而外部表不会删除数据。
    - PARTITIONED BY: 创建分区表
    - CLUSTERED BY创建分桶表
    - SORTED BY不常用，对桶中的一个或多个列另外排序
    - row format delimited `field`  ITEMS TERMINATED BY `char` 数据切分 
    - STORED AS 指定存储文件类型
        - SEQUENCEFILE（二进制序列文件）
        - TEXTFILE（文本）
        - RCFILE（列式存储格式文件）
    - LOCATION ：指定表在HDFS上的存储位置。默认在表目录下
    - 
     
# 2、DML数据操作
- 语法
```
load data [local] inpath '数据的path' [overwrite] into table student [partition (partcol1=val1,…)];
```

- 关键字说明
    - load data:表示加载数据
    - local: 表示从本地加载，否则从HDFS加载
    - inpath: 加载路径
    - overwrite： 表示中已有数据，会覆盖，默认参数不添加
    - partition： 表示数据加载到那个分区中

# 3、查询
```
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
    FROM table_reference
    [WHERE where_condition]
    [GROUP BY col_list]
    [ORDER BY col_list]
    [CLUSTER BY col_list
        | [DISTRIBUTE BY col_list] [SORT BY col_list]
    ]
[LIMIT number]
```
- 3.1 关键字说明
    - 1、
    - 
- 3.2 基本查询
    - 2、
    - 
- 3.3 分组查询
    - `Group By`: GROUP BY语句通常会和 `聚合函数一起使用`，按照一个或者多个列队结果进行分组，然后对`每个组执行聚合操作`(即`count()、sum()、avg()等等操作`)
    - 
    - Having语句 (即`可以是个条件语句`)
        - having(即`可以把select后得到的结果在一次做分组, select的后置`)与where（即`select的前置查询条件`）不同点
            - 1、where后面不能写分组函数，而having后面可以 `使用分组函数得到的结果`。
            - 2、having只用于group by分组统计语句。
            - 
        - 实操:
            ```
            --- 求每个部门的平均薪水大于2000的部门
                hive (default)> select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000;
            ```
# 4、排序
- 4.1 全局排序（order by）
    - DESC：降序
    - ASC： 升序
- 4.2 每个Reduce内部排序（Sort By）
    - Sort by为每个reducer产生一个排序文件 
    - 
- 4.3 分区（Distribute By）
    - 在有些情况下，我们需要控制某个特定行应该到哪个reducer，通常是为了进行后续的聚集操作
    - distribute by的分区规则是根据分区字段的hash码与reduce的个数进行模除后，余数相同的分到一个区
    - 
- 4.4 Cluster By
    - 当distribute by和sort by字段相同时，可以使用cluster by方式。
    - 
# 5、分区表和分桶表
- 5.1 动态分区
    - 静态分区
        - 步骤：
            - 
            - 
- 5.2 动态分区
    - 
    - 
# 6、函数
- 6.1 常用内置函数
    - 6.1.1
        - CASE [`filed_name`] WHEN [`filed_vale`] THEN [`exp`] ELSE [`exp`] END
    - 6.1.2
        - 行转列（`多行变一行，数据合并`）
            - `CONCAT`: 字符串`直接`连接
            - `CONCAT_WS`: 字符串安装`分隔符`链接,接收字符 `CONCAT_WS must be "string or array<string>`
            - `COLLECT_SET(col)`: 将某字段的值进行去重汇总，产生array类型字段。
        - 例子
            ```
            select
                constellation,
                blood_type,
                CONCAT_WS("|",COLLECT_SET(name))
                from 
                person_info
                group by
                constellation,blood_type
                ;
            ```
    - 6.1.3
        - 列转行（`一行变多行`）
            - `EXPLODE(col)`：将hive一列中复杂的array或者map结构拆分成多行。
            - LATERAL VIEW
                - 用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias
                - 用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。
        - 例子：
            ```
            select
                movie,
                category_name
                from
                movie_info
                lateral view
                explode(split(category, ',')) category_tmp as category_name
                ; 
            ```
    - 6.1.4 窗口函数（`开窗函数 必须和某个行数一起使用`）
        - 普通聚合函数聚合的行集是组，开窗函数聚合的行集是窗口。
因此，`普通聚合函数`每组（Group by）只有`一个返回值`，而`开窗函数`则可以为窗口中的`每行都返回一个值`。
        - `OVER()：`指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的改变而变化。
            - CURRENT ROW：当前行
            - n PRECEDING：往前n行数据
            - n FOLLOWING：往后n行数据
            - `UNBOUNDED`: 起点，
                - UNBOUNDED PRECEDING 表示从前面的起点，
                - UNBOUNDED FOLLOWING 表示到后面的终点
            - LAG(`col`, n, default_val)：往前第n行数据
            - LEAD(`col`, n, default_val)：往后第n行数据
            - NTILE(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从1开始， 对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。
     - 6.1.5 排序行数
        - RANK() 排序相同时会重复，总数不会变
        - DENSE_RANK() 排序相同时会重复，总数会减少
        - ROW_NUMBER() 会根据顺序计算 
    - 自定义函数
        - 根据用户自定义函数类别分为以下三种
            - UDF（User-Defined-Function）
                - `一进一出`
            - UDAF（User-Defined Aggregation Function）
                - `聚集函数，多进一出`
                - 例子: `count/max/min`
            - UDTF（User-Defined Table-Generating Functions）
                - `一进多出`
                - 例子：lateral view explode()
            -
# 企业级调优
## 1、执行计划（Explain）
-

## 2、Fetch抓取
- Fetch抓取指的时Hive中对某些情况的查询可以不必使用MapReduce计算
- 设置参数：( `hive-default.xml.template`)
    - `hive.fetch.task.conversion = more` 
    - 
- 该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。
- 
## 3、表的优化
### 1、小表大表Join(MapJoin)
- 1、思路：将key相对分散，并且数据量小的表放在join的左边，这样可以有效减少内存溢出错误发生的几率；再进一步，可以使用map join让小的维度表（1000条以下的记录条数）先进内存。在map端完成join。
- 实测：新版的hive已经对小表JOIN大表和大表JOIN小表进行了优化，表放左右没有影响
- 
- 具体实现：
    - 1、开启MapJoin参数设置 
    - 2、`set hive.auto.convert.join = true; 默认为true`
    - 3、大表小表的阈值设置（默认25M以下认为是小表）：
        - `set hive.mapjoin.smalltable.filesize = 25000000;` 
        - 
- 2、MapJoin工作机制
- ![image](68D8A2C498A74DE5B05F49EF954503E5)
-
### 2、Group By
- 1、默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜
    - ![image](37E5C9C56A9E4850B4FCD0F68A7E4B02)

- 2、思路：很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果
- 3、参数设置：
    - 1、是否在Map端进行聚合，默认为True
        - `set hive.map.aggr = true` 
        - 
    - 2、设置在Map端进行聚合操作的条目数目
        - `set hive.groupby.mapaggr.checkinterval = 100000`
        - 
    - 3、有数据倾斜的时候进行负载均衡（默认是false）
        - `set hive.groupby.skewindata = true`
        - 流程解释：
            - 1、当选项设定为 true，生成的查询计划会有两个MR Job
            - 2、第1个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，达到负载均衡的目的
            - 3、二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中，最后完成最终的聚合操作
            - 
### 3、尽量避免使用笛卡尔积
- 尽量避免笛卡尔积，join的时候不加on条件，或者无效的on条件
- 
### 4、合理设置Map及Reduce数
- 4.1 复杂文件增加Map数
    - 增加的依据：`computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M`
    - 
- 4.2 小文件进行合并
    - 1、在map执行前合并小文件，减少map数：CombineHiveInputFormat具有对小文件进行合并的功能（系统默认的格式）。HiveInputFormat没有对小文件合并功能。
        - `set hive.input.format= org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;`
        - 
    - 2、在Map-Reduce的任务结束时合并小文件的设置：
        - 2.1 在map-only任务结束时合并小文件，默认true
            - `SET hive.merge.mapfiles = true;`
            - 
        - 2.2 在map-reduce任务结束时合并小文件，默认false
            - 
            ```
            SET hive.merge.mapredfiles = true; 
            
            --- 合并文件的大小，默认256M
            SET hive.merge.size.per.task = 268435456;
            
            --- 当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
            SET hive.merge.smallfiles.avgsize = 16777216;
            ``` 
            - 
        - 2.3 
- 4.3 合理设置Reduce数
    - 
### 并行执行
### 严格模式
### JVM重用
### 压缩

















