---
title: jaml语言总结
date: 2019-01-16 15:17:15
tags:
categories: [markup language]
---

[参考链接](http://www.ruanyifeng.com/blog/2016/07/yaml.html)
本文介绍YAML的语法,以[JS-YAML](https://github.com/nodeca/js-yaml)的实现为例,[在线验证地址](http://nodeca.github.io/js-yaml/)
### 1. 简介
YAML 语言（发音 /ˈjæməl/ ）的设计目标，就是方便人类读写。它实质上是一种通用的数据串行化格式。
它的基本语法规则如下。

> - 大小写敏感
> - 使用缩进表示层级关系
> - 缩进时不允许使用Tab键，只允许使用空格。
> - 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

`#` 表示注释，从这个字符一直到行尾，都会被解析器忽略。
YAML 支持的数据结构有三种。

> - 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
> - 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
> - 纯量（scalars）：单个的、不可再分的值

### 2. 对象
```php
//对象的一组键值对，使用冒号结构表示。
animal: pets
//转为 JavaScript 如下。
{ animal: 'pets' }

//Yaml 也允许另一种写法，将所有键值对写成一个行内对象。
hash: { name: Steve, foo: bar }
hash:
 name: Steve
 foo: bar
//转为 JavaScript 如下。
{ hash: { name: 'Steve', foo: 'bar' } }
```

### 3. 数组
```php
//一组连词线开头的行，构成一个数组。
- Cat
- Dog
- Goldfish
//转为 JavaScript 如下。
{[ 'Cat', 'Dog', 'Goldfish' ]}

//数据结构的子成员是一个数组，则可以在该项下面缩进一个空格。
animal1:
 - Cat
 - Dog
//转为 JavaScript 如下。
{animal1: [ 'Cat', 'Dog' ]}

//数组也可以采用行内表示法。
animal: [Cat, Dog]
//转为 JavaScript 如下。
{ animal: [ 'Cat', 'Dog' ] }
```

### 4. 引用
```php
锚点&和别名*，可以用来引用。
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
等同于下面的代码。

defaults:
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  adapter:  postgres
  host:     localhost

test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
&用来建立锚点（defaults），<<表示合并到当前数据，*用来引用锚点。

下面是另一个例子。
- &showell Steve 
- Clark 
- Brian 
- Oren 
- *showell 
转为 JavaScript 代码如下。
[ 'Steve', 'Clark', 'Brian', 'Oren', 'Steve' ]
```

### 5. 其他:见参考链接
























