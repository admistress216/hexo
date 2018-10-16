---
title: 优雅的写php代码
date: 2018-06-22 00:42:38
categories:
    - php总结
---
### 1. 遍历数组获取新的数据结构

  
``` php
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

### 2.ajax返回错误状态使用try...catch...
  
下面有段逻辑:

``` php
class UserModel
{
    public function login($username = '', $password = '')
    {
        code...
        if (...) {
            // 用户不存在
            return -1;
        }
        code...
        if (...) {
            // 密码错误
            return -2;
        }
        code...
    }
}

class UserController
{
    public function login($username = '', $password = '')
    {
        $model = new UserModel();
        $res   = $model->login($username, $password);
        if ($res === -1) {
            return [
                'code' => '404',
                'message' => '用户不存在'
            ];
        }
        if ($res === -2) {
            return [
                'code' => '400',
                'message' => '密码错误'
            ];
        }
        code...
    }
}

```

 使用try...catch...改写后:
 
``` php
class UserModel
{
    public function login($username = '', $password = '')
    {
        code...
        if (...) {
            // 用户不存在
            throw new Exception('用户不存在', '404');
        }
        code...
        if (...) {
            // 密码错误
            throw new Exception('密码错误', '400');
        }
        code...
    }
}

class UserController
{
    public function login($username = '', $password = '')
    {
        try {
            $model = new UserModel();
            $res   = $model->login($username, $password);
            // 如果需要的话，我们可以在这里统一commit数据库事务
            // $db->commit();
        } catch (Exception $e) {
            // 如果需要的话，我们可以在这里统一rollback数据库事务
            // $db->rollback();
            return [
                'code'    => $e->getCode(),
                'message' => $e->getMessage()
            ]
        }
    }
}

```