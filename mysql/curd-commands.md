
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
