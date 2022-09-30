# PHP

PHP是一种弱类型语言，在声明变量（变量名不能以数字开头）时不需要指定数据类型，且在赋值时会根据值的类型自动进行转换

PHP关键字不区分大小写

## php标记风格

- xml风格

  ```php
  <?php phpinfo()?>
  ```

- javascript风格

  ```php
  <script language="php">phpinfo()</script>
  ```

- 简洁风格

  ```php
  <?=phpinfo()?>
  ```

- 简短风格

  需要在php.ini中开启short_open_tags=on（默认关闭）

  ```php
  <?phpinfo()?>
  ```

- asp风格

  需要在php.ini开启asp_tags=on（默认关闭）

  ```php
  <%phpinfo%>
  ```

## PHP注释

- `//`或`#`单行注释
- `/**/`多行注释

## 数据类型

- 字符串

  字符串常用函数

  ```php
  strlen($str);  //返回字符串的长度（1中文=3个长度）
  strpos($str,"y");  //返回子串的第一次位置
  trim($str);  //去除两边空格
  substr($str,1,20); //字符串截取
  strstr($a,'abc');  //查询子串在该字符串中是否存在，是则返回该字符串及剩余部分，否则返回FALSE（区分大小写）
  str_replace('1','0',$str); //字符串替换(例将$str字符串中的1替换为0)
  explode('',$a);   //以指定字符分割字符串，并以数组形式输出
  md5($str);  //md5加密
  base64_encode($a);  //base64编码
  base64_decode($a);  //base64解码
  urlencode($a);  //url编码
  urldecode($a);  //url解码
  ```

- 布尔

- 整型

- 浮点型

- 数组

- 对象

- NULL

## 数组

在 PHP 中，有三种类型的数组：

* **数值数组** - 带有数字 ID 键（索引）的数组

  ```php
  $arr=array('a','b','c');
  print_r($arr);  //Array ( [0] => a [1] => b [2] => c )
  $arr[]=4; //添加
  ```

* **关联数组** - 带有指定的键的数组，每个键关联一个值

  ```php
  $arr=array('a'=>1,'b'=>2,'c'=>3);
  print_r($arr);  //Array ( [a] => 1 [b] => 2 [c] => 3 )
  $arr['d']=4; //添加(也可在数值数组中添加,相同键即为更改)
  ```

* **多维数组** - 包含一个或多个数组的数组

  ```php
  $arr=array(array(1,2),array('a','b'));
  print_r($arr);  //Array ( [0] => Array ( [0] => 1 [1] => 2 ) [1] => Array ( [0] => a [1] => b ) )
  ```

### 常用数组函数

```php
array_push($arr1,'abc','e'); //向数组尾部添加一个或多个元素（入栈），然后返回新数组的长度
array_pop($arr1);  //将数组最后一个元素弹出
array_splice($arr1,1,0,$arr2); //插入数组(数组,插入位置,删除个数(从插入位置往后开始删除),插入值)
unset($arr['b']); //删除数组中指定值/键值对
is_array($arr); //判断是否为数组
count($arr); //返回数组的长度
array_search(1,$arr); //在数组中搜索给定的值，如果成功则返回相应的键名
array_key_exists(0,$arr); //判断key是否存在，存在则返回1
in_array(); //判断值是否存在，存在则返回true（区分大小写）
array_unique(); //去除重复键和值
sort();  //升序排序
```

## 创建并调用函数

```php
function get_name(){
}
get_name(); #调用函数
```

### 变量作用域

* local（局部）

  函数内部声明的变量拥有 LOCAL 作用域，只能在函数内部进行访问

* global（全局）

  函数之外声明的变量拥有 Global 作用域，只能在函数以外进行访问

* static（静态）

  其值不会随着函数的调用和退出而发生变化（一直占用存储空间）


> 要在函数内部使用全局变量，可以使用global关键字

### 变量相关函数

```php
is_numeric  #检测变量是否为数字或数字字符串
is_array    #检测变量是否为数组
isset       #检测变量是否设置且为非NULL
unset       #释放给定的变量
gettype     #获取变量的类型
var_dump    #打印变量的相关信息
empty       #检测变量是否为空或为false
```

### 日期相关函数

```php 
checkdate(month,day,tear);  #验证日期是否有效
date_default_timezone_set("PRC");  #设置默认时区
date("Y-m-d");  #格式化一个本地时间并输出
```

## 类与对象

### PHP类定义

```php
class Person{
    //成员变量
    var $name;
    var $age;
    //成员函数
    function setName($name){
        $this->name=$name;
    }
    function getName(){
        return $this->name;
    }
    //析构函数（结束时自动执行）
    function __destruct(){
        echo "请设置一个name";
    }
}
$p1 = new Person();  //对象实例化
$p1->setName('lyh'); //调用成员函数
echo $p1->getName(); 
```

## PHP文件处理

### Include文件

include （或 require）语句会获取指定文件中存在的所有文本/代码/标记，并复制到使用 include 语句的文件中

```php
include 'demo.php';
include_once 'demo.php';
require 'demo.php';
require_once 'demo.php';
```

PS：include 和 require 语句是相同的，除了错误处理方面：

*   include 只生成警告（E\_WARNING），并且脚本会继续

*   require 会生成致命错误（E\_COMPILE\_ERROR）并停止脚本

### PHP文件打开/读取/写入/关闭

fopen()：打开文件（创建句柄的过程），有以下几种方式打开：

![](https://oos.luoyunhao.com/blog-img/202208220908115.png)

fread()：读取文件

filesize()：返回指定文件的大小

fclose()：关闭文件（打开文件占用内存，使用完需要关闭句柄）

```php
//读取文件并打印出来
$myfile = fopen('1.txt','r');
echo fread($myfile,filesize('1.txt'));
fclose($myfile);
```

fgets()：从文件中读取单行内容

feof()：检查是否已经到达文件末尾（EOF）

```php
//使用循环读取文件并打印出来
$myfile = fopen('1.txt','r');
while (!feof($myfile)){  
    echo fgets($myfile);  #句柄，每执行一次，指针向下移动一行
}
```

fwrite()：写入文件，第一个参数为文件名(或句柄)，第二个参数是被写的字符串

```php
$myfile = fopen('1.txt','w');
fwrite($myfile,"xxxxx");
```

### PHP文件复制/删除/重名

rename()：文件重命名

copy()：文件复制

unlink()：文件删除

```php
rename('1.txt','2.txt');  //文件重命名
copy('1.txt','2.txt'); //将1.txt复制到当前目录下，并命名为2.txt
unlink('1.txt');  //删除文件
```

### PHP文件判断

file\_exists() 判断文件是否存在

is\_file() 是否为文件

```php
var_dump(file_exists('1.txt')); //判断文件是否存在
var_dump(is_file('1.txt')); //判断是否为文件
```

### PHP目录操作

mkdir()：新建目录

rename()：移动/改名

rmdir()：删除目录

```php
mkdir("table"); //新建目录
rename('table','table1'); //移动/改名
rmdir('table1'); //删除目录
```

## PHP操作mysql

mysql_connect($servername,$username,$password)：打开一个到mysql数据库的连接（创建句柄）

mysql_select_db($dbname,$link)：选择数据库

mysql_error()：返回上一个mysql操作返回的错误信息

mysql_query($sql,$link)：执行mysql语句（返回一个结果集）

- 结果集操作函数

  mysql_fetch_array()：从结果集中取得一行作为关联数组，或数字数组
  mysql_fetch_object()：从结果集中取得一行作为对象
  mysql_num_rows()：获取结果集中行的数目
  mysql_free_result()：释放结果集

mysql_close($link)：关闭mysql连接（关闭句柄）
