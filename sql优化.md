### <font face='华文行楷'>sql优化</font>

> 1. 对查询进行优化，要尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引

1. MySQL 索引的概念

   索引是一种特殊的文件 (InnoDB 数据表上的索引是表空间的一个组成部分)，它们包含着对数据表里所有记录的引用指针。

   更通俗的说，数据库索引好比是一本书前面的目录，能加快数据库的查询速度。

   索引分为聚簇索引和非聚簇索引两种，聚簇索引是按照数据存放的物理位置为顺序的，而非聚簇索引就不一样了；聚簇索引能提高多行检索的速度，而非聚簇索引对于单行的检索很快。

   - MySQL 索引的类型

     1. 普通索引：这是最基本的索引，它没有任何限制，比如上文中为 title 字段创建的索引就是一个普通索引，MyIASM 中默认的 BTREE 类型的索引，也是我们大多数情况下用到的索引。

     ```mysql
     -- 直接创建索引（如果是 CHAR，VARCHAR 类型，length 可以小于字段实际长度；如果是 BLOB 和 TEXT 类型，必须指定 length。）
     CREATE INDEX index_name ON table(column(length));
     
     -- 修改表结构的方式添加索引
     ALTER TABLE table_name ADD INDEX index_name ON (column(length));
     
     -- 创建表的时候直接指定
     CREATE TABLE table(
      id int(11) NOT NULL AUTO_INCREMENT ,
      title char(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL ,
      content text CHARACTER SET utf8 COLLATE utf8_general_ci NULL ,
      time int(10) NULL DEFAULT NULL ,
      PRIMARY KEY (`id`),
      INDEX index_name (title(length))
     );
     
     -- 删除索引的语法
     DROP INDEX index_name ON table_name; 
     # 使用 ALTER 命令中使用 DROP 子句来删除索引
     ALTER TABLE table_name DROP INDEX indexName;
     
     
     -- 显示索引信息
     SHOW INDEX FROM table_name;
     ```

     2. 唯一索引：与普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值（注意和主键不同）。如果是组合索引，则列值的组合必须唯一，创建方法和普通索引类似。

       ```mysql
     #创建唯一索引
     CREATE UNIQUE INDEX indexName ON table(column(length));
     #修改表结构
     ALTER TABLE table_name ADD UNIQUE indexName ON (column(length));
     #创建表的时候直接指定
     CREATE TABLE `table` (
      `id` int(11) NOT NULL AUTO_INCREMENT ,
      `title` char(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL ,
      `content` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL ,
      `time` int(10) NULL DEFAULT NULL ,
      PRIMARY KEY (`id`),
      UNIQUE indexName (title(length))
     );
       ```

     3. 全文索引（FULLTEXT）：MySQL 从 3.23.23 版开始支持全文索引和全文检索，FULLTEXT 索引仅可用于 MyISAM 表；他们可以从 CHAR、VARCHAR 或 TEXT 列中作为 CREATE TABLE 语句的一部分被创建，或是随后使用 ALTER TABLE 或 CREATE INDEX 被添加。

        不过新版的 MySQL5.6.24 上 InnoDB 引擎也加入了全文索引

        对于较大的数据集，将你的资料输入一个没有 FULLTEXT 索引的表中，然后创建索引，其速度比把资料输入现有 FULLTEXT 索引的速度更为快。不过切记对于大容量的数据表，生成全文索引是一个非常消耗时间非常消耗硬盘空间的做法。

        ```mysql
     /*直接创建索引*/
     CREATE FULLTEXT INDEX indexName ON table(column(length));
     /*修改表结构添加全文索引*/
     ALTER TABLE table_name ADD FULLTEXT indexName ON (column(length));
     
     /*–创建表的适合添加全文索引*/
     CREATE TABLE `article` (
      `id` int(11) NOT NULL AUTO_INCREMENT ,
      `title` char(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL ,
      `content` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL ,
      `time` int(10) NULL DEFAULT NULL ,
      PRIMARY KEY (`id`),
      FULLTEXT (content)
     );
        ```

     4. 单列索引、多列索引：多个单列索引与单个多列索引的查询效果不同，因为执行查询时，MySQL只能使用一个索引，会从多个索引中选择一个限制最为严格的索引。

     5. 组合索引（最左前缀）：平时用的 SQL 查询语句一般都有比较多的限制条件，所以为了进一步榨取 MySQL 的效率，就要考虑建立组合索引。

        ```mysql
        #例如上表中针对 title 和 time 建立一个组合索引：
        ALTER TABLE article ADD INDEX index_title_time (title (50),time (10));
        ```

        建立这样的组合索引，其实是相当于分别建立了下面两组组合索引：

        –title,time

        –title

        为什么没有 time 这样的组合索引呢？这是因为 MySQL 组合索引 “最左前缀” 的结果。简单的理解就是只从最左面的开始组合。并不是只要包含这两列的查询都会用到该组合索引，如下面的几个 SQL 所示：

        ```mysql
        #使用到上面的索引
        SELECT * FROM article WHREE title='测试' AND time=1234567890;
        SELECT * FROM article WHREE title='测试';
        #不使用上面的索引
        SELECT * FROM article WHREE time=1234567890;
        ```

   - MySQL索引的优化

     > 上面都在说使用索引的好处，但过多的使用索引将会造成滥用。
     >
     > 索引也会有它的缺点：虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行 INSERT、UPDATE 和 DELETE。
     >
     > 因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件。建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会膨胀很快。索引只是提高效率的一个因素，如果你的 MySQL 有[大数据](http://lib.csdn.net/base/20)量的表，就需要花时间研究建立最优秀的索引，或优化查询语句。下面是一些总结以及收藏的 MySQL 索引的注意事项和优化方法。

     1. 何时使用聚集索引或非聚集索引?

        > 聚集（clustered）索引，也叫聚簇索引
        >
        > 定义：数据行的物理顺序与列值（一般是主键的那一列）的逻辑顺序相同，一个表中只能拥有一个聚集索引
        >
        > 非聚集（unclustered）索引
        >
        > 定义：该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同，一个表中可以拥有多个非聚集索引。细分一下非聚集索引，分成普通索引，唯一索引，全文索引

        ![mark](http://cdn.cescforz.cn/blog/20190612/140223893.jpg)

        事实上，我们可以通过前面聚集索引和非聚集索引的定义的例子来理解上表。

        如：返回某范围内的数据一项。比如您的某个表有一个时间列，恰好您把聚合索引建立在了该列，这时您查询 2004 年 1 月 1 日至 2004 年 10 月 1 日之间的全部数据时，这个速度就将是很快的，因为您的这本字典正文是按日期进行排序的，聚类索引只需要找到要检索的所有数据中的开头和结尾数据即可；而不像非聚集索引，必须先查到目录中查到每一项数据对应的页码，然后再根据页码查到具体内容。

     2.  索引不会包含有 NULL 值的列

        只要列中包含有 NULL 值都将不会被包含在索引中，复合索引中只要有一列含有 NULL 值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为 NULL。

     3. 使用短索引

        对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个 CHAR (255) 的列，如果在前 10 个或 20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和 I/O 操作。

     4. 索引列排序

        MySQL 查询只使用一个索引，因此如果 where 子句中已经使用了索引的话，那么 order by 中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。

     5. like 语句操作

        一般情况下不鼓励使用 like 操作，如果非使用不可，如何使用也是一个问题。like “% aaa%” 不会使用索引而 like “aaa%” 可以使用索引。

     6. 不要在列上进行运算

        例如：

        select * from users where YEAR (adddate)<2007；

        将在每个行上进行运算，这将导致索引失效而进行全表扫描，因此我们可以改成：

        select * from users where adddate<’2007-01-01′。

        关于这一点可以围观：[一个单引号引发的 MYSQL 性能损失。](http://www.zendstudio.net/archives/single-quotes-or-no-single-quotes-in-sql-query)

     7. MySQL 只对以下操作符才使用索引：

        > <,<=,=,>,>=,between,in, 以及某些时候的 like (不以通配符 % 或_开头的情形)。
        >
        > 而理论上每张表里面最多可创建 16 个索引，不过除非是数据量真的很多，否则过多的使用索引也不是那么好玩的，比如我刚才针对 text 类型的字段创建索引的时候，系统差点就卡死了。



> 2.应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：
>
> ```mysql
> select id from t where num is null
> ```
>
> 最好不要给数据库留 NULL，尽可能的使用 NOT NULL 填充数据库
>
> 可以在 num 上设置默认值 0，确保表中 num 列没有 null 值，然后这样查询：
>
> ```mysql
> select id from t where num = 0
> ```



> 3.应尽量避免在 where 子句中使用 != 或 <> 操作符，否则将引擎放弃使用索引而进行全表扫描



> 4.应尽量避免在 where 子句中使用 or 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描，如：

```mysql
select id from t where num=10 or Name = 'admin'
```

改为：

```mysql
select id from t where num = 10
union all
select id from t where Name = 'admin'
```

> 拓展
>
> MySQL UNION 用于把来自多个select语句的结果组合到一个结果集合中。语法为：
>
> ```
> SELECT column,... FROM table1 
> UNION [ALL]
> SELECT column,... FROM table2
> ...
> ```
>
> 在多个 SELECT 语句中，对应的列应该具有相同的字段属性，且第一个 SELECT 语句中被使用的字段名称也被用于结果的字段名称。
>
> UNION 与 UNION ALL 的区别
>
> 当使用 UNION 时，MySQL 会把结果集中重复的记录删掉，而使用 UNION ALL ，MySQL 会把所有的记录返回，且效率高于 UNION。

AND和OR优先级问题

```mysql
#结果集中不仅包含 READ 还包含 OPERATE,AND条件失效
SELECT * FROM common_message WHERE message_category = 'READ' AND 
message_status != '0' OR message_status IS NULL;
#原因是 AND 和 OR 优先级一样，sql 执行的时候按照顺序，所以结果集就不对了，如果加了 “（）” 之后，优先级高了，就会先查找不等于 0 的和为 null 的，然后在这个基础上进行 and 条件的查询
SELECT * FROM common_message WHERE message_category = 'READ' AND 
(message_status != '0' OR message_status IS NULL);
```

> 5.in 和 not in 也要慎用，否则会导致全表扫描，如：

```mysql
select id from t where num in(1,2,3);
```

对于连续的数值，能用 between 就不要用 in 了：

```mysql
select id from t where num between 1 and 3;
```

很多时候用 exists 代替 in 是一个好的选择：

```mysql
select num from a where num in(select num from b);
#用下面的语句替换
select num from a where exists(select 1 from b where num=a.num)

-- 精准解读 in 和 exists
#这两个关键字的区别主要是在于子查询上面，in 是独立子查询，exists 是相关子查询，例如：

-- 用 in 查询有员工的部门：
select dept_name from dept where id in (select dept_id from emp);

-- 用 exists 查询有员工的部门：
select dept_name from dept where exists (select 1 from emp where dept.id=emp.dept_id);

-- 小数据集驱动大数据集的思想
-- 对于 in 来说，它是先执行子查询然后得到子查询的结果集，再用子查询的结果去匹配外部表。这样的话需要遍历一边刚刚的结果集，如果外部表的相应字段建立了索引的话，在匹配外部表的时候就能使用上外部表的索引了。假设子查询结果大小为 M，外部表的大小为 N，外部表使用 B+Tree 索引匹配每一条数据的时间复杂度是 O（log N），那么这个总的时间复杂度就相当于 O（M*log N）。

-- 对于 exists 来说，它是执行外表的遍历操作（不一定是全表扫描也可能是索引扫描，但是差别不是很大），然后里面的相关子查询会利用外部表的数据对内部表进行匹配，这个时候如果内部表的相关字段建立了索引的话，匹配的时候就能走索引了。同样假设子查询结果大小为 M，外部表的大小为 N，内部表使用 B+Tree 索引匹配每一条数据的时间复杂度是 O（log M），那么这个总的时间复杂度就相当于 O（N*log M）

-- not exists 和 not in 来说毫不犹豫的使用 not exists 
```



> 6.下面的查询也将导致全表扫描，若要提高效率，可以考虑全文检索

```mysql
select id from t where name like ‘%abc%’;
```

> 7.如果在 where 子句中使用参数，也会导致全表扫描。
>
> 因为 SQL 只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```mysql
select id from t where num = @num
#可以改为强制查询使用索引：
select id from t with(index(索引名)) where num = @num
```

> 8.应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```mysql
select id from t where num/2 = 100
#改为
select id from t where num = 100*2
```

> 9.应尽量避免在 where 子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描

```mysql
-– name以abc开头的id
select id from t where substring(name,1,3) = ’abc’    
-– '2005-11-30' 生成的id
select id from t where datediff(day,createdate,’2005-11-30′) = 0    
#修改
select id from t where name like 'abc%'
select id from t where createdate >= '2005-11-30' and createdate < '2005-12-1'
```

> 10.不要在 where 子句中的 “=” 左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引



> 11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。



> 12.不要写一些没有意义的查询，如需要生成一个空表结构：

```mysql
select col1,col2 into #t from t where 1=0
```

> 13.Update 语句，如果只更改 1、2 个字段，不要 Update 全部字段，否则频繁调用会引起明显的性能消耗，同时带来大量日志



> 14.对于多张大数据量（这里几百条就算大了）的表 JOIN，要先分页再 JOIN，否则逻辑读会很高，性能很差



> 15.select count (*) from table；
>
> 这样不带任何条件的 count 会引起全表扫描，并且没有任何业务意义，是一定要杜绝的



> 16.索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过 6 个，若太多则应考虑一些不常使用到的列上建的索引是否有 必要



> 17.应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引



> 18.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连 接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了



> 19.尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些



> 20.任何地方都不要使用 select * from t ，用具体的字段列表代替 “*”， 不要返回用不到的任何字段



> 21.尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）



> 22.在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先 create table，然后 insert



> 23.如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定



> 24.尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过 1 万行，那么就应该考虑改写



> 25.使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效



> 26.与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括 “合计” 的例程通常要比使用游标执行的速度快。如果开发时 间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好



> 27.在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息



> 28.尽量避免大事务操作，提高系统并发能力



> 29.尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理