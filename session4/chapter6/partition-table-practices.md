## 6.4.3 TiDB 分区表最佳实践

* 当业务写入有问题，需要清理某个分区数据时，不用批量 DELETE 数据，可以通过 TRUNCATE 命令直接清理分区数据：

    `ALTER TABLE employees_attendance TRUNCATE PARTITION p20200306;`

* TiDB 收集的统计信息可能不够准确，会导致 SQL 因选错索引而出现性能问题。如果表太大，手动收集全表统计信息时间太长，解决不了当时的 SQL 性能问题。此时可以只对 SQL 中用到的分区进行统计信息收集：

    `ALTER TABLE employees_attendance ANALYZE PARTITION p20200306;`

* 使用分区表函数来简化运维。在实际环境中，有的程序会用时间戳格式来存储时间， DBA 创建新分区时需要将日期转换成时间戳，然后建立分区。这种操作比较麻烦，此时可以使用 TiDB 支持的 UNIX_TIMESTAMP 函数来搞定：
  
    `ALTER TABLE ADD PARTITION p20200306 VALUES LESS THAN (UNIX_TIMESTAMP('2020-03-07'))`

    关于 TiDB 支持的分区表达式函数，详情见[函数](https://pingcap.com/docs-cn/v3.1/reference/sql/partitioning/)

* Range 分区中 MAXVALUE 表示一个比所有整数都大的整数。避免因 Data Maintance 脚本问题导致分区没有创建，影响业务写入，详情见上面例子。

* Hash 分区中最高效的 Hash 函数是作用在单列上，并且函数的单调性跟列的值是一样递增或者递减的，这种情况可以像 Range 分区一样进行分区裁剪。不推荐在 Hash 分区表达式中涉及多个列。

* 分区表有损调整字段 ( TiDB 默认不支持有损更新 ) ，可以通过创建新字段列，将原列的值 UPDATE 到新列，然后 DROP 原字段的方式实现。
