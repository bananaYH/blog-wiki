## 会话控制

HTTP协议为无状态性，不会将多个事务保存起来，因此需要利用cookie或session来维持会话状态

### cookie

cookie保存在客户端浏览器中

> 默认保存30个cookie数，最大为4kb

cookie认证过程

1. 客户端登录成功后，服务端会利用setcookie函数生成cookie，然后放到set-cookie字段中再发送给客户端

   ```php
   setcookie(name,md5('admin'),time()+3600);  #设置一个cookie
   ```

2. 客户端接收到set-cookie字段后，会将cookie保存到浏览器中

3. 当cookie未失效时，客户端再次向该网站访问会添加上cookie字段发送给服务器

4. 服务器校验cookie合法性通过后，用户成功自动登录

销毁cookie

```php
unset($_COOKIE);
setcookie(name,null,time()-1);
```

### session

session保存在服务器中（通常保存在/var/lib/php/sess_sessionid，session文件中是序列化后的内容），用于记录当前会话信息，同样通过cookie字段发送。

> 默认20分钟有效，无长度限制

```php
session_start();  #启动新会话
$_SESSION['user']='admin';  #在会话中记录数据
unset($_SESSION['user']);  #销毁会话中指定变量
session_destroy();  #销毁会话（清除会话中所有数据）
```
