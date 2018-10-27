# 1. 系统管理

## 1.1. 连接mysql

格式： mysql -h主机地址 -u用户名 －p用户密码

```bash
#连接本地
mysql -h<localhost/127.0.0.1> -u用户名 －p用户密码

#连接远程
mysql -h<主机地址> -u用户名 －p用户密码

#退出连接
exit
```

## 1.2. 备份数据库

**1.导出整个数据库**

导出文件默认是存在mysql\bin目录下

```bash
#1）备份单个数据库
mysqldump -u 用户名 -p 数据库名 > 导出的文件名

mysqldump -u user_name -p123456 database_name > outfile_name.sql

#2）同时备份多个数据库，例如database1_name，database2_name
mysqldump -u user_name -p123456 --databases database1_name database2_name > outfile_name.sql

#3）备份全部数据库
mysqldump -u user_name -p123456 --all-databases > outfile_name.sql
```

**2.导出一个表**

```bash
mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名
mysqldump -u user_name -p database_name table_name > outfile_name.sql
```

**3.导出一个数据库结构**

```bash
mysqldump -u user_name -p -d –add-drop-table database_name > outfile_name.sql

-d 没有数据 –add-drop-table 在每个create语句之前增加一个drop table
```

**4.带语言参数导出**

```bash
mysqldump -uroot -p –default-character-set=latin1 –set-charset=gbk –skip-opt database_name > outfile_name.sql
```

**5、导入数据库**

```bash
#1）多个个数据库
mysql -u root –p < [备份文件的保存路径] 或者source [备份文件的保存路径]

#2）单个数据库
mysql -uroot –p database_name < [备份文件的保存路径] 或者source [备份文件的保存路径]
```

## 1.3. 用户管理

```sql
#创建用户
create user '用户名'@'IP地址' identified by '密码';

#删除用户
drop user '用户名'@'IP地址';
delete from user where user='用户名' and host='localhost';

#修改用户
rename user '用户名'@'IP地址'; to '新用户名'@'IP地址';;

#修改密码
set password for '用户名'@'IP地址' = Password('新密码')
mysqladmin -u用户名 -p旧密码 password 新密码
```

## 1.4. 权限管理

### 1.4.1. grant

**1、grant 权限 on 数据库对象 to 用户**

数据库对象的格式为`<database>.<table>`。`<database>.*`：表示授权数据库对象该数据库的所有表；`*.*`：表示授权数据库对象为所有数据库的所有表。

```sql
grant all privileges on . to <user>@'<ip>' identified by '<passwd>';如果<ip>为'%'表示不限制IP。
```

**2、撤销权限**：

```sql
revoke all on . from <user>@<ip>; 
```

### 1.4.2. 普通数据库用户

查询、插入、更新、删除 数据库中所有表数据的权利

```sql
grant select, insert, update, delete on testdb.* to <user>@'<ip>';
```

### 1.4.3. DBA 用户

```sql
#1、授权
grant all privileges on . to <dba>@'<ip>' identified by '<passwd>';

#2、刷新系统权限
flush privileges;
```

### 1.4.4. 查看用户权限

```sql
#查看当前用户（自己）权限
show grants;

#查看指定MySQL 用户权限
show grants for <user>@<localhost>;

#查看user和host
select user,host from mysql.user order by user;
```

### 1.4.5. 权限列表

| 权限             | 说明                           | 网站使用账户是否给予 |
| ---------------- | ------------------------------ | -------------------- |
| Select           | 可对其下所有表进行查询         | 建议给予             |
| Insert           | 可对其下所有表进行插入         | 建议给予             |
| Update           | 可对其下所有表进行更新         | 建议给予             |
| Delete           | 可对其下所有表进行删除         | 建议给予             |
| Create           | 可在此数据库下创建表或索引     | 建议给予             |
| Drop             | 可删除此数据库及数据库下所有表 | 不建议给予           |
| Grant            | 赋予权限选项                   | 不建议给予           |
| References       | 未来MySQL特性的占位符          | 不建议给予           |
| Index            | 可对其下所有表进行索引         | 建议给予             |
| Alter            | 可对其下所有表进行更改         | 建议给予             |
| Create_tmp_table | 创建临时表                     | 不建议给予           |
| Lock_tables      | 可对其下所有表进行锁定         | 不建议给予           |
| Create_view      | 可在此数据下创建视图           | 建议给予             |
| Show_view        | 可在此数据下查看视图           | 建议给予             |
| Create_routine   | 可在此数据下创建存储过程       | 不建议给予           |
| Alter_routine    | 可在此数据下更改存储过程       | 不建议给予           |
| Execute          | 可在此数据下执行存储过程       | 不建议给予           |
| Event            | 可在此数据下创建事件调度器     | 不建议给予           |
| Trigger          | 可在此数据下创建触发器         | 不建议给予           |

### 1.4.6.查看主从关系

```sql
#登录主机
show slave hosts;
#登录从机
show slave status;
```

# 2. 数据库操作

```sql
#创建数据库
create database <数据库名>

#显示数据库
show databases

#删除数据
drop database <数据库名>
```

# 3. 数据表操作

## 3.1. 创建表

```sql
create table 表名(

    列名  类型  是否可以为空，

    列名  类型  是否可以为空

)ENGINE=InnoDB DEFAULT CHARSET=utf8
```

- 默认值，创建列时可以指定默认值，当插入数据时如果未主动设置，则自动添加默认值
- 自增，如果为某列设置自增列，插入数据时无需设置此列，默认将自增（表中只能有一个自增列）注意：1、对于自增列，必须是索引（含主键）2、对于自增可以设置步长和起始值
- 主键，一种特殊的唯一索引，不允许有空值，如果主键使用单个列，则它的值必须唯一，如果是多列，则其组合必须唯一。

## 3.2. 查看表

```sql
show tables;                    # 查看数据库全部表
select * from 表名;             # 查看表所有内容
```

## 3.3. 删除表

```sql
drop table 表名
```

## 3.4. 清空表内容

```sql
delete from 表名
truncate table 表名
```

## 3.5. 查看表结构

```sql
desc 表名
```

## 3.6. 修改表

**列操作**

```sql
#添加列  
alter table 表名 add 列名 类型

#删除列   
alter table 表名 drop column 列名

#修改列
alter table 表名 modify column 列名 类型;  -- 类型
alter table 表名 change 原列名 新列名 类型; -- 列名，类型
```

**主键操作**

```sql
#添加主键
alter table 表名 add primary key(列名);

#删除主键
alter table 表名 drop primary key;
alter table 表名  modify  列名 int, drop primary key;

#修改主键：先删除后添加
alter table 表名 drop primary key;
alter table 表名 add primary key(列名);


#添加外键
alter table 从表 add constraint 外键名称（形如：FK从表主表） foreign key 从表(外键字段) references 主表(主键字段);

#删除外键
alter table 表名 drop foreign key 外键名称
```

**默认值操作**

```sql
#修改默认值：
ALTER TABLE testalter_tbl ALTER i SET DEFAULT 1000;

#删除默认值：
ALTER TABLE testalter_tbl ALTER i DROP DEFAULT;
```

**调整表结构字段顺序**

```sql
alter table <table_name> modify <字段1> varchar(10) after <字段2>;
alter table <table_name> modify id int(10) unsigned auto_increment first;
```

# 4. 表内容操作

## 4.1. 增

```sql
insert into 表 (列名,列名...) values (值,值,...)
insert into 表 (列名,列名...) values (值,值,...),(值,值,值...)
insert into 表 (列名,列名...) select (列名,列名...) from 表

例：
insert into tab1(name,email) values('zhangyanlin','zhangyanlin8851@163.com')
```

## 4.2. 删

```sql
delete from 表                                      # 删除表里全部数据
delete from 表 where id＝1 and name＝'zhangyanlin'   # 删除ID =1 和name='zhangyanlin' 那一行数据
```

## 4.3. 改

```sql
update 表 set name ＝ 'zhangyanlin' where id>1
```

## 4.4. 查

```sql
select * from 表
select * from 表 where id > 1
select nid,name,gender as gg from 表 where id > 1
```

## 4.5. 条件判断

### 4.5.1. where

```sql
select * from <table> where id >1 and name!='huwh' and num =12;
select * from <table> where id between 5 and 6;
select * from <table> where id in (11,22,33);
select * from <table> where id not in (11,22,33);
select * from <table> where id in (select nid from <table>)
```

### 4.5.2. 通配符like

```sql
select * from <table> where name like 'hu%';   #hu开头
select * from <table> where name like 'hu_'    #hu开头后接一个字符
```

### 4.5.3. 限制limit

```sql
select * from <table> limit 5;   #前5行
select * from <table> limit 4,5  #从第四行开始的5行
select * from <table> limit 5 offset 4;#从第四行开始的5行
```

### 4.5.4. 排序asc，desc

```sql
select * from <table> order by 列 asc;            #跟据“列”从小到大排序（不指定默认为从小到大排序）
select * from <table> order by 列 desc;           #根据“列”从大到小排序
select * from <table> order by 列1 desc,列2 asc;  #根据“列1”从大到小排序，如果相同则按“列2”从小到大排序
```

### 4.5.5. 分组group by

group by 必须在where之后，order by之前。

```sql
select num,from <table> group by num;     
select num,nid from <table> group by num,nid;
select num from <table> where nid > 10 group by num,nid order nid desc;
select num,nid,count(*),sum(score),max(score) from <table> group by num;
select num from <table> group by num having max(id) > 10;
select num from <table> group by num;
```
