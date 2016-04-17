+++
date = "2015-03-03T12:50:03+08:00"
draft = false
mathjax = false
title = "TOML 简介"
categories = ["programming"]
tags = ["toml","language","tutorial"]
author = "Ryan Zhao"
keywords = ["TOML","教程","配置"]
description = "关于TOML的简单介绍"
alias = "introduction-to-toml"
+++

## TOML的由来

配置文件的使用由来已久，从.ini、XML、JSON、YAML再到TOML，语言的表达能力越来越强，同时书写便捷性也在不断提升。
TOML是前GitHub CEO， Tom Preston-Werner，于2013年创建的语言，其目标是成为一个小规模的易于使用的语义化配置文件格式。TOML被设计为可以无二义性的转换为一个哈希表(Hash table)。

<!--more-->
## 例子

```
# 这是一个TOML文件

title = "TOML Example"

[owner]
name = "Lance Uppercut"
dob = 1979-05-27T07:32:00-08:00 # 日期是一等公民

[database]
server = "192.168.1.1"
ports = [ 8001, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]
  #你可以使用空格、制表符进行缩进，或者根本不缩进。TOML不关心缩进。
  [servers.alpha]
  ip = "10.0.0.1"
  dc = "eqdc10"

  [servers.beta]
  ip = "10.0.0.2"
  dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ]

# 数组内可以混入换行符
hosts = [
  "alpha",
  "omega"
]

```

## 规范

- TOML是大小写敏感的
- TOML文件必须是UTF8编码的
- 空白符可以是制表符(0x09)或空格(0x20)
- 换行符可以是 LF (0x0A) 或 CRLF (0x0D0A)

TOML仍在不断完善，目前的版本0.4.0，下面是最新的规范。

### 注释

使用\#来表示注释开始，至当前行尾结束。
```
# I am a comment. Hear me roar. Roar.
key = "value" # Yeah, you can do this.
```

### 字符串

TOML中有4种字符串表示方法：基本、多行-基本、字面量、多行-字面量。所有字符串必须是合法的UTF8字符。

**基本字符串**由双引号包裹，所有Unicode字符均可出现，除了双引号、反斜线、控制字符(U+0000 to U+001F)需要转义。
```
str = "I'm a string. \"You can quote me\". Name\tJos\u00E9\nLocation\tSF."
```
常用的转义序列：
```
\b         - backspace       (U+0008)
\t         - tab             (U+0009)
\n         - linefeed        (U+000A)
\f         - form feed       (U+000C)
\r         - carriage return (U+000D)
\"         - quote           (U+0022)
\\         - backslash       (U+005C)
\uXXXX     - unicode         (U+XXXX)
\UXXXXXXXX - unicode         (U+XXXXXXXX)
```

**多行-基本字符串**由三个双引号包裹，除了分隔符开始的换行外，字符串内的换行将被保留。
```
str1 = """
Roses are red
Violets are blue"""
```
TOML解析器可以将其翻译为平台相关的字符串，如
```
# Unix上，上述字符串等同于
str2 = "Roses are red\nViolets are blue"

# Windows上，上述字符串等同于
str3 = "Roses are red\r\nViolets are blue"
```

在**多行-基本字符串**中可以在行尾使用```\```来忽略其后的所有(换行符和空白符）直到第一个非空白符。
```
# 以下字符串等价
str1 = "The quick brown fox jumps over the lazy dog."

str2 = """
The quick brown \


  fox jumps over \
    the lazy dog."""

key3 = """\
       The quick brown \
       fox jumps over \
       the lazy dog.\
       """
```

**字面量字符串**由单引号包裹，其内不允许转义，因此可以方便的表示基本字符串中需要转义的内容。
```
# What you see is what you get.
winpath  = 'C:\Users\nodejs\templates'
winpath2 = '\\ServerX\admin$\system32\'
quoted   = 'Tom "Dubs" Preston-Werner'
regex    = '<\i\c*\s*>'
```

**多行-字面量字符串**与多行-基本字符串类似。

### 整数
```
int1 = +99
int2 = 42
int3 = 0
int4 = -17
```
为了增加可读性，整数可以使用```_```分隔。每个```_```必须被至少一个数字环绕。

不被允许的表达：前置0，2、8、16进制、无穷、NaN。
整数的范围是64bit signed long类型的范围。

### 浮点数
```
# fractional
flt1 = +1.0
flt2 = 3.1415
flt3 = -0.01

# exponent
flt4 = 5e+22
flt5 = 1e6
flt6 = -2E-2

# both
flt7 = 6.626e-34
```
浮点数的范围是64 bit double类型的范围。

### 布尔值
小写的true或false。
```
bool1 = true
bool2 = false
```

### 日期时间
使用RFC 3339描述的时间格式
```
date1 = 1979-05-27T07:32:00Z
date2 = 1979-05-27T00:32:00-07:00
date3 = 1979-05-27T00:32:00.999999-07:00
```

### 数组
数组使用方括号包裹。空格会被忽略，包括换行符。元素使用逗号分隔。注意，不允许混用数据类型(所有的字符串类型均为同一类型)。
```
arr1 = [ 1, 2, 3 ]
arr2 = [ "red", "yellow", "green" ]
arr3 = [ [ 1, 2 ], [3, 4, 5] ]
arr4 = [ "all", 'strings', """are the same""", '''type'''] # this is ok
arr5 = [ [ 1, 2 ], ["a", "b", "c"] ] # this is ok
arr6 = [ 1, 2.0 ] # note: this is NOT ok
```

### 表格
表格叶称为哈希表或字典，用来存储键值对。表格名由方括号包裹，且自成一行。
```
[table]
```
表格名下，直到下一个表格名或文件尾，均为当前表格的内容。
```
[table]
key = "value"
bare_key = "value"
bare-key = "value"

"127.0.0.1" = "value"
"character encoding" = "value"
"ʎǝʞ" = "value"
```

表格可以嵌套，即表格中某个键的值可以为表格。
```
[dog]
onekey = onevalue

[dog.tater]
type = "pug"
```
等价于
```
{
	"dog": { 
		"onekey":"onevalue",
		"tater": { "type": "pug" } 
	} 
}
```
如果你不想的话，你不用声明所有的父表。TOML　知道该如何处理。

```
# [x] 你
# [x.y] 不需要
# [x.y.z] 这些
[x.y.z.w] # 可以直接写
```

### 表格数组
```
[[products]]
name = "Hammer"
sku = 738594937

[[products]]

[[products]]
name = "Nail"
sku = 284758393
color = "gray"
```
等价于以下的　JSON 结构：
```
{
  "products": [
    { "name": "Hammer", "sku": 738594937 },
    { },
    { "name": "Nail", "sku": 284758393, "color": "gray" }
  ]
}
```

详细信息可以参考[toml-lang/toml at github](https://github.com/toml-lang/toml)。


> Written with [StackEdit](https://stackedit.io/).
