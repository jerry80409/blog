---
title: 計算 MySql 資料量
cover: /img/home-bg.jpg
date: 2019-02-12 11:54:44
categories: ['mysql']
tags: ['mysql']
---
### 計算 Database 大小
```sql
SELECT TABLE_SCHEMA, SUM(DATA_LENGTH + INDEX_LENGTH)/1024/1024 "DATA_SIZE(MB)"
FROM INFORMATION_SCHEMA.TABLES 
GROUP BY TABLE_SCHEMA;
```

`TABLE_SCHEMA` 其實是資料庫
```
+-------------------------+---------------+
| TABLE_SCHEMA            | DATA_SIZE(MB) |
+-------------------------+---------------+
| employees               |  146.79687500 |
| information_schema      |    0.15625000 |
| mysql                   |    2.49447918 |
| performance_schema      |    0.00000000 |
| spring_transaction_demo |    0.00214386 |
| sys                     |    0.01562500 |
+-------------------------+---------------+
```

`GROUP BY` 可以替換成 `WHERE TABLE_SCHEMA=[your-data-base-name]`
```sql
SELECT TABLE_SCHEMA, SUM(DATA_LENGTH + INDEX_LENGTH)/1024/1024 "DATA_SIZE(MB)"
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA="employees";
```

```
+--------------+---------------+
| TABLE_SCHEMA | DATA_SIZE(MB) |
+--------------+---------------+
| employees    |  146.79687500 |
+--------------+---------------+
```

INFORMATION_SCHEMA.TABLES 可以提供的 [資訊](https://dev.mysql.com/doc/refman/5.7/en/tables-table.html)
* AVG_ROW_LENGTH
* DATA_LENGTH
* MAX_DATA_LENGTH
* INDEX_LENGTH
* DATA_FREE
* CHECKSUM

### 計算 Table 資料大小
```sql
SELECT TABLE_SCHEMA, TABLE_NAME, ROUND(DATA_LENGTH + INDEX_LENGTH)/1024/1024 "DATA_SIZE(MB)"
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA="employees";
```

```
+--------------+----------------------+---------------+
| TABLE_SCHEMA | TABLE_NAME           | DATA_SIZE(MB) |
+--------------+----------------------+---------------+
| employees    | current_dept_emp     |          NULL |
| employees    | departments          |    0.03125000 |
| employees    | dept_emp             |   17.03125000 |
| employees    | dept_emp_latest_date |          NULL |
| employees    | dept_manager         |    0.03125000 |
| employees    | employees            |   14.51562500 |
| employees    | salaries             |   95.62500000 |
| employees    | titles               |   19.56250000 |
+--------------+----------------------+---------------+
```

### 計算某個欄位所有資料量的大小
```sql
SELECT SUM(LENGTH(`first_name`))/1024/1024 "FIRST_NAME_COL_SIZE(MB)"
FROM employees;
```

```
+-------------------------+
| FIRST_NAME_COL_SIZE(MB) |
+-------------------------+
|              1.77847385 |
+-------------------------+
```

### LENGTH(str) 用途
> Returns the length of the string str, measured in bytes. A multibyte character counts as multiple bytes. This means that for a string containing five 2-byte characters, LENGTH() returns 10, whereas CHAR_LENGTH() returns 5.

用來計算 string 大小, 假設 string 有 5 個 characters, 每個 characters 為 2 bytes, 所以 `LENGTH()` 會得到 10, 而 `CHAR_LENGTH()` 則會得到 5。

```sql
SELECT LENGTH('hello_world')
```

```
+-----------------------+
| LENGTH('hello_world') |
+-----------------------+
|                    11 |
+-----------------------+
```

### Ref
* [MySql 5.7 Docs - https://dev.mysql.com/doc/refman/5.7/en/tables-table.html](https://dev.mysql.com/doc/refman/5.7/en/tables-table.html)
* [mkyong, calculate the mysql database size - https://www.mkyong.com/mysql/how-to-calculate-the-mysql-database-size/](https://www.mkyong.com/mysql/how-to-calculate-the-mysql-database-size/)
* [chartio - https://chartio.com/resources/tutorials/how-to-get-the-size-of-a-table-in-mysql/](https://chartio.com/resources/tutorials/how-to-get-the-size-of-a-table-in-mysql/)
