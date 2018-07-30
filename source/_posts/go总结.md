---
title: go总结
date: 2018-07-17 17:13:46
tags:
---

### 1. 类型:
#### 1.1 类型分类:

- 基本类型:int,float,bool,string;
- 结构化(复合的)类型:struct,array,slice,map,channel
- 描述类型分行为:interface
- type自定义类型:type IZ int ;var a IZ = 5
    - type定义多自定义类型:
```php
type (
	IZ int
	FZ fload64
	STR string
)
```

#### 1.2 类型转换
```php
int(),float(),bool()
[]byte() //字符串转[]byte
```

#### 1.3 常量与变量
```php
<1>常量: const identifier [type] = value
const b [string] = "abc"
定义多常量:
const (
	Thursday, Friday, Saturday = 4,5,6
)
<2>变量:
var identifier [type] = value
var a, b *int
var a int = 15
var i = 5
定义多变量:
var (
	a int
	b bool
	str string
	c = false
	d = "Go says hello to the world"
)
当变量被声明时,会自动赋予零值:
float:0.0
bool:false
指针: nil
```

#### 1.4 map,数组,切片与结构体
```php
<7>map类型:
声明:
	var identifier map[keytype]valuetype
例:
	var mp1 map[string]int //这种方式在使用时需make()
	或者:
	mp2 := map[string]float32{"C":5, "Go":4.5, "Python":4.5, "C++":2}
<8>数组与切片[数组可以视为键为数字的map]:
数组声明:
	var identifier [len]type
例:
	var arr1 [5]int
切片声明:
    var identifier []type
例:
    arr := [10]{1,2,3,4}
    sli := [:] //取出所有值
    或者: sli := []int{1,2,3,4}
<9>结构体:
声明:
	type identifier struct {
		field1 type1
		field2 type2
		.......
	}
赋值:
	var i identifier
	i.field1 = int
	i.field2 = "string"
	注:此时的t是类型main.identifier
指向结构体的指针:
	1>var t *identifier
	t = new(identifier) 
	2>通常使用: t := new(identifier)
			   t.field1 = 10
			   t.field2 = 12.3
	3>指针赋值: t := &identifier{10, 12.3}
举例:
	package main

	import (
		"strings"
		"fmt"
	)

	type Person struct{
		firstName, lastName string
	}

	func upPerson(p *Person) {
		p.firstName = strings.ToUpper(p.firstName)
		p.lastName = strings.ToUpper(p.lastName)
	}

	func main() {
		//1-struct as a value type:
		var pers1 Person
		pers1.firstName = "Chris"
		pers1.lastName = "Woodward"
		upPerson(&pers1)
		fmt.Printf("%s,%s\n", pers1.firstName, pers1.lastName)

		//2-struct as a pointer:
		pers2 := new(Person)
		pers2.firstName = "Chris"
		pers2.lastName = "Woodward"
		(*pers2).lastName = "Li"
		upPerson(pers2)
		fmt.Printf("%s,%s\n", pers2.firstName, pers2.lastName)

		//3-struct as a literal
		pers3 := &Person{"Chris", "Woodward"}
		upPerson(pers3)
		fmt.Printf("%s,%s",pers3.firstName, pers3.lastName)
	}
```

#### 1.5 byte和rune

Go语言中byte和rune实质上就是uint8和int32类型。byte用来强调数据是raw data，而不是数字；而rune用来表示Unicode的code point。参考规范：
> uint8       the set of all unsigned  8-bit integers (0 to 255)
> int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
> byte        alias for uint8
> rune        alias for int32

可以通过下面程序验证：

```php
package main

import "fmt"

func byteSlience(b []byte) []byte {
	return b
}

func runeSlience(r []rune) []rune {
	return r
}
type Color byte

func main() {
	b := []byte{0,1}
	u8 := []uint8{2, 3}
	fmt.Printf("%T,%T\n", b, u8)
	fmt.Println(byteSlience(b))
	fmt.Println(byteSlience(u8))

	r := []rune{4,5}
	i32 := []int32{6,7}
	fmt.Printf("%T,%T\n", r, i32)
	fmt.Println(runeSlience(r))
	fmt.Println(runeSlience(i32))

	c := Color(3)
	fmt.Printf("%T\n", c)
	fmt.Println(c)
}
```

运行结果如下:

```php
[]uint8,[]uint8
[0 1]
[2 3]
[]int32,[]int32
[4 5]
[6 7]
main.Color
3
```

### 2. sprintf的使用

#### 2.1 定义示例类型和变量
```php
type Human struct {
	Name string
}

var people = Human{Name:"zhangsan"}
```

#### 2.2 普通占位符
```php
占位符     说明                           举例                   输出
%v      相应值的默认格式。            Printf("%v", people)   {zhangsan}，
%+v     打印结构体时，会添加字段名     Printf("%+v", people)  {Name:zhangsan}
%#v     相应值的Go语法表示            Printf("#v", people)   main.Human{Name:"zhangsan"}
%T      相应值的类型的Go语法表示       Printf("%T", people)   main.Human
%%      字面上的百分号，并非值的占位符  Printf("%%")            %
```

#### 2.3 布尔占位符
```php
占位符       说明                举例                     输出
%t          true 或 false。     Printf("%t", true)       true
```

#### 2.4 整数占位符
```php
占位符     说明                                  举例                       输出
%b      二进制表示                             Printf("%b", 5)             101
%c      相应Unicode码点所表示的字符              Printf("%c", 0x4E2D)        中
%d      十进制表示                             Printf("%d", 0x12)          18
%o      八进制表示                             Printf("%d", 10)            12
%q      单引号围绕的字符字面值，由Go语法安全地转义 Printf("%q", 0x4E2D)        '中'
%x      十六进制表示，字母形式为小写 a-f         Printf("%x", 13)             d
%X      十六进制表示，字母形式为大写 A-F         Printf("%x", 13)             D
%U      Unicode格式：U+1234，等同于 "U+%04X"   Printf("%U", 0x4E2D)         U+4E2D
```
#### 2.5 浮点数和复数的组成部分（实部和虚部）
```php
占位符     说明                              举例            输出
%b      无小数部分的，指数为二的幂的科学计数法，
        与 strconv.FormatFloat 的 'b' 转换格式一致。例如 -123456p-78
%e      科学计数法，例如 -1234.456e+78        Printf("%e", 10.2)     1.020000e+01
%E      科学计数法，例如 -1234.456E+78        Printf("%e", 10.2)     1.020000E+01
%f      有小数点而无指数，例如 123.456        Printf("%f", 10.2)     10.200000
%g      根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出 Printf("%g", 10.20)   10.2
%G      根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）输出 Printf("%G", 10.20+2i) (10.2+2i)
```

#### 2.6 字符串和字节切片
```php
占位符     说明                              举例                           输出
%s      输出字符串表示（string类型或[]byte)   Printf("%s", []byte("Go语言"))  Go语言
%q      双引号围绕的字符串，由Go语法安全地转义  Printf("%q", "Go语言")         "Go语言"
%x      十六进制，小写字母，每字节两个字符      Printf("%x", "golang")         676f6c616e67
%X      十六进制，大写字母，每字节两个字符      Printf("%X", "golang")         676F6C616E67
```
#### 2.7 指针
```php
占位符         说明                      举例                             输出
%p      十六进制表示，前缀 0x          Printf("%p", &people)             0x4f57f0
```

#### 2.8 其他标记
```php
占位符      说明                             举例          输出
+      总打印数值的正负号；对于%q（%+q）保证只输出ASCII编码的字符。 
                                           Printf("%+q", "中文")  "\u4e2d\u6587"
-      在右侧而非左侧填充空格（左对齐该区域）
#      备用格式：为八进制添加前导 0（%#o）      Printf("%#U", '中')      U+4E2D
       为十六进制添加前导 0x（%#x）或 0X（%#X），为 %p（%#p）去掉前导 0x；
       如果可能的话，%q（%#q）会打印原始 （即反引号围绕的）字符串；
       如果是可打印字符，%U（%#U）会写出该字符的
       Unicode 编码形式（如字符 x 会被打印成 U+0078 'x'）。
' '    (空格)为数值中省略的正负号留出空白（% d）；
       以十六进制（% x, % X）打印字符串或切片时，在字节之间用空格隔开
0      填充前导的0而非空格；对于数字，这会将填充移到正负号之后
```

