---
title: go总结
date: 2018-07-17 17:13:46
tags:
---

### 1.类型:
#### 1.1类型分类:

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

#### 1.2类型转换
```php
int(),float(),bool()
[]byte() //字符串转[]byte
```

#### 1.3常量与变量
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

#### 1.4map,数组,切片与结构体
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

#### 1.5byte和rune

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