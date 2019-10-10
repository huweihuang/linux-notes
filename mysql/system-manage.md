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
