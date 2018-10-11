---
title: api接口定义
date: 2018-10-10 14:42:15
tags:
---
### 1. REST API
```php
Method          URI                     Action      Comment
post            api/login               store       登录
post            api/logout              logout      退出登录

get             api/users               index       获取用户列表
get             api/users/{userid}      show        获取单个用户信息
get             api/users/{userid}/edit edit        编辑用户
put             api/users/{userid}      update      更新用户
delete          api/users/{userid}      destroy     删除用户
```

### 2. API详情

#### 2.1 登录
> Method: post
> URI: api/login

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| email | string | yes | 用户名 | 无 | lynch.bertram@example.com |
| password | string | yes | 加密密码 | 无 | rtrim(strtr(base64_encode('password'), '+/', '-_'), '=') |

| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |
| access_token | string | 令牌 |
| id | int | 用户id |
| user_nickname | string | 用户昵称 |
|avatar | string | 头像 |

#### 2.2 退出登录
> Method: post
> URI: api/logout

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |

#### 2.3 获取单个用户信息
> Method: get
> URI: api/users/{userid}

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| id | int | yes | 用户id | 无 | 123 |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- |
| user_name | string | 登录名 | 无 | xiaoming |
| user_nickname | string | 昵称 | 无 | 小明 |
| mobile | string | 手机号 | 无 | 13312341234 |
| email | string | 邮箱 | 无 | xiaoming@163.com |
| avatar | string | 头像 | 无 | http://www.test.com/1.jpg |
| role_name | string | 角色名称 | 无 | 系统管理员 |
| company_name | string | 公司名称 | 无 | 央视新闻移动网 |
| expirydate | string | 过期时间 | 无 | 2019-09-20 |
| t_status | string | 状态(0:关闭,1:开启) | 无 | 1 |
| reporter_group_name | string | 部门分组 | 无 | 技术部/合作组 |



#### 2.4 编辑用户信息
> Method: get
> URI: api/users/{userid}/edit

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| userid | string | yes | 用户名 | 无 | 1 |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |

#### 2.5 更新用户信息
> Method: put
> URI: api/users/{userid}

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| userid | string | yes | 用户名 | 无 | 1 |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |
| user_name | string | yes | 登录名 | 无 | xiaoming |
| user_nickname | string | yes | 昵称 | 无 | 小明 |
| mobile | string | yes | 手机号 | 无 | 13312341234 |
| t_status | string | yes | 状态(0:关闭,1:开启) | 无 | 1 |
| reporter_group_name | yes | string | 部门分组 | 无 | 技术部/合作组 |
| expirydate | string | yes | 过期时间 | 无 | 2019-09-20 |
| description | string | yes | 描述 | 无 | i am a description |
| role_name | string | yes | 角色名称 | 无 | 系统管理员 |
| company_name | string | yes | 公司名称 | 无 | 央视新闻移动网 |
| user_password | string | yes | 用户密码 | 无 | $2y$10$jrYGOgJPS/Rs.AAVyesEE./Oq8coNKU/Is0yBV2KQGXlngZ6no6Di |


| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |

#### 2.6 删除用户
> Method: delete
> URI: api/users/{userid}

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| userid | string | yes | 用户名 | 无 | 1 |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |
