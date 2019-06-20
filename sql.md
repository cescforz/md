sql

把一个表字段更新到另一个表

```sql
UPDATE t_company_change c,t_2b_base b set c.cus_type=b.cus_type WHERE c.cus_id=b.cus_id
```

```sql
mysql条件判断
IF(expr,v1,v2)函数,如果表达式expr成立，返回结果v1；否则，返回结果v2。
 
IFNULL(v1,v2)函数,如果v1的值不为NULL，则返回v1，否则返回v2。
 
CASE
 
　　WHEN e1　THEN v1
 
　　WHEN e2　THEN v2
 
　　...
 
　　ELSE vn
 
END
 
CASE表示函数开始，END表示函数结束。如果e1成立，则返回v1,如果e2成立，则返回v2，当全部不成立则返回vn，而当有一个成立之后，后面的就不执行了。

 
相当于java中的if else
 
select ename, case
 
   when salary<10000 then '实习生'
 
   when salary<15000 then '普通职员'
 
   when salary<100000 then '主管'
 
   else '高级主管'
 
   end
 
 from t_emp;
 
select ename ,case sex
 
     when '女' then '美女'
 
     when '男' then 'MAN'
 
     end AS "性别描述"
 
from t_emp;
 
相当于java 中的switch case
```

```
**4.0版本以下**，varchar(100)，指的是**100字节**，如果存放UTF8汉字时，只能存33个（每个汉字3字节） 

**5.0版本以上**，varchar(100)，指的是**100字符**，无论存放的是数字、字母还是UTF8汉字（每个汉字3字节），都可以存放100个。
```

```sql
一、包含中文字符
select * from 表名 where 列名 like '%[吖 - 座]%'
二、包含英文字符
select * from 表名 where 列名 like '%[a-z]%' 
三、包含纯数字
select * from 表名 where 列名 like '%[0-9]%'


```

