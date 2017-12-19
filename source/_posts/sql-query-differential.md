---
title: MySQL查询：获取两个表的差集
date: 2017-12-19 14:54:38
tags:
- MySQL
- 差积
categories:
- MySQL
---
场景：假设有`A`、`B`两表， `id` 为关联键
### 方法一
最好理解的方法
```sql
select * from A where A.id not in (select B.id from B)
```

### 方法二
最常用的方法
```sql
select * from A left join B on A.id = B.id where B.id is null 
```

### 方法三
最快的方法
```sql
select * from B where (select count(1) from A where A.id = B.id ) = 0
```
