# 《高性能MySQL》学习梳理

## 第一章 MySQL架构与历史

## 1.1MySQL逻辑架构

![MySQL服务器逻辑架构图](.\images\MySQL服务器逻辑架构图.jpg)

(图片来源于网络，侵删)

**最上层**：连接处理、授权认证、安全等；

**第二层**：大多数Mysql的核心服务功能都在这一层，包括查询解析、分析、优化、缓存以及所有的内置函数（例如，日期、时间、数学和加密函数）；

**第三层**：存储引擎。存储引擎负责MySQL中数据的存储和提取。



## 1.2 并发控制

**读写锁**

+ 共享锁、读锁
+ 排他锁、写锁

**锁粒度**：

+ 加锁也需要消耗资源。

+ 表锁：最基本；开销最小；
+ 行级锁：最大程度支持并发处理；最大的锁开销；行级锁只在存储引擎层，而MySQL服务器层没有实现。



## 1.3 事务

**定义**：事务内的语句要么全部执行成功，要么全部执行失败。

**事务的ACID概念**

+ 原子性
+ 一致性
+ 隔离性
+ 持久性

**须知**：

+ 事务处理过程中的安全性，也会需要数据库系统做更多的额外工作。
+ 用户可以根据业务是否需要事务处理，来选择合适的存储引擎。

**隔离级别**

+ READ UNCOMMITED（未提交读）：脏读；
+ READ COMMITED（提交读）
+ REPEATABLE READ（可重复读）
+ SERIALIZABLE（可串行化）

**死锁**

+ 当多个事务试图以不同的顺序锁定资源时，就可能会产生死锁。

+ 解决：死锁检测；死锁超时；

+ 可能完全由于存储引擎的实现方式导致。

**事务日志**

+ 追加的方式，因此写日志的操作是磁盘上一小块区域内的顺序IO，而不像随机IO需要在磁盘的多个地方移动磁头。
+ 预写式日志（write-Ahead Logging），修改数据需要两次写盘。

**MySQL中的事务**

+ 自动提交（AUTOCOMMIT）:如果不是显示的开始一个事务，每一个查询都被当作一个事务执行提交操作。
+ `set autocommit = 1` ：1 或者 ON 表示启用，0 或者 OFF 表示禁用。
+ 有一些命令，在执行之前会强制执行COMMIT提交当前的活动事务。如在数据定义语言（DDL）中。
+ **设置隔离级别**：
  + `set session transaction isolation leavel read commited`
+ 事务是由**存储引擎**实现的。所以在同一个事务中是使用多种存储引擎是不可靠的。
+ 如果事务需要回滚，非事务型上的表上的变更就无法撤销。
+ **隐式和显示锁定**
  + InnoDB采用的是两阶段锁协定（two-phase lodking protocol）。
  + 一般的锁都是隐式锁，InnoDB会根据隔离级别在需要的时候自动加锁。
  + 显示锁定：
    + InnoDB支持通过特定的语句进行显示锁定。不属于SQL规范。
    + `select ... lock in share mode`
    + `select ... for update`

**多并发版本控制**

**须知**：

+ 可以认为MVCC是行级锁的一个变种。但是很多情况下避免了加锁操作，因此开销更低。
+ 不同存储引擎的MVCC实现不同，分为乐观并发控制和悲观并发控制。

InnoDB的MVCC

+ 通过每行记录后面保存两个隐藏的列来实现。一个保存的行的创建时间，一个保存了行的过期时间（或删除时间）。这里的时间指系统版本号。
+ SELECT:
+ INSERT:
+ DELECT:
+ UPDATE:
+ 好处：大多数读操作都可以不用加锁。
+ 不足：
  + 需要额外的存储空间；
  + 需要更多的行检查操作；
  + 额外的维护工作。

## 1.5 MySQL的存储引擎

须知：

+ 在文件系统中，MySQL将每个数据库（或者说schema）保存威数据目录下的一个子目录。
+ 创建表时，MySQL会在数据库在目录下创建一个和表同名的 .frm 文件保存表的定义。
+ 大小写敏感和具体平台相关。Windows系统，大小写不敏感；类Unix系统中则是敏感的。
+ `show table status like '[table_name]'\G` 可以显示表的相关信息。



**InnoDB 存储引擎**

+ InnoDB的数据存储在表空间（tablespace）中。
+ 默认隔离级别时 REPEATABLE READ（可重复读），并且通过间隙锁（next-key locking）策略防止幻读的出现。
+ InnoDB的表基于聚簇索引建立，对主键查询有很高的性能。
+ 若表上索引较多，主键应当尽可能的小。因为每个索引都会在最后带上主键。
+ 内部优化：
  + 自适应哈希索引

**MyISAM存储引擎**

须知：

+ 提供特性：全文索引、压缩、空间函数（GIS）
+ 不支持事务和行级锁；
+ 崩溃后无法安全恢复；

特性：

+ 存储：包含动态或者静态（长度固定）行。
+ 加锁与并发
+ 修复
+ 索引特性
+ 延迟更新索引键
+ MyISAM压缩表
  + 使用myisampack对myisam表进行压缩，也叫打包pack。
  + 压缩表无法修改
  + 极大减少从磁盘空间占用，也可以减少磁盘I/O，从而提升查询性能。
  + 压缩表也支持索引，但索引也是只读的。

## 第二章 MySQL基准测试



## 第三章 服务器性能剖析



## 第四章 Schema与数据类型优化



## 第五章 创建高性能的索引



## 第六章 查询性能优化



## 第七章 MySQL高级特性

### 7.1 分区表



### 7.9 字符集和校对



## 第十章 复制

