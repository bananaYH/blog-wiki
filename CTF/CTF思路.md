# CTF思路

## 文件操作与隐写

### 识别文件

首先当文件没有后缀名或者有后缀名而无法打开时，就需要识别出文件类型

1、File命令

格式：file 文件路径

![image-20210813145616856](https://oos.luoyunhao.com/blog-img/20210813145623.png)

> 结果为data说明文件可能存在头部残缺或字段错误

2、winhex工具

windows下通过文件头信息判断文件类型

常见文件头类型如下图所示：

![image-20210814165605774](https://oos.luoyunhao.com/blog-img/20210814165605.png)

## 文件可能会出现的情况

### 1、文件无法正常打开（头部残缺/字段错误）

- 头部头部残缺

  使用winhex添加相应的文件头

- 头部字段错误

  可以找一个相同类型的文件进行替换

### 2、文件由多个文件合并而成（文件分离）

- 使用Binwalk工具

  Binwalk是Linux下用来分析和分离文件的工具

  命令格式：

  ```zsh
  binwalk 文件名               //分析文件
  binwalk -e 文件名            //分离文件
  ```

  ![image-20210813153456277](https://oos.luoyunhao.com/blog-img/20210813153456.png)

- 使用foremost工具

  如果Binwalk无法正确分离出文件，可以尝试使用foremost

  命令格式：

  ```zsh
  foremost 文件名 -o 输出目录名
  ```

  > -o：设置输出目录

  ![image-20210813154543188](https://oos.luoyunhao.com/blog-img/20210813154543.png)

- 使用dd工具

  当文件自动分离出错或者因为其他原因无法自动分离时，可以先使用Binwalk分析，再使用dd实现文件手动分离

  命令格式：

  ```zsh
  dd if=源文件 of=输出文件名 bs=1 count=1 skip=开始分离的字节数
  ```

  > bs=bytes：同时设置读写块的大小为bytes，可代替ibs和obs
  >
  > count：设置读写块的个数
  >
  > skip=blocks：从输入文件开头跳过blocks个块后再开始复制

  例1：

  ![image-20210813160419613](https://oos.luoyunhao.com/blog-img/20210813160419.png)

  ![image-20210813160341716](https://oos.luoyunhao.com/blog-img/20210813160341.png)

  ![image-20210813160355607](https://oos.luoyunhao.com/blog-img/20210813160355.png)

  例2：

  ![](https://oos.luoyunhao.com/blog-img/20210813161820.png)

- 使用010Editor分离文件

  选中文件16进制后保存为新文件即可

  ![image-20210813162533223](https://oos.luoyunhao.com/blog-img/20210813162533.png)
  
  eg：Insert键可切换插入或替换模式

### 3、题目中有多个文件，需要将它们合并（文件合并）

- Linux下的文件合并

  命令格式：

  ```zsh
  cat [文件1 文件2 ...] > 输出文件名
  ```

  ![image-20210813163346330](https://oos.luoyunhao.com/blog-img/20210813163346.png)

  eg：完整性检测（计算文件md5），命令格式如下

  ```zsh
  md5sum 文件名
  ```

- Windows下的文件合并

  命令格式：

  ```
  copy /B [文件1+文件2+...] 输出文件名
  ```

  ![image-20210813164015852](https://oos.luoyunhao.com/blog-img/20210813164015.png)

  eg：完整性检测（计算文件md5），命令格式如下

  ```
  certutil -hashfile 文件名 md5
  ```

## 隐写术

### 文件内容隐写

文件内容隐写就是直接将flag以十六进制的形式写在文件中，通常在文件的开头或结尾部分。如果在文件中间部分，通常搜索关键字KEY或flag来查找隐藏内容

- Winhex/010Editor

  ![image-20210813165046370](https://oos.luoyunhao.com/blog-img/20210813165046.png)

- Notepad++

  查看文件中可能存在的URL等隐写内容

### 图片隐写

图片隐写常见的隐写方法：

①细微的颜色差别

②GIF图多帧隐藏

- 颜色通道隐藏
- 不同帧图信息隐藏
- 不同帧对比隐写

③Exif信息隐藏（通过照片获取到GPS等信息）

④图片修复

- 图片头修复
- 图片尾修复
- CRC校验修复
- 长、宽、高度修复

⑤最低有效位LSB隐写（二进制最后一位）

⑥图片加密

- Stegdetect
- outguess
- Jphide
- F5 

常用解题方法：

#### 1、Photoshop

可以用于图层分解、帧分解等

#### 2、Exif

windows可以直接右键属性查看exif信息

Linux命令如下：

```zsh
exiftool 文件名
```

#### 3、Stegsolve

Stegsolve可以用于通道分离、LSB分离、图像对比等

当两张jpg图片外观、大小、像素都基本相同时，可以考虑进行结合分析。即将两个文件的像素RGB值进行XOR、ADD、SUB等操作，查看能否得到有用的信息。

![image-20210813173808897](https://oos.luoyunhao.com/blog-img/20210813173809.png)

eg：有时候合并顺序不同，结果也不同，比如SUB操作

#### 4、LSB（最低有效位）

LSB替换隐写基本思想是用嵌入的秘密信息取代载体图像的最低比特位，原来的的7个高位平面与替代秘密信息的最低位平面组合成含隐藏信息的新图形

可以总结为通过修改像素中最低位的1bit来达到隐藏的效果

![image-20210813180126696](https://oos.luoyunhao.com/blog-img/20210813180126.png)

一般使用工具：stegsolve、zsteg、wbstego4、python脚本

- Stegsolve

  ![image-20210813180946402](https://oos.luoyunhao.com/blog-img/20210813180946.png)

- zsteg

  zsteg可以检测PNG和BMP中的隐写隐藏数据，在kali中需要先进行安装，安装命令如下：

  ```zsh
  gem install zsteg
  ```

  检测LSB隐写命令

  ```zsh
  zsteg 文件名
  ```

  ![image-20210813181420599](https://oos.luoyunhao.com/blog-img/20210813181420.png)

- wbstego4

  解密通过lsb加密的图片（特别针对.bmp文件解密，也可以对pdf、txt进行解密）

  eg：可以使用画图工具将文件转为.bmp文件

- python脚本

  某些情况下，常规解法无法解出时，就需要使用py脚本来进行解题

  如使用以下py脚本解题：

  ![image-20210813183743914](https://oos.luoyunhao.com/blog-img/20210813183744.png)

#### 5、TweakPNG检验CRC

TweakPNG是一个PNG图像浏览工具，它可以查看和修改一些PNG图像文件的元信息存储（针对PNG）。原理是根据图片的长宽高算出CRC校验值

使用场景：文件头正常却无法打开文件

首先排除CRC错误，利用TweakPNG检验CRC，如果错误，使用winhex进行修改，例：

![image-20210813201643743](https://oos.luoyunhao.com/blog-img/20210813201650.png)

当修改完CRC没有错误时，还没有想要的flag，就可能是图片的高度或者宽度发生了错误（将之前修改的CRC还原），通过最初的CRC计算出正确的高度或者宽度。

使用下面的python脚本可以计算高度或宽度，下图为PNG数据格式

![image-20210813203259031](https://oos.luoyunhao.com/blog-img/20210813203259.png)

eg：jpg数据格式（根据标记找到jpg修改宽高度的位置）

![image-20210813232517244](https://oos.luoyunhao.com/blog-img/20210813232517.png)

#### 6、Bftools解密

Bftools用于解密图片信息

使用方式：在windows的cmd下，对加密过的图片文件进行解密

命令格式：

```
Bftools.exe decode braincopter 需要解密的图片名 -output 输出文件名
Bftools.exe run 上一步输出文件名
```

![image-20210813210229411](https://oos.luoyunhao.com/blog-img/20210813210229.png)

#### 7、SilentEye解密

silenteye是一款针对将文字或者文件隐藏到图片的加解密工具（分析jpg和bmp）

使用方式：（新版中image为Media）

![image-20210813212913498](https://oos.luoyunhao.com/blog-img/20210813212913.png)

#### 8、Stegdetect工具探测加密方式

Stegdetect工具**主要用于分析JPEG文件**。使用Stegdetect可以检测到通过JSteg、JPHide、OutGuess、Invisible Secrets、F5、appendX和Camouflage等这些隐写工具隐藏的信息

kali环境需先进行安装，安装命令如下：

```zsh
apt-get install stegdetect
```

命令格式：（windows下加.exe）

```
stegdetect XXX.jpg
stegdetect -s 敏感度 XXX.jpgexi
```

> -s：修改检测算法的敏感度，算法敏感度的值越大，检测出的可疑文件包含敏感信息的可能性越大，默认值为1

![image-20210813214939880](https://oos.luoyunhao.com/blog-img/20210813214939.png)

知道了是哪种方式进行隐写的，就使用相应工具进行解密即可

- JPHide

  jphide是基于最低有效位LSB的JPEG格式图像隐写算法

  使用方式：

  ![image-20210813215922257](https://oos.luoyunhao.com/blog-img/20210813215922.png)

- Outguess

  outguess一般用于解密文件信息

  Linux需在网上下载到其[源码包](https://github.com/crorvick/outguess.git)，然后在关键目录下编译后再使用，编译命令如下：

  ```zsh
  ./configure && make && make install
  ```

  Linux下命令格式：(Windows工具有图形界面)

  ```zsh
  outguess -r 需解密的文件名 输出文件名 
  ```

  > -r：解密
  >
  > -k "key"：如有密钥，需加上该参数
  >
  > -d：加密

  ![image-20210813220632474](https://oos.luoyunhao.com/blog-img/20210813220632.png)

- F5

  F5一般用于解密文件信息（主要要调好java环境）

  Windows下命令格式：

  ```
  java Extract 需解密的文件名 -p 密码
  ```

  ![image-20210813220857029](https://oos.luoyunhao.com/blog-img/20210813220857.png)

#### 9、二维码处理

![image-20210813221250950](https://oos.luoyunhao.com/blog-img/20210813221251.png)

![image-20210813222634211](https://oos.luoyunhao.com/blog-img/20210813222634.png)

eg：二维码黑白颠倒处理：使用画图工具直接取反

### 音频隐写

#### 1、mp3stego

mp3stego主要用于mp3隐写

使用方式：

```zsh
encode -E hidden_text.txt -P pass svega.wav svega_stego.mp3   //加密
decode -X -P pass svega_stego.mp3         //解密
```

> -E：选择需要加密的文件
> -P：密码（加密需要将密码放入wav文件）
> -X：选择需要解密的文件

## 压缩文件处理

### 1、伪加密

如果压缩文件是加密的，或文件头正常但解压缩错误，首先尝试文件是否为伪加密。

**zip文件**是否加密是通过标识符来显示的，在每个文件的文件目录字段有一位专门标识了文件是否加密，将其设置为00表示该文件未加密后，如果成功解压则表示文件为伪加密，如果解压岀错说明文件为真加密。

操作方式：从文件头504B0102的50开始数，一直到第九和第十个字节就是设置加密的字段，将其修改为0000（ZIP）

![image-20210814105529335](https://oos.luoyunhao.com/blog-img/20210814105536.png)

   ![image-20210814110427556](https://oos.luoyunhao.com/blog-img/20210814110427.png)

**RAR文件**由于有头部校验,使用伪加密时打开文件会出现报错,使用winhex修改标志位后如报错消失且正常解压缩,说明是伪加密。

操作方式：使用winhex打开RAR文件,找到第24个字节,该字节尾数为4表示加密,0表示无加密,将尾数改为0即可破解伪加密

![image-20210814110903412](https://oos.luoyunhao.com/blog-img/20210814110903.png)

### 2、暴力破解

可以使用ARCHPR.exe工具来破解zip文件

![image-20210814111247868](https://oos.luoyunhao.com/blog-img/20210814111247.png)

   ![image-20210814112122917](https://oos.luoyunhao.com/blog-img/20210814112123.png)

### 3、明文攻击

明文攻击指知道加密的ZIP中部分文件的明文内容,利用这些内容推测岀密钥并解密ZI文件的攻击方法,相比于暴力破解,这种方法在破解密码较为复杂的压缩包时效率更高

攻击方式：

![image-20210814112620272](https://oos.luoyunhao.com/blog-img/20210814112620.png)

![image-20210814113200781](https://oos.luoyunhao.com/blog-img/20210814113200.png)

有时候不一定能破解出文件口令，但能获取到加密秘钥，有可能加密秘钥就是flag

![image-20210814113459290](https://oos.luoyunhao.com/blog-img/20210814113459.png)

使用该方法需要注意两个关键点:
1、有一个明文文件,压缩后CRC值与加密压缩包中的文件一致
2、明文文件的压缩算法需要与加密压缩文件的压缩算法一致

![image-20210814113700800](https://oos.luoyunhao.com/blog-img/20210814113700.png)

可以使用好压来对明文文件进行压缩

![image-20210814141205622](https://oos.luoyunhao.com/blog-img/20210814141212.png)

### 4、RAR文件修复

有时候给出的RAR文件的头部各个字块故意给错，导致无法识别而少文件

![image-20210814142811219](https://oos.luoyunhao.com/blog-img/20210814142811.png)

下图为RAR文件快类型位置说明：

![image-20210814142826136](https://oos.luoyunhao.com/blog-img/20210814142826.png)

## 流量取证

这种题型通常比赛中会提供一个包含流量数据的PCAP文件，有时候也会需要选手们先进行修复或重构传输文件后，再进行分析

比赛中的流量分析可以概括为以下三个方向：

- 流量包修复
- 协议分析
- 数据提取

流量取证一般使用Wireshark工具对PCAP文件进行分析

### Wireshark过滤

Wireshark工具常用的过滤命令如下：

![image-20210814150833865](https://oos.luoyunhao.com/blog-img/20210814150833.png)

![image-20210814151105101](https://oos.luoyunhao.com/blog-img/20210814151105.png)

![image-20210814151421439](https://oos.luoyunhao.com/blog-img/20210814151421.png)

### Wireshark协议分级

当流量包的数据特别大和复杂时，可以使用协议分级统计功能，能够更好地分析数据（主要观察占百分比大的包）

![image-20210814152456783](https://oos.luoyunhao.com/blog-img/20210814152456.png)

### Wireshark流汇聚

在关注的http数据包或tcp数据包中选择流汇聚，可以将HTTP流或TCP流汇聚或还原成数据，在弹出的框中可以看到数据内容

![image-20210814152747229](https://oos.luoyunhao.com/blog-img/20210814152747.png)

常见的HTTP流关键内容：

- HTML中直接包含重要信息
- 上传或下载文件内容，通常包含文件名、hash值等关键信息，常用POST请求上传
- 一句话木马，特征有：POST请求、内容包含eval、内容使用base64加密

### Wireshark数据提取

- 自动提取http传输的文件内容（与文件分离相似）

  ![image-20210814154625611](https://oos.luoyunhao.com/blog-img/20210814154625.png)

- 手动提取文件内容

  ![image-20210814155634574](https://oos.luoyunhao.com/blog-img/20210814155634.png)

### 无线流量包

Wireshark协议分析发现只有wireless LAN协议，则很有可能是WPA或者WEP加密的无线数据包。这时我们可以用到kali中相关的无线工具

1、kali使用aircrack-ng检查cap包

命令格式：

```zsh
aircrack-ng xxx.cap
```

![image-20210814162204392](https://oos.luoyunhao.com/blog-img/20210814162204.png)

2、aircrack-ng跑字典进行握手包破解

命令格式：

```zsh
aircrack-ng xxx.cap -w 字典txt文件
```

![image-20210814162354394](https://oos.luoyunhao.com/blog-img/20210814162354.png)

### USB流量包

USB流量一般考察项涉及键盘击键、鼠标移动与点击、存储设备的明文传输通信、USB无线网络传输内容等

#### USB流量分析

USB协议的数据部分在Leftover Capture Data域之中

![image-20210814164729365](https://oos.luoyunhao.com/blog-img/20210814164729.png)

USB鼠标流量规则如下图所示：

![image-20210814172019772](https://oos.luoyunhao.com/blog-img/20210814172019.png)

![image-20210814194746717](https://oos.luoyunhao.com/blog-img/20210814194746.png)

USB键盘流量规则如下图所示：

![image-20210814172126475](https://oos.luoyunhao.com/blog-img/20210814172126.png)

#### USB键盘流量抓取

使用wireshark提供的命令行工具tshark，可以将Leftover Capture Data数据单独复制出来

命令格式：（输出的文件中如果没有冒号，可以配合python脚本将冒号加上去）

```zsh
tshark -r xxx.pcap -T fields -e usb.capdata > xxx.txt
```

![image-20210814193206085](https://oos.luoyunhao.com/blog-img/20210814193206.png)

之后使用python脚本对键盘数据进行相应转换即可

#### USB鼠标流量抓取分析

首先将Leftover Capture Data数据提取出，命令如下：

```zsh
tshark -r xxx.pcap -T fields -e usb.capdata > xxx.txt
```

kali使用gnuplot工具把坐标画出来

```zsh
gnuplot
plot "xxx.txt"
```

![image-20210816004832275](https://oos.luoyunhao.com/blog-img/20210816004832.png)

### HTTPS流量

HTTPS流量是经过TLS协议加密过的，需要导入key才能看到原始的HTTP流量（key需要在题目中寻找）

![image-20210814201635340](https://oos.luoyunhao.com/blog-img/20210814201635.png)

导入key后会出现原始的HTTP流量

![image-20210814201923654](https://oos.luoyunhao.com/blog-img/20210814201923.png)

## Crypto密码学

### 密码学概述

- 目的：为了保证数据传输的可靠性
- 核心：密码学（用于数据动态传输和静态存储）

### 编码

![image-20210815125754963](https://oos.luoyunhao.com/blog-img/20210815125755.png)

编码相当于有一张映射表

#### 常见编码

- ASCII编码

  ![image-20210815160401097](https://oos.luoyunhao.com/blog-img/20210815160401.png)

  ![image-20210815160346066](https://oos.luoyunhao.com/blog-img/20210815160346.png)

- Base64编码

  ![image-20210815133101699](https://oos.luoyunhao.com/blog-img/20210815133101.png)

- URL编码

  ![image-20210815133206122](https://oos.luoyunhao.com/blog-img/20210815133206.png)

- Unicode编码

  ![image-20210815134517034](https://oos.luoyunhao.com/blog-img/20210815134517.png)

- JS混淆

  ![image-20210815134859656](https://oos.luoyunhao.com/blog-img/20210815134955.png)

- JSFuck

  ![image-20210815163129614](https://oos.luoyunhao.com/blog-img/20210815163129.png)

  ![image-20210816221343261](https://oos.luoyunhao.com/blog-img/20210816221343.png)

- Jother

  ![image-20210815163309685](https://oos.luoyunhao.com/blog-img/20210815163310.png)

  先将`{`和`}`分别替换成`[`和`]`后再解密即可

- aaencode

  ![image-20210815163533983](https://oos.luoyunhao.com/blog-img/20210815163534.png)
  
  直接在浏览器的控制台上解密即可

### 加密算法

![image-20210815125823957](https://oos.luoyunhao.com/blog-img/20210815125824.png)

#### RSA基本原理

公钥与秘钥的产生原理

![image-20210817140928281](https://oos.luoyunhao.com/blog-img/20210817140935.png)

加密消息的过程

![image-20210817141125622](https://oos.luoyunhao.com/blog-img/20210817141125.png)

解密消息的过程

![image-20210817141458668](https://oos.luoyunhao.com/blog-img/20210817141458.png)

#### 常见的几种算法

- 换位加密：栅栏密码、曲路密码、列位移密码

  栅栏密码

  ![image-20210815164653167](https://oos.luoyunhao.com/blog-img/20210815164653.png)

  曲路密码

  ![image-20210815164738014](https://oos.luoyunhao.com/blog-img/20210815164738.png)

  位列移密码

  ![image-20210815165950971](https://oos.luoyunhao.com/blog-img/20210815165951.png)

- 替换加密：凯撒密码、摩斯密码、ROT5/13/18/47、维吉尼亚密码、培根密码、键盘密码

  凯撒密码

  ![image-20210815170103713](https://oos.luoyunhao.com/blog-img/20210815170104.png)

  摩斯密码

  ![image-20210815170541292](https://oos.luoyunhao.com/blog-img/20210815170541.png)

  ROT5/13/18/47

  ![image-20210815170749596](https://oos.luoyunhao.com/blog-img/20210815170749.png)

  维吉尼亚密码

  ![image-20210815171237926](https://oos.luoyunhao.com/blog-img/20210815171238.png)

  培根密码

  ![image-20210815171355774](https://oos.luoyunhao.com/blog-img/20210815171355.png)

  这里注意培根密码有两种对应关系，分别为大写和小写的对应

  键盘密码

  ![image-20210815171447113](https://oos.luoyunhao.com/blog-img/20210815171447.png)

- 其他密码：MD5、SHA（摘要）

### 摘要算法

![image-20210815130831799](https://oos.luoyunhao.com/blog-img/20210815130831.png)

摘要算法有两大特点：雪崩效应和不可逆

MD5和SHA主要应用领域：

![image-20210815160030715](https://oos.luoyunhao.com/blog-img/20210815160030.png)

##  Web基础

### HTTP请求报文

http请求由三部分组成，分别是：请求行、消息报头、请求正文

![image-20210816213851395](https://oos.luoyunhao.com/blog-img/20210816213851.png)

#### 请求行

请求行包括http请求的种类/方法、请求资源、http协议版本三个部分，以空格符（%20）分割，以回车换行符结尾（%0d%0a）

![image-20210816214214938](https://oos.luoyunhao.com/blog-img/20210816214215.png)

#### 请求消息报头

请求消息报头说明了客户端的基本信息，以及如何与客户端进行交互，消息报头由多行组成，每行以key:value的形式体现，每行末尾包括一个回车换行符（0%0a0d），消息报头末尾以两个回车换行符（%0a%0d）作为结束

![](https://oos.luoyunhao.com/blog-img/20210816174530.png)

### HTTP响应报文

http响应由三部分组成，分别是：状态行、消息报头、响应正文

![image-20210816214628779](https://oos.luoyunhao.com/blog-img/20210816214628.png)

#### 状态行

状态行包括http协议版本、状态码、状态描述三个部分，以空格符（%20）分隔，以回车换行符结尾（%0d%0a）

![image-20210816214959921](https://oos.luoyunhao.com/blog-img/20210816215000.png)

#### 响应消息报头

请求响应报头说明了服务端的基本信息、客户端如何处理返回的消息等，消息报头由多行组成,每行以key: value的形式体现，每行未尾包括一个回车换行符(%0a%0d)，消息报头末尾以两个回车换行符（%0a%0d）作为结束

![image-20210816215343383](https://oos.luoyunhao.com/blog-img/20210816215343.png)

## SQL注入

### SQL注入介绍

![image-20210816224622524](https://oos.luoyunhao.com/blog-img/20210816224622.png)

### SQL注入万能密码

从万能密码中可以看出SQL注入的基本原理及如何利用

![image-20210816230242839](https://oos.luoyunhao.com/blog-img/20210816230242.png)

### 自动化注入

有时候手工注入非常累且费时间，因此可以使用自动化注入工具，如椰树等工具

![image-20210816232236437](https://oos.luoyunhao.com/blog-img/20210816232236.png)

## 文件上传漏洞

### 介绍

如果WEB应用在文件上传过程中没有对文件的安全性进行有效的校验，攻击者可以通过上传Webshell等恶意文件对服务器进行攻击，这种情况下认为系统存在文件上传漏洞

![image-20210816232938933](https://oos.luoyunhao.com/blog-img/20210816232939.png)

### 上传绕过方式

#### 前端JS校验

先将php后缀改为符合前端JS校验的后缀，再使用Burp截获上传操作，将后缀名再改为php放行即可

#### 后端校验

使用burp拦截上传流，将请求消息报头中Content-Type的值修改为符合后端校验的文件格式即可

![image-20210817003001941](https://oos.luoyunhao.com/blog-img/20210817003002.png)

### 利用

最常见利用文件上传漏洞的方法就是上传网站木马（webshell）文件，webshell又称网页木马文件，根据开发语言的不同又分为ASP木马、PHP木马、JSP木马等，该类木马利用了脚本语言中的系统命令执行、文件读写等函数的功能，一旦上传到服务器被脚本引擎解析，攻击者就可以实现对服务器的控制

![image-20210816233450666](https://oos.luoyunhao.com/blog-img/20210816233450.png)

#### 小马介绍

例：使用一句话木马控制服务器

首先准备好一个小马，这里以PHP为例，代码可以为如下：

```php
<?php eval($_POST['gok']);?>
```

> Eval函数可以将传入的内容作为php脚本执行
>
> `$_POST['v']`传入想要执行的代码
>
> `gok`是传递数据的参数，只有使用正确的参数名提交代码才能够被执行，因此也称为一句话木马的密码

利用上传漏洞将该PHP文件上传至服务器后，使用菜刀进行连接

![image-20210816234633487](https://oos.luoyunhao.com/blog-img/20210816234633.png)

连接成功之后可以对其文件、数据库进行操作，以及使用终端进行命令的输入

![image-20210816234828294](https://oos.luoyunhao.com/blog-img/20210816234828.png)

大马介绍

大马可以直接通过浏览器对服务器进行相关操作和管理

![image-20210816235534403](https://oos.luoyunhao.com/blog-img/20210816235534.png)

