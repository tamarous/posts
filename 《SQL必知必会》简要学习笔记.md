##  《SQL必知必会》简要学习笔记
我主要学习的是 SQLite 的语法，实验用的数据库文件可以从[这里](http://forta.com/books/0672336073/TeachYourselfSQL_SQLite.zip)下载，使用的 SQL 工具是免费的 [SQLStudio](https://sqlitestudio.pl/)。

### 基本概念

主键：一列或一组列，其值用来**唯一**标记表中的每一行

### 简单查询

#### SELECT

从表格中检索某些列：

```sql lite
SELECT prod_name FROM Products;
SELECT prod_id, prod_name, prod_price FROM Products;
SELECT * FROM Products;
```

使用**DISTINCT** 可以去除重复的结果：

```sql lite
SELECT DISTINCT prod_id FROM Products;
```

取出前5个结果：

```
SELECT prod_id FROM Products LIMIT 5;
```

取出从第6行开始的5个结果：

```
SELECT prod_id FROM Products LIMIT 5 OFFSET 5;
```

#### ORDER BY

对检索的结果进行排序：

```
SELECT prod_name FROM Products ORDER BY prod_name;
SELECT prod_name, prod_price, prod_name FROM Products ORDER BY prod_price, prod_name;
```

按照列索引号进行排序：

```sql lite
-- 按照第二第三列进行排序
SELECT prod_id, prod_price, prod_name FROM Products ORDER BY 2,3; 
```

如果需要将结果进行降序排序，则需使用**DESC** 关键字：

```sql lite
SELECT prod_name FROM Products ORDER BY prod_name DESC;
SELECT prod_name, prod_price, prod_name FROM Products ORDER BY prod_price DESC, prod_name;
SELECT prod_name, prod_price, prod_name FROM Products ORDER BY prod_price DESC, prod_name DESC;
```

#### WHERE

对检索到的数据进行筛选：

```sql lite
SELECT prod_name, prod_price FROM Products WHERE prod_price = 3.49;
SELECT prod_name, prod_price FROM Products WHERE prod_price BETWEEN 5 AND 10;
SELECT prod_name FROM Products WHERE prod_price IS NULL;
SELECT prod_id, prod_price, prod_name FROM Products WHERE vend_id = 'DLL01' AND prod_price < 4;
SELECT prod_name, prod_price FROM Products WHERE vend_id = 'DLL01' OR vend_id = 'BRS01';
```

#### IN

用来制定条件范围：

```
SELECT prod_name, prod_price FROM Products WHERE vend_id IN ('DLL01', 'BRS01') ORDER BY prod_name;
```

#### NOT

否定后面所跟的任何条件：

```
SELECT prod_name FROM Products WHERE NOT vend_id = 'DLL01' ORDER BY prod_name;
```

#### Like

在搜索子句中使用通配符。

##### %

表示任意字符出现任意次数：

```sql lite
SELECT prod_id, prod_name FROM Products WHERE prod_name LIKE 'Fish%';
```

##### _

匹配单个字符：

```sql lite
SELECT prod_id, prod_name FROM Products WHERE prod_name LIKE '_ish';
```



#### GROUP BY

分组，可以将数据分为多个逻辑组，进而对每个组进行聚集计算。

```sql lite
-- 将表按照vend_id进行分组，然后分别显示组号和每组的行数
SELECT vend_id, COUNT(*) AS num_prods FROM Products GROUP BY vend_id;
```



#### HAVING

对分组进行过滤，相当于WHERE, 不过WHERE只能对行进行过滤：

```sql lite
-- 将表按照cust_id进行分组，然后从分组中选出数量大于2的分组并进行显示
SELECT cust_id, COUNT(*) AS orders FROM Orders GROUP BY cust_id HAVING COUNT(*) >= 2;
```



### 计算字段

对表中元素进行运算

```sql lite
-- ACCESS, SQL Server
SELECT vend_name + '(' + vend_country + ')' FROM Vendors ORDER BY vend_name;

-- ORACLE, SQLite
SELECT vend_name || '(' || vend_country || ')' FROM Vendors ORDER BY vend_name;

-- 去除字符串多余空格
SELECT TRIM(vend_name) || '(' || TRIM(vend_country) || ')' FROM Vendors ORDER BY vend_name;

-- 给计算结果取别名
SELECT TRIM(vend_name) || '(' || TRIM(vend_country) || ')' AS vend_title FROM Vendors ORDER BY vend_name;

-- 执行算数运算
SELECT prod_id, quantity, item_price, quantity * item_price AS expanded_price FROM OrderItems WHERE order_num = 28000;
```



### 使用函数处理数据

#### 文本处理

| 函数                     | 作用               |
| :--------------------- | ---------------- |
| LENGTH()               | 返回字符串的长度         |
| LOWER()/UPPER()        | 将字符串转成小写/大写      |
| LTRIM()/TRIM()/RTRIM() | 去除字符串左边/两边/右边的空格 |

#### 数值处理
| 函数     | 作用         |
| :----- | ---------- |
| ABS()  | 返回一个数的绝对值  |
| COS()  | 返回一个数的余弦值  |
| SIN()  | 返回一个数的正弦值  |
| TAN()  | 返回一个数的正切值  |
| SQRT() | 返回一个数的平方根值 |

#### 聚集函数
| 函数      | 作用       |
| :------ | -------- |
| AVG()   | 返回一列的平均值 |
| COUNT() | 返回一列的行数  |
| MAX()   | 返回一列的最大值 |
| MIN()   | 返回一列的最小值 |
| SUM()   | 返回一列的和   |

以上五个函数都可以对所有行进行运算，或者是只对不同的值进行运算:

```
SELECT AVG(DISTINCT prod_price) AS avg_price FROM Products WHERE vend_id = 'DLL01';
```

还可以对多个聚集函数进行组合：

```
SELECT COUNT(*) AS num_items, MIN(prod_price) AS price_min, MAX(prod_price) AS price_max, AVG(prod_price) AS price_avg FROM Products;
```



### 子查询

子查询即嵌套在其他查询中的查询。

```
SELECT cust_id FROM Orders WHERE order_num IN (SELECT order_num FROM OrderItems WHERE prod_id = 'RGAN01');
```

作为子查询的语句不能返回多个列。

子查询可以作为计算字段来使用：

```sql lite
SELECT cust_name, cust_state, (SELECT COUNT(*) FROM Orders WHERE Orders.cust_id = Customers.cust_id) AS orders FROM Customers ORDER BY cust_name;
```

### 联结

#### 内联结

联结是一种机制，用来在一条SELECT语句中关联表，因此称为联结

```sql lite
-- Vendors和Products表被WHERE语句联结在一起
SELECT vend_name, prod_name, prod_price FROM Vendors, Products WHERE Vendors.vend_id = Products.vend_id;
```

例子中的联结称为等值联结，也被称为内联结，它基于两个表之间的相等测试。

下面的语句与上面语句的作用完全等价：

```sql lite
SELECT vend_name, prod_name, prod_price FROM Vendors INNER JOIN Products ON Vendors.vend_id = Products.vend_id;
```

可以联结的表的数量没有限制，因此可以尽情使用WHERE或上面的形式联结多个表：

```sql lite
SELECT prod_name, vend_name, prod_price, quantity FROM OrderItems, Products, Vendors 
	WHERE Products.vend_id = Vendors.vend_id AND OrderItems.prod_id = Products.prod_id AND order_num = 20007;
```

#### 外联结

联结包含了那些在相关表中没有关联行的行，这种联结称为外联结，使用外联结时，必须用 LEFT 或者 RIGHT 来指定其包含所有行的表:

```
SELECT Customers.cust_id, Orders.order_num FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;
```

#### 高级联结

##### 表别名

SQL可以给表起别名，这样可以缩短SQL语句：

```sql lite
SELECT cust_name, cust_contact FROM Customers AS C, Orders AS O, OrderItems AS OI WHERE C.cust_id = O.cust_id AND OI.order_num = O.order_num AND prod_id = 'RGAN01';
```

##### 自联结

```
SELECT c1.cust_id, c1.cust_name, c1.cust_contact FROM Customers AS c1, Customers AS c2 WHERE c1.cust_name = c2.cust_name AND c2.cust_contact = 'Jim Jones'; 
```

### 组合查询
可以将多个查询的结果作为一个查询结果集返回:

```
SELECT cust_name, cust_country, cust_email FROM Customers WHERE cust_state IN ('IL', 'IN', 'MI') 
UNION SELECT cust_name, cust_contact, cust_email FROM Customers WHERE cust_name = 'Fun4ALL';
```

UNION 必须由两条或两条以上 SELECT 语句组成，且每条 SELECT 语句必须包含相同的列、表达式或聚集函数。

仍然可以使用 ORDER BY来对查询结果集进行排序，不过此时只能有一个 ORDER BY 语句，并且必须位于最后一条 SELECT 语句之后：

```
SELECT cust_name, cust_country, cust_email FROM Customers WHERE cust_state IN ('IL', 'IN', 'MI') 
UNION SELECT cust_name, cust_contact, cust_email FROM Customers WHERE cust_name = 'Fun4ALL' ORDER BY cust_name, cust_contact;
```

### 插入数据

```
-- 向表中插入一条数据
INSERT INTO Customers VALUES('1000000006','Toy Land', '123 Street', 'New York', 'NY','11111','USA',NULL,NULL);

-- 更安全的插入方法
INSERT INTO Customers(cust_id,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country,cust_contact,cust_email) VALUES('1000000006','Toy Land', '123 Street', 'New York', 'NY','11111','USA',NULL,NULL);

-- 省略某些行
INSERT INTO Customers(cust_id,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country) VALUES('1000000006','Toy Land', '123 Street', 'New York', 'NY','11111','USA');

-- 从检索出的结果中插入到表中
INSERT INTO Customers(cust_id,cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country) SELECT cust_id,cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country FROM CustNew;

-- 将一个表复制到另一个表
SELECT * INTO CustCopy FROM CustSource;
```

### 更新和删除数据

```
-- 更新某一列的信息
UPDATE Customers SET cust_email = 'kim@test.com' WHERE cust_id = '1000000005';

-- 更新某几列的信息
UPDATE Customers SET cust_contact = 'Sam Roberts' cust_email = 'sam@test.com' WHERE cust_id = '1000000006';

-- 删除某一行
DELETE FROM Customers WHERE cust_id = '100000006';

-- 删除整张表
DELETE FROM Customers;
```

### 创建和操纵表

```

-- 创建一张表
CREATE TABLE Products 
(
    prod_id   CHAR(10)  NOT NULL,
    vend_id   CHAR(10)  NOT NULL,
    prod_name CHAR(254) NOT NULL,
    prod_price DECIMAL(8,2) NOT NULL,
    prod_desc  VARCHAR(1000) NULL
);

-- 指定默认值
CREATE TABLE Products 
(
    prod_id   CHAR(10)  NOT NULL,
    vend_id   CHAR(10)  NOT NULL,
    prod_name CHAR(254) NOT NULL,
    prod_price DECIMAL(8,2) NOT NULL DEFAULT 5,
    prod_desc  VARCHAR(1000) NULL
);

-- 更新一张表，为表增加一列
ALTER TABLE Vendors ADD vend_phone CHAR(20);

-- 更新一张表，删除一列
ALTER TABLE Vendors DROP COLUMN vend_phone;

-- 删除表
DROP TABLE CustCopy;

-- 重命名一张表
ALTER TABLE table_old RENAME TO table_new;





