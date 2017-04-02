### Mysql Replace Into

replace into 跟 insert 功能类似，

不同点在于：replace into 首先尝试插入数据到表中

1. 如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据。

2. 否则，直接插入新数据。

> 要注意的是：插入数据的表必须有主键或者是唯一索引！否则的话，replace into 会直接插入数据，这将导致表中出现重复的数据。

replace into 有三种形式：

	replace into tbl_name(col_name, ...) values(...)
	replace into tbl_name(col_name, ...) select ...
	replace into tbl_name set col_name=value, ...

前两种形式用的多些。其中 “into” 关键字可以省略，不过最好加上 “into”，这样意思更加直观。另外，对于那些没有给予值的列，MySQL 将自动为这些列赋上默认值。

#### Insert update
INSERT 中 ON DUPLICATE KEY UPDATE的使用

如果您指定了ON DUPLICATE KEY UPDATE，并且insert行后会导致在一个UNIQUE索引或PRIMARY KEY中出现重复值，则执行旧行UPDATE。例如，如果列a被定义为UNIQUE，并且包含值1，则以下两个语句具有相同的效果：

	mysql> INSERT INTO table (a,b,c) VALUES (1,2,3)  ON DUPLICATE KEY UPDATE c=c+1;
	mysql> UPDATE table SET c=c+1 WHERE a=1;

总之
如果表中不存在主键记录，replace和insert*update都与insert是一样的特点。

如果表中存在主键记录，replace相当于执行delete 和 insert两条操作，而insert*update的相当于执行if exist do update else do insert操作。

因此，如果replace填充的字段不全，则会导致未被更新的字段都会修改为默认值，并且如果有自增id的话，自增id会变化为最新的值（这样如果是以自增id为标志的话可能导致记录丢失）；而insert*update只是更新部分字段，对于未被更新的字段不会变化（不会强制修改为默认值）。