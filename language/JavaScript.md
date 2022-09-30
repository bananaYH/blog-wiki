## JavaScript

### 数据类型

- 数值型：number（凡是数字都是数值型，不区分整数和小数）
- 字符串：string（凡是引号包裹起来的内容全部都是字符串）
- 布尔：boolean（true、false）
- 索引数组：key为0，后面依次加一
- 关联数组：键值对
- 对象类型：object（特殊取值null，数组 字典）
- 未定义型：undefined

> 可以使用typeof判断数据的类型

### 数组

- 数组内可以存放任意数据类型的数据（本质上它也是对象）
- 数组元素不赋值的情况下值为undefined

#### 定义数组

```js
var arr=[]  //定义一个空数组
var arr=new Array();  //定义一个空数组
var arr = new Array(10,20,{"name":"lyh","age":19},0.1,"string",true,["aaa","bbb"])  //定义的同时赋值
```

#### 常用数组操作

```js
obj.length          //数组的大小
obj.push(ele)       //尾部插入元素
obj.pop()           //尾部弹出一个元素
obj.sort( )         //对数组元素进行排序
obj.join(sep)       //将数组元素连接起来以构建一个字符串
obj.concat(val,..)  //连接数组
obj.splice(start, deleteCount, value, ...)  //插入、删除或替换数组的元素
```

### 函数

#### 普通函数

```JavaScript
function get_name(){
}
```

#### 匿名函数

没有名字的函数称为匿名函数

```JavaScript
//每隔6秒输出1次1
<script>
  setInterval(function(){
    console.log(1);
  },6000);
</script>
```

#### 自执行函数

创建函数并且自动执行

```JavaScript
<script>
  (function(name){
    console.log(name);
  })('lyh');  //传实参(没有形参也要有括号)
</script>
```

#### 利用事件调用函数

- onclick（点击触发）
- onload（页面或图像加载完后触发）
- onunload（退出页面时触发）
- onsubmit（表单提交时触发）
- onkeydown（按下键触发）
- onkeyup（按上键触发）
- onmouseover（鼠标移动到该元素上时触发）
- onmouseout（鼠标移出元素时触发）
- oninput（vlaue值发生变化时触发）
- onchange（失去焦点且value值发生变化时触发）
- onblur（失去焦点时触发）

```html
例如：<input type="button" onclick=get_name() value="调用函数"/>
```

### 自定义对象

定义方式一：

```JavaScript
var person = new Objext();  //声明一个自定义对象
person.name="bananaYH";  //给person对象添加属性
//输出1
function getname(user){
  document.write(person[user]);
}
getname('name');
//输出2
document.write(person['name']);  //输出对象属性值
```

定义方式二：

```JavaScript
var person ={
  name:bananaYH,
  getname:function(){
    return this.name;
  }
}
document.write(person.name);  //输出一
document.write(person.getname());  //输出二
```

### DOM操作对象

document是获取节点的基本方法（将html作为文档来进行操作）

#### 打印内容到页面

```CSS
document.write('username is Hao');
```

#### 查找元素

```JavaScript
document.getElementById('myid');  //通过id来获取元素,返回指定的唯一元素
document.getElementsByName("myname"); //通过name来获取元素，返回name='myname'的集合
document.getElementsByClassName("classname")  //用classname来获取元素，返回的是一个class="classname"的集合(不兼容IE8及以下)
document.getElementsByTagName('div'); //用元素的标签获取元素，返回所有标签=“div”的集合
```

直接查找

```JavaScript
var demo = document.getElementById('div1');
```

间接查找（获取内容）

- innerText仅文本

```JavaScript
document.getElementById('div1').innerText;
document.getElementById('div1').innerText="内容"; //修改
```

- innerHTML全内容

```JavaScript
document.getElementById('div1').innerHTML;
document.getElementById('div1').innerHTML="内容"; //修改
```

- value（例input标签和select标签中的value值）

```JavaScript
document.getElementById('myinupt').value; //获取input输入框中的内容
document.getElementById('myinupt')[0].value; //获取第一个输入框中的内容
document.getElementById('myinupt').value='aaaaa'; //修改输入框中的内容
```
