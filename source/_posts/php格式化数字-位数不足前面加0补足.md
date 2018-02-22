---
title: php格式化数字-位数不足前面加0补足
categories: 编程
tags:
  - php
  - 格式化输出
translate_title: php-formatting-numbers-plus-0-in-front-of-insufficient-digits
date: 2016-07-12 14:37:11
---

## 第一种方法：
```php
$var=sprintf("%04d", 2);//生成4位数，不足前面补0
echo $var;//结果为0002
```
### 语法

```php
sprintf(format,arg1,arg2,arg++)
```
|参数	|描述|
| --------   |--------   |
|format|	必需。转换格式。|
|arg1	|必需。规定插到 format 字符串中第一个 % 符号处的参数。|
|arg2	|可选。规定插到 format 字符串中第二个 % 符号处的参数。|
|arg++	|可选。规定插到 format 字符串中第三、四等等 % 符号处的参数。|

### 说明

参数 format 是转换的格式，以百分比符号 (“%”) 开始到转换字符结束。下面的可能的 format 值：
|参数|描述|
| --------   |--------   |
|%%  |返回百分比符号|
|%b  |二进制数|
|%c  |依照 ASCII 值的字符|
|%d  |带符号十进制数|
|%e  |可续计数法（比如 1.5e+3）|
|%u  |无符号十进制数|
|%f  |浮点数(local settings aware)|
|%F  |浮点数(not local settings aware)|
|%o  |八进制数|
|%s  |字符串|
|%x  |十六进制数（小写字母）|
|%X  |十六进制数（大写字母）|
arg1, arg2, ++ 等参数将插入到主字符串中的百分号 (%) 符号处。该函数是逐步执行的。在第一个 % 符号中，插入 arg1，在第二个 % 符号处，插入 arg2，依此类推。

### 第二种方法：
```bash
$number = 1234.56;
// english notation (default)
$english_format_number = number_format($number);
// 1,235
// French notation
$nombre_format_francais = number_format($number, 2, ',', ' ');
// 1 234,56
$number = 1234.5678;
// english notation without thousands seperator
$english_format_number = number_format($number, 2, '.', '');
// 1234.57
```

两种方法都不错，适合于使用在生成序列号或者流水号的时候使用




