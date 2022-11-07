---
layout: post
title: "基于Polyworks的宏脚本编程语言（MSCL）基础"
categories: Polyworks
tags: 编程语言
excerpt: Polyworks作为通用测量软件平台，提供了内置的宏脚本编程语言，可以使用基于COM组件的技术调用软件底层指令与功能，实现测量计划自动化编制，和全流程的自动化测量，提供软件的使用效率。
---
* content
{:toc}

&emsp;&emsp;Polyworks作为通用测量软件平台，提供了内置的宏脚本编程语言，可以使用基于COM组件的技术调用软件底层指令与功能，实现测量计划自动化编制，和全流程的自动化测量，提供软件的使用效率。

## **一.语法基础**
### **变量**
&emsp;&emsp;变量是用于存储和查询信息的容器，在Polyworks中存在三种基本变量类型，包括整型，浮点型和字符串型等

- 整型：&ensp;&ensp;&ensp;&ensp;整数（无小数点）
- 浮点型：&ensp;&ensp;允许小数点的数字
- 字符串型：字母和数字字符的序列

&emsp;&emsp;在定义变量时不需要显式的指明变量类型，当为其赋值时，宏脚本内核会自动为其适配变量类型，定义变量使用`DECLEAR`指令，变量初始值可以在声明变量是指定，也可以使用`SET`指令指定：
``` as
DECLEAR varName
DECLEAR featureCount 5
SET featureCount 10
```
> 变量名可以由任意非保留变量构成，保留变量包含：`_DOLLAR`, `_INSTALL_PATH`, `_PI`, `_NEWLINE`等。在实际编程中，尽可能使用描述性的命名, 别心疼空间, 毕竟相比之下让代码易于新读者理解更重要. 不要用只有项目开发者能理解的缩写, 也不要通过砍掉几个字母来缩写单词。命名方法推荐使用**骆驼式命名法**：`featureCount`，和**下划线法**`feature_count`。

&emsp;&emsp;**变量值的获取**方法在变量名前添加`$`，如果变量存在于字符串中，则需要使用`{ }`将变量名包裹起来:
``` as
DECLARE variable1 "Message 1"
MACRO ECHO ( $variable1 )
MACRO ECHO ( "${variable1} followed by extra words" )
```
```
OUTPUT: Message 1
OUTPUT: Message 1 followed by extra words
```

**变量修饰符**
&emsp;&emsp;在某些情况下，比如字符串比较，需要将用户输入的字符串转换为小写字母或者大写字符，可以使用`:l`将修饰的字符串转化为小写，使用`:u`会将修饰的字符串转化大写：
``` as
DECLARE variable1 "Message 1"
MACRO ECHO ( "${variable1} Uppercase = ${variable1:u}" )
```
```
OUTPUT: Message 1 Uppercase = MESSAGE 1
```

### **保留变量**
&emsp;&emsp;保留变量用于存储与Polyworks环境相关的信息，可以用于查询相关的软件环境信息，主要环境变量如下：

| 保留变量           | 变量意义             |
|-------------------|--------------------|
| _DOLLAR           | $ |
| _QUOTES           | " |
| _PI               | 3.14159 |
| _NEWLINE          |  换行符 |
| _INSTALL_PATH     | Polyworks安装目录 |
| _PWK_FILES_PATH   | 当前项目_Files目录 |
| _TEMP_PATH        | 软件临时目录 |
| _USERCONFIG_PATH  | 用户配置目录 |


### **表达式**
#### **数学表达式**
- 数学表达式使用`EXPR`和`EXPR_I`来获得计算结果，两种不同的地方在于`EXPR_I`去除结果的小数部分；
- 基本数学操作符：`+`, `-`, `*`,`/`, `%` (取模)；
- 基本数学函数：`ABS` (取绝对值), `SQRT` (平方根), `SIN`, `COS`, `TAN`等等；

``` as
DECLARE a 1
DECLARE b 2
DECLARE c
DECLARE d
DECLARE e

SET c EXPR ( $a + $b )
SET d EXPR ( SQRT( $c ) )
SET e EXPR_I ( SQRT( $c ) )

MACRO PAUSE ( "Math", "$a + $b = $c", "Square Root of $c = $d", "Square Root of $c (no decimals) = $e" )
```
#### **逻辑表达式**
&emsp;&emsp;在宏脚本语言中，需要使用逻辑表达式判断指令执行的结果和当前变量的状态。

**比较操作符**

- `<` &ensp;&ensp;小于
- `<=` &ensp;小于等于
- `>` &ensp;&ensp;大于
- `>=` &ensp;大于等于
- `==` &ensp;等于
- `!=` &ensp;不等于

**逻辑操作符**

- `AND` &ensp;**逻辑与**，两个表达式同时为真时，结果为真，否则为假；
- `OR`  &ensp;&ensp;**逻辑或**，两个表达式只需要一个为真，结果即为真，同时为假时，结果才为假；
- `NOT` &ensp;**逻辑非**，表达式为真时，返回结果为假，表达式为假时，返回结果为真；

``` as
IF $c == 0
    MACRO PAUSE ( "Zero", "$c equals zero." )
ELSEIF $c < 0
    MACRO PAUSE ( "Smaller than zero", "$c is negative (<0)." )
ELSE
    MACRO PAUSE ( "Bigger than zero", "$c is positive (>0)." )
ENDIF

IF $a > 0 AND $b > 0
    MACRO PAUSE ( "Bigger than zero", "$a AND $b are both bigger than zero." )
ENDIF

IF $a > 0 OR $b > 0
    MACRO PAUSE ( "Bigger than zero", "$a OR $b is bigger than zero." )
ENDIF
```

### **语句**
#### **条件语句**
&emsp;&emsp;条件语句用于控制程序的执行顺序，条件语句的指令如下：
- `IF`: &emsp;&emsp;&emsp;条件语句的开始，用于判断逻辑表达式
- `ELSEIF`: &emsp;增加分支判断
- `ELSE`: &emsp;&emsp;当前面所有的判断语句均为假时，执行该分支
- `ENDIF`: &emsp;&ensp;条件语句退出

[![xvJoIH.jpg](https://s1.ax1x.com/2022/11/07/xvJoIH.jpg)](https://imgse.com/i/xvJoIH)

语法结构如下：
``` as 
IF <Condition_1>
    <Statement_1>
ELSEIF <Condition_1>
    <Statement_2>
ELSE
    <Statement_3>
ENDIF
```
#### **循环语句**
&emsp;&emsp;循环语句用于重复的执行特定的语句，并在每一次循环迭代后，判断当前的循环条件。在宏脚本编程语言中只存在一种循环结构：`WHILE`循环，相关指令如下：
- `WHILE`: 循环结构开始
- `ENDWHILE`: 循环结构结束
- `BREAK`: 终止当前循环结构
- `CONTINUE`: 跳出本次循环
- `++`: 当前变量（循环变量）累加`1` (与`SET i EXPR ( $i + 1 )`相同 )
- `--`: 当前变量（循环变量）累减`1` (与`SET i EXPR ( $i - 1 )`相同 )


语法结构如下：
``` as
DECLARE PolyWorksIter 1
WHILE $PolyWorksIter <= $PolyWorksSize
    MACRO ECHO( "PolyWorks Module $PolyWorksIter = $PolyWorks[$PolyWorksIter]" )    
    ++ PolyWorksIter
ENDWHILE

```

### **数组**
&emsp;&emsp;**数组**是由一组元素组成的序列，其中的每个元素都分配一个数字（索引）--它的位置，其中第一次元素的索引为1，第二个元素的索引为2，以此类推。

> MSCL中的仅支持一维数组，不支持二维或三维等高维数组；
> 
> 数组中的元素的类型可以不同，比如`{5.0, 2, "user"}`就是一个合法的数组

MSCL中提供了操作数组的指令：

|宏指令|描述|
|---|---|
|MACRO ARRAY ARE_EQUAL|将一个数组与另一个数组进行对比|
|MACRO ARRAY ELEMENTS AT_INDEXES|返回包含位于指定索引处的元素的数组|
|MACRO ARRAY ELEMENTS COUNT_OF|返回在数组中找到的元素的出现次数|
|MACRO ARRAY ELEMENTS INDEXES_OF|返回数组的值等于给定元素的所有对应索引|
|MACRO ARRAY ELEMENTS INDEX_OF|返回对应的数组值等于给定元素的最低索引|
|MACRO ARRAY ELEMENTS INSERT AT_INDEX|将包含元素的给定数组插入到数组的给定索引处|
|MACRO ARRAY ELEMENTS REMOVE AT_INDEXES|从数组的指定索引处删除元素|
|MACRO ARRAY ELEMENTS SORT DOUBLE|使用指定的排序排列双精度数组|
|MACRO ARRAY ELEMENTS SORT INTEGER|使用指定的排序排列整形数组|
|MACRO ARRAY ELEMENTS SORT STRING|使用指定的排序排列字符串数组|

### **用户输入**