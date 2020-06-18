# optimize table命令

> 本篇介绍optimize table命令的使用背景和方法。

**背景**：在使用mysql的时候有时候，可能会发现尽管一张表删除了许多数据，但是这张表的数据文件和索引文件却奇怪的没有变小。这是因为mysql在删除数据（特别是有Text和BLOB）的时候，会留下许多的数据空洞，这些空洞会占据原来数据的空间，所以文件的大小没有改变。这些空洞在以后插入数据的时候可能会被再度利用起来，当然也有可能一直存在。这种空洞不仅额外增加了存储代价，同时也因为数据碎片化降低了表的扫描效率。

**使用场景**：如果你已经删除了表的一大部分，或者如果您已经对含有可变长度行的表（含有VARCHAR, BLOB或TEXT列的表）进行了很多更改，则应使用OPTIMIZE TABLE。被删除的记录被保持在链接清单中，后续的INSERT操作会重新使用旧的记录位置。您可以使用OPTIMIZE TABLE来重新利用未使用的空间，并整理数据文件的碎片。（当您的库中删除了大量的数据后，您可能会发现数据文件尺寸并没有减小。这是因为删除操作后在数据文件中留下碎片所致。）

**注意事项**

+ 在多数的设置中根本不需要运行OPTIMIZE TABLE。即使您对可变长度的行进行了大量的更新，您也不需要经常运行，每周一次或每月一次即可，只对特定的表运行。

+ OPTIMIZE TABLE只对MyISAM, BDB和InnoDB表起作用。

+ 对于BDB表，OPTIMIZE TABLE目前被映射到ANALYZE TABLE上。对于InnoDB表，OPTIMIZE TABLE被映射到ALTER TABLE上，这会重建表。重建操作能更新索引统计数据并释放成簇索引中的未使用的空间。

+ 注意：在OPTIMIZE TABLE运行过程中，MySQL会锁定表。

**使用方法**

对于myisam可以直接使用 optimize table table.name。

当是InnoDB引擎时，会报“Table does not support optimize, doing recreate + analyze instead”，一般情况下，由myisam转成innodb，会用alter table table.name engine='innodb'进行转换，优化也可以用这个。所以当是InnoDB引擎时我们就用alter table table.name engine='innodb'来代替optimize做优化就可以。

查看前后效果可以使用show table status命令,例如show table status from [database] like '[table_name]';返回结果中的data_free即为空洞所占据的存储空间。