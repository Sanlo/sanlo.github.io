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

**宏脚本使用场景汇总**

|场景|使用性|备注|
|-|-|
|工具栏|&emsp;✅&emsp;|包括Workspace Manager，Inspector和Modeler等|
|序列编辑器|&emsp;✅&emsp;|位于Inspector中|
|宏脚本编辑器|&emsp;✅&emsp;|主要用于编辑调试宏脚本|
|对象测量|&emsp;✅&emsp;|位于Inspector中|
|事件触发|&emsp;✅&emsp;|位于Inspector中|
|响应特殊事件|&emsp;⛔&emsp;|主要由工作区的打开关闭，和扫描会话结束来触发，需要使用定义特定文件名|

## **一.语法基础**
### **变量**
&emsp;&emsp;变量是用于存储和查询信息的容器，在Polyworks中存在三种基本变量类型，包括整型，双精度型和字符串型等

- 整型：&emsp;&ensp;&emsp;整数（无小数点）
- 双精度型：&ensp;允许小数点的数字
- 字符串型：&ensp;字母和数字字符的序列

&emsp;&emsp;在定义变量时不需要显式的指明变量类型，当为其赋值时，宏脚本内核会自动为其适配变量类型，定义变量使用`DECLEAR`指令，变量初始值可以在声明变量是指定，也可以使用`SET`指令指定：
``` as
DECLEAR varName
DECLEAR featureCount 5
SET featureCount 10
```
> 变量名可以由任意非保留变量构成，保留变量包含：`_DOLLAR`, `_INSTALL_PATH`, `_PI`, `_NEWLINE`等。在实际编程中，尽可能使用描述性的命名, 别心疼空间, 毕竟相比之下让代码易于新读者理解更重要. 不要用只有项目开发者能理解的缩写, 也不要通过砍掉几个字母来缩写单词。命名方法推荐使用**骆驼式命名法**：`featureCount`，和**下划线法**`feature_count`。

&emsp;&emsp;**获取变量值**方法在变量名前添加`$`，如果变量存在于字符串中，则需要使用`{ }`将变量名包裹起来:
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

> ***（需Polyworks 2020 IR1及以上版本）***

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

语法结构如下：
``` as 
IF <Condition_1>
    <Statement_1>
ELSEIF <Condition_2>
    <Statement_2>
ELSE
    <Statement_3>
ENDIF
```

![zSAkvj.png](https://s1.ax1x.com/2022/11/09/zSAkvj.png)

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
WHILE <Condition>
    <ProcessCode>
ENDWHILE
```

![zSEbXd.png](https://s1.ax1x.com/2022/11/09/zSEbXd.png)

### **数组**
&emsp;&emsp;**数组**是由一组元素组成的序列，其中的每个元素都分配一个数字（索引）--它的位置，其中第一次元素的索引为1，第二个元素的索引为2，以此类推。

> MSCL中的仅支持一维数组，不支持二维或三维等高维数组；
> 
> 数组中的元素的类型可以不同，比如`{5.0, 2, "user"}`就是一个合法的数组

MSCL中提供了完善的操作数组的指令 ：

> ***（需Polyworks 2021 IR1及以上版本）***
 
|宏指令|描述|
|---|---|
|`MACRO ARRAY ARE_EQUAL`|将一个数组与另一个数组进行对比|
|`MACRO ARRAY ELEMENTS AT_INDEXES`|返回包含位于指定索引处的元素的数组|
|`MACRO ARRAY ELEMENTS COUNT_OF`|返回在数组中找到的元素的出现次数|
|`MACRO ARRAY ELEMENTS INDEXES_OF`|返回数组的值等于给定元素的所有对应索引|
|`MACRO ARRAY ELEMENTS INDEX_OF`|返回对应的数组值等于给定元素的最低索引|
|`MACRO ARRAY ELEMENTS INSERT AT_INDEX`|将包含元素的给定数组插入到数组的给定索引处|
|`MACRO ARRAY ELEMENTS REMOVE AT_INDEXES`|从数组的指定索引处删除元素|
|`MACRO ARRAY ELEMENTS SORT DOUBLE`|使用指定的排序排列双精度数组|
|`MACRO ARRAY ELEMENTS SORT INTEGER`|使用指定的排序排列整型数组|
|`MACRO ARRAY ELEMENTS SORT STRING`|使用指定的排序排列字符串数组|

### **用户输入**

&emsp;&emsp;用户界面是用户与运行中的宏脚本进行信息交互的途径，用户可以通过键盘输入相应的参数，或者选择用户处理的文件或者文件夹。

#### **基本用户界面**
&emsp;&emsp;在基本用户界面中用户通常可以一次输入一个参数，通常包括整型，双精度型，字符串型，以及密码类型参数，也支持文件路径和文件夹路径的选择，借助`MACRO INPUT MULTIPLE_PARAMETERS`指令也可以将其合并到一个用户界面中，一次性输入，常用的宏指令如下：

|宏指令|描述|
|---|---|
|`MACRO INPUT DOUBLE`|输入双精度型参数对话框|
|`MACRO INPUT INTEGER`|输入整型参数对话框|
|`MACRO INPUT STRING`|输入字符串参数对话框|
|`MACRO INPUT PASSWORD`|输入密码型参数对话框，用户键盘输入遮盖为`*` |
|`MACRO INPUT QUESTION`|提示用户选择【是】或【否】的对话框|
|`MACRO INPUT FILE_PATH`|提示用户选择文件路径|
|`MACRO INPUT FOLDER_PATH`|提示用户选择文件夹路径|
|`MACRO INPUT PWK_OBJECTS`|提示用户选择工作区对象|
|`MACRO INPUT MULTIPLE_PARAMETERS`|使用多个参数创建的对话框|
|`MACRO PAUSE`|输出信息对话框|

#### **高级用户界面**

> ***（需Polyworks 2020 IR2及以上版本）***

&emsp;&emsp;使用`MACRO INPUT DIALOG_BOX`命令，可以在创建对话框时提供更多可能性，不仅可以实现常规的参数输入，还可以添加**多选列表框**，**单选框**和**复选框**等高级的用户界面组件，这在用户需要多个参数值时非常有用。

效果图如下：
![zSunOg.png](https://s1.ax1x.com/2022/11/09/zSunOg.png)

<br/>

常用的宏指令如下：

|宏指令|描述|
|---|---|
|`MACRO INPUT DIALOG_BOX CHECKBOX`|将复选框添加到对话框|
|`MACRO INPUT DIALOG_BOX DEFINE`|定义一个对话框|
|`MACRO INPUT DIALOG_BOX DROP_DOWN_LIST`|将列表框添加到对话框。请注意，可创建单选和多选列表框|
|`MACRO INPUT DIALOG_BOX EDITBOX DOUBLE`|将有效值为双精度值的文本框添加到对话框|
|`MACRO INPUT DIALOG_BOX EDITBOX INTEGER`|将有效值为整数值的文本框添加到对话框|
|`MACRO INPUT DIALOG_BOX EDITBOX PASSWORD`|将密码文本框添加到对话框|
|`MACRO INPUT DIALOG_BOX EDITBOX STRING`|将有效值为字符串值的文本框添加到对话框|
|`MACRO INPUT DIALOG_BOX FILE_PATH`|将文件路径文本框和浏览按钮添加到对话框|
|`MACRO INPUT DIALOG_BOX FOLDER_PATH`|将文件夹路径文本框和浏览按钮添加到对话框|
|`MACRO INPUT DIALOG_BOX LABEL`|将标签添加到对话框|
|`MACRO INPUT DIALOG_BOX RADIO_GROUP`|将一组选项按钮添加到对话框|
|`MACRO INPUT DIALOG_BOX SECTION`|将区段添加到对话框。请注意，区段可展开或折叠|
|`MACRO INPUT DIALOG_BOX SHOW`|向用户显示对话框|

&emsp;&emsp;要使用这些命令，首先使用`MACRO INPUT DIALOG_BOX DEFINE`命令定义对话框并为其分配一个标题。然后，使用相应的`MACRO INPUT DIALOG_BOX`命令将所需参数添加到对话框。请注意，它们的显示顺序与输入顺序相同。

### **文件操作**
&emsp;&emsp;常见的文件类型可以分为文本型文件和二进制型文件，二进制型文件无法使用记事本等文本编辑器直接编辑，需要使用专用的文件解析器解析其中的二进制对象，而文本型文件可以使用MSCL对其进行文件的创建，读取与信息写入等操作，也可以获得文件的行数和字段数等信息。
#### **文件创建**
&emsp;&emsp;文本型文件在使用MSCL进行创建时可以指定文件编码类型，包含`Unicode`和`Ascii`两种类型，默认为`Ascii`，也可以指定文件覆盖模式，包括覆写原有文件和追加原有文件两种方式，文件创建指令如下：
``` as
DATA_FILE CREATE ( <File name>, <Unicode or ASCII file>, <Overwrite the file> )
```

#### **文件写入**
文件写入方法包括追究字符串和追加单行以及追加多行，具体包含如下指令：

|宏指令|描述|
|---|---|
|`DATA_FILE APPEND`|追加字符串至文件|
|`DATA_FILE APPEND LINE`|追加字符串数组至文件，数组元素使用空格键分割，形成一行文本|
|`DATA_FILE APPEND LINES`|追加字符串数组至文件，每个数组元素占据一行|

#### **文件内容读取**
&emsp;&emsp;文本型文件的读取包含多种形式，可以对一行，一行中的某个字段或者所有字段，以及每一列的信息进行读取。**实际编程中，为了提高脚本执行效率，运行一条指令，应该读取尽可能多的信息**。具体包含如下指令：

|宏指令|描述|
|---|---|
|`DATA_FILE READ LINE`|从文件中获得指定一行的信息|
|`DATA_FILE READ COLUMNS`|获得每一列元素的字符串数组，可以跳过不需要的列，**推荐使用**|
|`DATA_FILE READ LINE_FIELD`|获得文件指定行中指定列的字段信息|
|`DATA_FILE READ LINE_FIELDS`|获得文件指定行中所有列的信息，返回一个字段数组|

#### **文件属性读取**
&emsp;&emsp;文本型文件的行数和每一行的字段数可以通过如下指令获得：

|宏指令|描述|
|---|---|
|`DATA_FILE PROPERTIES NB_LINES GET`|获得文件的行数|
|`DATA_FILE PROPERTIES NB_FIELDS_IN_LINE GET`|基于分隔符获得文件指定行的字段数（列数），|

<br/>
> 在MSCL中，提供了用于监听文件读写状态的的指令`DATA_FILE WAIT_UNTIL_MODIFIED`，该指令会暂停脚本的执行，直到指定的文件被修改，或者等待超时。如果文件不存在，脚本将会等待文件创建完成后，再执行后续指令。

## **二.对象操作（Inspector）**
&emsp;&emsp;在MSCL中的对象操作主要包括对象选择，对象类型判断，对象属性查询与设置，对象尺寸控制查询与设置等。主要的操作指令可以分为两类，即基于`OBJECT`的对象操作，和基于具体对象类型的对象操作，前者指令格式为`TREEVIEW OBJECT <OPERATION COMMAND>`，后者格式为`TREEVIEW <OBJECT NAME> <OPERATION COMMAND>`，为了提高脚本的通用性，首选基于`OBJECT`版的对象操作指令。
### **选择对象**

|宏指令|描述|
|---|---|
|`TREEVIEW OBJECT SELECT NONE`|取消目录树中所有对象的选择|
|`TREEVIEW OBJECT SELECT`|选择或者取消选择指定对象|
|`TREEVIEW OBJECT SELECT GET`|获得对象的选择状态|

### **获取对象类型**
&emsp;&emsp;使用`TREEVIEW OBJECT PROPERTIES TYPES GET`可以作为通用的对象类型查询指令，可以获得对象的类型（Reference，Feature，Comparison Point等），和主类型（Plane，Circle，Slot等）等信息。
### **对象尺寸控制**
&emsp;&emsp;获取和设置目录树中对象的尺寸和控制建议使用`TREEVIEW OBJECT PROPERTIES DIMENSION`系列的宏指令，不建议使用`MEASURE CONTROL MEASURED`和`MEASURE CONTROL NOMINAL`相关的宏指令，因为这两个宏指令只能设置和获取当前对象在**几何控制**里已经勾选过的尺寸控制。

|宏指令|描述|
|---|---|
|`TREEVIEW OBJECT PROPERTIES DIMENSION NOMINAL`|设定对象指定尺寸控制的名义值|
|`TREEVIEW OBJECT PROPERTIES DIMENSION NOMINAL GET`|获取对象指定尺寸控制的名义值|
|`TREEVIEW OBJECT PROPERTIES DIMENSION MEASURED`|设定对象指定尺寸控制的测量值|
|`TREEVIEW OBJECT PROPERTIES DIMENSION MEASURED GET`|获取对象指定尺寸控制的测量值|


## **三.脚本与进程调用**
通常情况下，编写宏脚本时，会将不同的功能放在不同的脚本文件中，脚本之间通过互相调用来实现更加复杂的功能，同时也可以调用Polyworks外部的进程完成一些当前MSCL指令无法实现的数据计算或者通信等。涉及到的宏指令如下：

|宏指令|描述|
|---|---|
|`MACRO EXEC`|执行指定的宏脚本|
|`MACRO OUTPUT_ARGUMENT`|设置传递到输出自变量中的值 |
|`MACRO EXEC REMOTE_COMMAND`|执行远程指令，当在工作区中使用时，需要指定具体的模块ID，在模块中执行的仅仅是工作区中的远程指令|
|`MACRO EXEC REMOTE_SCRIPT`|执行远程脚本，当在工作区中使用时，需要指定具体的模块ID，在模块中执行的仅仅是工作区中的远程脚本|
|`SYSTEM COMMAND EXEC ASYNC`|异步执行指定的系统命令，并返回一个唯一的调用ID|
|`SYSTEM COMMAND EXEC ASYNC CANCEL `|取消指定调用ID对应的系统命令|
|`SYSTEM COMMAND EXEC ASYNC WAIT`|指定异步执行系统命令的超时事件，0为为无限长时间|
|`SYSTEM COMMAND EXEC SYNC`|同步执行系统命令，只有当前命令运行结束后，才会执行后续脚本|
|`SYSTEM COMMAND EXEC OUTPUT GET`|获得指定系统命令调用的输出|

## **四.杂项**

### **脚本执行效率的提升**
通常情况下，每一条指令运行结束后，都会伴随着这个软件界面的刷新，包括目录树，3D场景，状态栏和信息栏等。因此，为了提高执行效率可以暂时的关闭这些软件区域的刷新，如下所示，使用`Toggle`参数，可以控制相关区域刷新的开关：

```as
TREEVIEW REFRESH ( "Toggle" )
WINDOW REFRESH ( "Toggle","On","On")
```


<br/>
