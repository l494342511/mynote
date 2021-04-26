# 数据库多表查询 JOIN

## 1. JOIN

join的作用是，通过一个属性连接两张表，具体有三种使用方法：

### 1.1 内连接 INNER JOIN

用法：

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Persons.Id_P=Orders.Id_P
```

INNER JOIN 根据同一个属性连接两个表，并且查出两张表相匹配的数据。如若A表有id为1的记录，而B表也有id为1的记录，则将两条记录连接并返回。若A表有id为1的记录，而B表没有id为1的记录，则不返回该记录。

### 1.1 左连接 LEFT JOIN

内连接是公共属性找出两个表中匹配的记录，而左连接是以左表为主表，通过左表的属性，查询相匹配的右表中的属性对左表进行补充和扩充，若在右表中没有与左表记录相对应的数据，则只返回左表的记录。

用法：

```sql
SELECT column_name(s)
FROM table_name1
LEFT JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```

### 1.2 右连接 RIGHT JOIN

与左连接类似；