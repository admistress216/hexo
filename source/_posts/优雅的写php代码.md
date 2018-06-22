---
title: 优雅的写php代码
date: 2018-06-22 00:42:38
tags:
---
1. 遍历数组获取新的数据结构<br/>
  
```
$arr = [
    'test1' => 'John',
    'test2' => 'Jerry',
    'test3' => 'Marry',
];

//方式一:传统方式(不推荐):增加内存开销
$tmp = [];
foreach($arr as $key => $val) {
    if (in_array($val, ['John', 'Jerry'], true)) {
        $val = 'Anker';
    }
    $tmp[$key] = $val;
}

//方式二:
foreach($arr as $key => $val) {
    if ($val === 'John' || $val === 'Jerry') {
        $arr[$key] = 'Anker';
    }
}

//方式三:引用(代码简洁,节省内存空间)
foreach($arr as $key => &$val) {
    if (in_array($val, ['John', 'Jerry'], true)) {
        $val = 'Anker';
    }
}
unset($val);

```
