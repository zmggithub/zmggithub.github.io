### select语句的执行顺序是（C）---
（1）select（2）from（3）where（4）group by（5）having（6）order by
- A.	123456
- B.	234561
- C.	234516
- D.	124563

> 说明：
select 后面尽量不写*，写出具体的列名，提高性能。
group by分组，此时select 字段只能是group by后面的分组字段或者聚合函数（count、max、min、avg、sum）
having跟在groupby 后面，对分组进行条件过滤
order by 默认asc升序，desc降序


>大于18岁人，知道每个年龄段大于5人的年龄 count(*)>5
语义分析，
prepareStatemen和statement区别：前者还行重复代码时，不用语义分析，可以直接执行。后者每一次携带的sql语句都要进行语义分析，影响执行性能。

> SQL语言不同于其他编程语言的最明显特征是处理代码的顺序。在大多数据库语言中，代码按编码顺序被处理。但在SQL语句中，第一个被处理的子句是FROM，而不是第一出现的SELECT。SQL查询处理的步骤序号：

   ```mysql
(8) SELECT (9) DISTINCT (11) <TOP_specification> <select_list>
(1) FROM <left_table>
(3) <join_type> JOIN <right_table>
(2) ON <join_condition>
(4) WHERE <where_condition>
(5) GROUP BY <group_by_list>
(6) WITH {CUBE | ROLLUP}
(7) HAVING <having_condition>
(10) ORDER BY <order_by_list>
   ```

> 以上每个步骤都会产生一个虚拟表，该虚拟表被用作下一个步骤的输入。这些虚拟表对调用者(客户端应用程序或者外部查询)不可用。只有最后一步生成的表才会会给调用者。如果没有在查询中指定某一个子句，将跳过相应的步骤。
逻辑查询处理阶段简介：
>
1、 FROM：对FROM子句中的前两个表执行笛卡尔积(交叉联接)，生成虚拟表VT1。表名执行顺序是从后往前，所以数据较少的表尽量放后。
2、 ON：对VT1应用ON筛选器，只有那些使为真才被插入到TV2。
3、 OUTER (JOIN):如果指定了OUTER JOIN(相对于CROSS JOIN或INNER JOIN)，保留表中未找到匹配的行将作为外部行添加到VT2，生成TV3。如果FROM子句包含两个以上的表，则对上一个联接生成的结果表和下一个表重复执行步骤1到步骤3，直到处理完所有的表位置。
4、 WHERE：对TV3应用WHERE筛选器，只有使为true的行才插入TV4。执行顺序为从前往后或者说从左到右。
5、 GROUP BY：按GROUP BY子句中的列列表对TV4中的行进行分组，生成TV5。执行顺序从左往右分组。
6、 CUTE|ROLLUP：把超组插入VT5，生成VT6。
7、 HAVING：对VT6应用HAVING筛选器，只有使为true的组插入到VT7。Having语句很耗资源，尽量少用　　    8、 SELECT：处理SELECT列表，产生VT8。
9、 DISTINCT：将重复的行从VT8中删除，产品VT9。
10、ORDER BY：将VT9中的行按ORDER BY子句中的列列表顺序，生成一个游标(VC10)。执行顺序从左到右，是一个很耗资源的语句。
11、TOP：从VC10的开始处选择指定数量或比例的行，生成表TV11，并返回给调用者。

### 创建索引的规则
索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。
索引的使用规范：索引的创建要与应用结合考虑，建议大的OLTP表不要超过6个索引；尽可能的使用索引字段作为查询条件，尤其是聚簇索引，必要时可以通过index index_name来强制指定索引；避免对大表查询时进行table scan，必要时考虑新建索引；在使用索引字段作为条件时，如果该索引是联合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用；要注意索引的维护，周期性重建索引，重新编译存储过程。

索引创建规则：
表的主键、外键必须有索引；  
数据量超过300的表应该有索引；  
经常与其他表进行连接的表，在连接字段上应该建立索引；  
经常出现在Where子句中的字段，特别是大表的字段，应该建立索引；  
索引应该建在选择性高的字段上；  
索引应该建在小字段上，对于大的文本字段甚至超长字段，不要建索引；  
复合索引的建立需要进行仔细分析，尽量考虑用单字段索引代替；  
正确选择复合索引中的主列字段，一般是选择性较好的字段；
复合索引的几个字段是否经常同时以AND方式出现在Where子句中？单字段查询是否极少甚至没有？如果是，则可以建立复合索引；否则考虑单字段索引；  
如果复合索引中包含的字段经常单独出现在Where子句中，则分解为多个单字段索引；  
如果复合索引所包含的字段超过3个，那么仔细考虑其必要性，考虑减少复合的字段；  
如果既有单字段索引，又有这几个字段上的复合索引，一般可以删除复合索引；  
频繁进行数据操作的表，不要建立太多的索引；  
删除无用的索引，避免对执行计划造成负面影响；  
表上建立的每个索引都会增加存储开销，索引对于插入、删除、更新操作也会增加处理上的开销。另外，过多的复合索引，在有单字段索引的情况下，一般都是没有存在价值的；相反，还会降低数据增加删除时的性能，特别是对频繁更新的表来说，负面影响更大。  
尽量不要对数据库中某个含有大量重复的值的字段建立索引。

SQL语句性能优化	--- 
1，对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。  
   2，应尽量避免在 where 子句中对字段进行 null 值判断，创建表时NULL是默认值，但大多数时候应该使用NOT NULL，或者使用一个特殊的值，如0，-1作为默 认值。
   3，应尽量避免在 where 子句中使用!=或<>操作符， MySQL只有对以下操作符才使用索引：<，<=，=，>，>=，BETWEEN，IN，以及某些时候的LIKE。  
   4，应尽量避免在 where 子句中使用 or 来连接条件， 否则将导致引擎放弃使用索引而进行全表扫描， 可以 使用UNION合并查询： select id from t where num=10 union all select id from t where num=20  
   5，in 和 not in 也要慎用，否则会导致全表扫描，对于连续的数值，能用 between 就不要用 in 了：Select id from t where num between 1 and 3  
   6，下面的查询也将导致全表扫描：select id from t where name like ‘%abc%’ 或者select id from t where name like ‘%abc’若要提高效率，可以考虑全文检索。而select id from t where name like ‘abc%’ 才用到索引  
   7， 如果在 where 子句中使用参数，也会导致全表扫描。  
   8，应尽量避免在 where 子句中对字段进行表达式操作，尽量避免在where子句中对字段进行函数操作  
   9,很多时候用 exists 代替 in 是一个好的选择： select num from a where num in(select num from b).用下面的语句替换： select num from a where exists(select 1 from b where num=a.num)  
   10,尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。  
   11，尽可能的使用 varchar/nvarchar 代替 char/nchar ， 因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。  
   12，最好不要使用*返回所有： select * from t ，用具体的字段列表代替“*”。  
   13，使用表的别名(Alias)：当在SQL语句中连接多个表时,请使用表的别名并把别名前缀于每个Column上.这样一来,就可以减少解析的时间并减少那些由Column歧义引起的语法错误。  
   14，常见的简化规则如下：不要有超过5个以上的表连接（JOIN），考虑使用临时表或表变量存放中间结果。少用子查询，视图嵌套不要过深,一般视图嵌套不要超过2个为宜。  
   15，在IN后面值的列表中，将出现最频繁的值放在最前面，出现得最少的放在最后面，减少判断的次数。  
   16，尽量使用exists代替select count(1)来判断是否存在记录，count函数只有在统计表中所有行数时使用，而且count(1)比count(*)更有效率。  
   17，下列SQL条件语句中的列都建有恰当的索引，但执行速度却非常慢：
   ```mysql
      SELECT * FROM record WHERE substrINg(card_no,1,4)=’5378’ (13秒)  
      SELECT * FROM record WHERE amount/30< 1000 （11秒）  
      SELECT * FROM record WHERE convert(char(10),date,112)=’19991201’ （10秒）  
   ```

   分析：  
   WHERE子句中对列的任何操作结果都是在SQL运行时逐列计算得到的，因此它不得不进行表搜索，而没有使用该列上面的索引；如果这些结果在查询编译时就能得到，那么就可以被SQL优化器优化，使用索引，避免表搜索，因此将SQL重写成下面这样：
   ```mysql
         SELECT * FROM record WHERE card_no like ‘5378%’ （< 1秒）  
         SELECT * FROM record WHERE amount< 1000*30 （< 1秒）
         SELECT * FROM record WHERE date= ‘1999/12/01’ （< 1秒） 
   ```

   18，选择最有效率的表名顺序(只在基于规则的优化器中有效)：
   oracle 的解析器按照从右到左的顺序处理FROM子句中的表名，FROM子句中写在最后的表(基础表 driving table)将被最先处理，在FROM子句中包含多个表的情况下,你必须选择记录条数最少的表作为基础表。如果有3个以上的表连接查询, 那就需要选择交叉表(intersection table)作为基础表, 交叉表是指那个被其他表所引用的表.  
   19，提高GROUP BY语句的效率, 可以通过将不需要的记录在GROUP BY 之前过滤掉.下面两个查询返回相同结果，但第二个明显就快了许多.  
   低效:  
   ```mysql
      SELECT JOB , AVG(SAL)
      FROM EMP
      GROUP BY JOB
      HAVING JOB =’PRESIDENT’
          OR JOB =’MANAGER’
   ```  

高效:  

```mysql
   SELECT JOB , AVG(SAL)
   FROM EMP
   WHERE JOB =’PRESIDENT’
      OR JOB =’MANAGER’
   GROUP BY JOB
   ```

   20，mysql查询优化总结：使用慢查询日志去发现慢查询，使用执行计划去判断查询是否正常运行，总是去测试你的查询看看是否他们运行在最佳状态下。久而久之性能总会变化，避免在整个表上使用count(*),它可能锁住整张表，使查询保持一致以便后续相似的查询可以使用查询缓存
   ，在适当的情形下使用GROUP BY而不是DISTINCT，在WHERE, GROUP BY和ORDER BY子句中使用有索引的列，保持索引简单,不在多个索引中包含同一个列，有时候MySQL会使用错误的索引,对于这种情况使用USE INDEX，检查使用SQL_MODE=STRICT的问题，对于记录数小于5的索引字段，在UNION的时候使用LIMIT不是是用OR。


### Mysql和Oracle的不同
- 1.数据类型定义不同，字符串 varchar(mysql), varchar2(oracle)
数值  decimal(mysql), number(oracle)
- 2.事务处理机制不同，mysql默认自动提交增删改查的操作，oracle手动提交。
- 3.分页语句不同，mysql中使用limit, oracle使用rownum
- 4.主键自增长的方式不同
mysql auto_increment, 主键必须是数值型。
oracle: sequence 序列。主键可以是数值也可以是字符串。
- 5.oracle支持集合操作，mysql部分支持
mysql 中没有intersect, minus
- 6.oracle支持check约束，mysql不支持
age的范围，oracle 可以约束，不满足约束的记录不允许插入。
- 7.oracle支持全外连接，mysql只支持左外连接和右外连接。
full (outer) join    ----全外连接   o
left (outer) join     ----左        o    m
righnt (outer) join   ----右o    m

- 8.数据类型不同
mysql：
数值类型
oracle: number(整数n,整数m)：存储有效数字n为，m为小数
mysql: decimal(M,d): 小数
字符类型
oracle: varchar2(n)：n为存储的字节数
mysql:varchar(n): 最多可存储的字符数
- 9.语句
关于delete
Oracle 中的语法是delete from, from 可以省略。
MySQL 中, 不能省略from
delete from 表名 where条件
约束constraint，mysql中没有s，oracle中可以有s也可以没有s
- 10.关于外连接
MySQL 中不能使用(+)符号, 所以在外连接时, 应该使用SQL99 的语法.
select * from a left join b on a.id=b.fid;

Oracle:
SQL92: select * from a,b where a.id=b.fid(+);
where后 有(+)是附加表，没有（+）的是主表
SQL99:和上述一样用法
- 11.自增长机制
MySQL中支持auto_increment
create table ss(id int primary key auto_increment)在给表插入数据时，不用给主键赋值，自动增加。
Oracle使用序列实现自增长机制，不支持auto_increment;
create sequence seq_表名_主键;
添加数据时，主键的值是seq_表名_主键.nextval
