## mysql

### mysql安装

```zsh
yum install -y mariadb mariadb-server
mysql_secure_installation  #安全初始化安装
```

### mysql忘记密码处理

1. 修改文件/etc/my.cnf，在[mysqld]下面加上skip-grant-tables，并重启mysql

2. mysql -u root -p直接登录mysql

3. 更改root密码，并刷新权限

   ```mysql
   use mysql;
   update user set password=password('root') where user='root';  #内置password加密算法：sha1(unhe(xsha1('root')))
   flush privleges;
   ```

### SQL创建数据表CREATE TABLE语句

```sql
CREATE TABLE [IF NOT EXISTS] <表名>(
	字段名1 数据类型 [属性] [索引],
	字段名2 数据类型 [属性] [索引],
	···
) [储存引擎] [表字符集];
```

> 属性主要包括：AUTO_INCREMENT、NOT NULL、DEFAULT等
> 索引主要包括：PRIMARY KEY、UNIQUE、FOREIGN KEY、INDEX等子句
> 储存引擎：格式(ENGINE=InnoDB) MySQL默认储存引擎为InnoDB
> 表字符集：格式(DEFAULT CHARSET=utf8mb4)

### SQL修改表结构ALTER TABLE语句

```sql
ALTER TABLE 表名
	ADD 字段名 数据类型 [属性] [索引] [FIRST | AFTER 字段名]
	MODIFY 字段名 数据类型 [属性] [索引]
	CHANGE 字段名 新字段名 数据类型 [属性] [索引]
	DROP 字段名
	RENAME AS 新表名
```

> ADD 用来添加一个新的字段，如果没有指定FIRST或AFTER，则在表的列尾添加一个字段
> MODIFY 用来更改指定字段的数据类型等
> CHANGE 在更改指定字段的数据类型时，可以修改字段名
> DROP 用来删除字段
> RENAME AS 用来给数据表重新命名

### SQL插入表数据INSERT语句

```sql
INSERT [INTO] <表名> [(字段名1,字段名2,···)] VALUES(值1,值2,···);
```

### SQL修改表数据UPDATE语句

```sql
UPDATE <表名> SET 字段名1=值1 [,字段名2=值2,···]
[WHERE 条件];
```

### SQL删除表数据DELETE语句

```sql
DELETE FROM <表名>
[WHERE 条件];
```

### SQL查询SELECT语句

```sql
SELECT [ALL|DISTINCT] * | 字段列表 FROM 表名
[WHERE 查询条件]
[GROUP BY 分组字段 [HAVING 分组条件]]
[ORDER BY 排序字段 [ASC|DESC]]
[LIMIT [初始位置,] 记录数];
```
