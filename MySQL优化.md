#MySQL优化
## OPTIMIZE 
``` sql
REPAIR TABLE `table_name` 修复表 
OPTIMIZE TABLE `table_name` 优化表 
```
* REPAIR TABLE 用于修复被破坏的表。 
* OPTIMIZE TABLE 用于回收闲置的数据库空间，当表上的数据行被删除时，所占据的磁盘空间并没有立即被回收，使用了OPTIMIZE TABLE命令后这些空间将被回收，并且对磁盘上的数据行进行重排（注意：是磁盘上，而非数据库）。多数时间并不需要运行OPTIMIZE TABLE，只需在批量删除数据行之后，或定期（每周一次或每月一次）进行一次数据表优化操作即可，只对那些特定的表运行。

##OPTIMIZE TABLE对InnoDB 和 MyISAM相关知识
1. InnoDB 和 MyISAM
目前支持optimize命令的引擎有 MyISAM, InnoDB, and ARCHIVE，对于InnoDB，会将optimize命令映射为ALTER TABLE命令，该命令会重建数据表，更新索引统计信息、回收主键索引中空间。
2. InnoDB 和 MyISAM
如果你的MySQL是有备库的，如果你只希望在主库上执行的话，那么可以加上关键字NO_WRITE_TO_BINLOG（或者LOCAL，意思完全相同）。
``` sql
OPTIMIZE [NO_WRITE_TO_BINLOG | LOCAL] TABLE
```