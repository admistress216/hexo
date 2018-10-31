---
title: api接口定义
date: 2018-10-10 14:42:15
tags:
---
### 1. REST API
```php

Method          URI                             Comment
post            api/login                       登录
post            api/logout                      退出登录

get             api/users/{userid}              获取单个用户信息
put             api/users/{userid}              更新用户
delete          api/users/{userid}              删除用户
get             api/users/list                  获取用户列表
get             api/role_company_list           获取角色,公司,部门
post            api/users                       新建cms用户

get             api/roles/list                  角色列表展示
post            api/roles/add                   新增角色
put             api/roles/{roleid}/edit         角色编辑
get             api/rolepriv/{roleid}           角色管理中角色授权页面展示
put             api/rolepriv/{userid}/upd       更新角色权限
get             api/usermenu/{role_id}          获取用户菜单

```

### 2. 登录相关API

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

### 3. 用户相关API
#### 3.1 获取单个用户信息
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

#### 3.2 更新用户信息
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
| role_id | string | no | 角色id | 无 | 1 |
| companyid | string | yes | 公司id | 无 | 10001 |
| user_password | string | yes | 用户密码 | 无 | $2y$10$jrYGOgJPS/Rs.AAVyesEE./Oq8coNKU/Is0yBV2KQGXlngZ6no6Di |


| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |

#### 3.3 删除用户
> Method: delete
> URI: api/users/{userid}

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| userid | string | yes | 用户名 | 无 | 1 |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- | :--- | :--- |

#### 3.4 获取用户列表
> Method: get
> URI: api/users/list

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |
| page | string | yes | 页数 | 无 | 1 |
| limit | string | yes | 条数 | 无 | 10 |
| user_name | string | yes | 用户名称 | 无 | xiaoming |
| user_nickname | string | yes | 用户昵称 | 无 | 小明 |
| mobile | string | yes | 手机号 | 无 | 13341028691 |
| t_status | int | yes | 状态(0:关闭,1:开启) | 无 | 1 |
| role_id | int | yes | 角色id | 无 | 1 |
| alone_role_id | int | yes | 独立授权角色id | 无 | 1 |
| group_id | int | yes | 分组id | 无 | 1 |
| companyid | int | yes | 公司id | 无 | 1 |


| 出参 | 类型 | 参数说明 | 例子 |
| :--- | :--- | :--- | :--- | :--- |
| id | string | ID | 1009 |
| user_name | string | 用户名 | xiaoming |
| user_nickname | string | 昵称 | 小明 |
| mobile | string | 手机号 | 13341028691 |
| role_id | int | 角色id | 1 |
| role_name | string | 角色名称 | 最高运维权限 |
| company_id | int | 公司id | 1 |
| company_name | string | 公司名称 | 央视网 |
| group_name | string | 分组名称 | 技术部 |
| expirydate | string | 过期时间 | 2019-05-31 |
| actionBtn | string | 动作按钮 | .... |

#### 3.5 获取角色,公司,部门
> Method: get
> URI: api/role_company_list

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 | 例子 |
| :--- | :--- | :--- | :--- | :--- |
| role_lists.rid | int | 角色ID | 1009 |
| role_lists.rolename | string | 角色名称 | 机构管理员 |
| role_lists.role_company_id | int | 角色对应公司id | 10001 |
| company_lists.id | int | 公司id | 10001 |
| company_lists.name | string | 公司名称 | 央视新闻移动网 |
| group_data.reporter_group_id | int | 部门id | 303 |
| group_data.reporter_group_name | string | 部门名称 | 技术部 |
| group_data.company_id | int | 部门所属公司id | 10001 |
| group_data.parent_id | int | 父级id | 302 |

#### 3.6 新建cms用户
> Method: post
> URI: api/users

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |
| user_name | string | yes | 登录名 | 无 | xiaoming |
| user_nickname | string | yes | 昵称 | 无 | 小明 |
| mobile | string | yes | 手机号 | 无 | 13312341234 |
| t_status | string | yes | 状态(0:关闭,1:开启) | 无 | 1 |
| reporter_group_name | yes | string | 部门分组 | 无 | 技术部/合作组 |
| expirydate | string | yes | 过期时间 | 无 | 2019-09-20 |
| description | string | yes | 描述 | 无 | i am a description |
| role_id | string | no | 角色id | 无 | 1 |
| companyid | string | yes | 公司id | 无 | 10001 |
| user_password | string | yes | 用户密码 | 无 | $2y$10$jrYGOgJPS/Rs.AAVyesEE./Oq8coNKU/Is0yBV2KQGXlngZ6no6Di |

| 出参 | 类型 | 参数说明 |
| :--- | :--- | :--- |
| access_token | string | 令牌 |
| user_id | int | 用户id |

### 4. 权限相关API
#### 4.1 角色列表展示
> Method: get
> URI: api/roles/list

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 | 例子 |
| :--- | :--- | :--- | :--- | :--- |
| role | int | 角色ID | 1009 |
| role_name | string | 角色名称 | 机构管理员 |
| company_name | string | 公司名称 | 央视新闻移动网 |
| cactionBtn | string | 动作按钮 | .... |

#### 4.2 新增角色
> Method: post
> URI: api/roles/add

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |
| role_name | string | yes | 角色名称 | 无 | 运维 |
| role_company_id | int | yes | 公司id | 无 | 10001 |
| role_status | string | yes | 状态,0:冻结,1:正常 | 无 | 1 |

#### 4.3 角色编辑
> Method: put
> URI: api/roles/{roleid}/edit

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |
| roleid | int | yes | 角色id | 无 | 1 |
| role_name | string | yes | 角色名称 | 无 | 运维 |
| role_company_id | int | yes | 公司id | 无 | 10001 |
| role_status | string | yes | 状态,0:冻结,1:正常 | 无 | 1 |

#### 4.4 角色管理中角色授权页面展示
> Method: get
> URI: api/rolepriv/{roleid}

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| roleid | string | yes | 角色id | 无 | 1 |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- |
| role_info.role_id | int | 角色id | 无 | 2 |
| role_info.role_name | string | 角色名称 | 无 | 央视新闻移动网版面编辑 |
| role_info.role_company_id | string | 公司id | 无 | 10001 |
| role_info.role_shortname | string | 别名 | 无 | admin |
| role_info.role_status | string | 角色状态,1:正常,-1:删除 | 无 | 1 |
| role_info.role_alone | string | 是否是单独角色,2:单独个人角色 | 无 | 1 |
| role_id | int | 角色id | 无 | 2 |
| priv_list[0].pid | string | 节点id | 无 | 232 |
| priv_list[0].menuName | string | 节点名称 | 无 | 新建 |
| priv_list[0].module | string | 模块 | 无 | new |
| priv_list[0].controller | string | 控制器 | 无 | Manuscript |
| priv_list[0].action | string | 动作 | 无 | addPageContactManuscript |
| priv_list[0].parentId | string | 父级id | 无 | 0 |
| priv_list[0].isMenu | string | 是否是菜单项 | 无 | 1 |
| priv_list[0].menuDesc | string | 菜单描述 | 无 | 新增/编辑稿件（包含短视频发布/直播发布） |
| priv_list[0].yes | string | 是否选中,1:选中,0:未选中 | 无 | 1 |
| priv_list[0].child[0].pid | int | 子节点id | 无 | 190 |
| priv_list[0].child[0].menuName | string | 子节点名称 | 无 | 新建稿件 |
| ...... | ....... | ....... | ...... | ......... |
| priv_list[0].child[0].child[0].menuName | string | 子节点名称 | 无 | 新建稿件 |
| ...... | ....... | ....... | ...... | ......... |

#### 4.5 更新角色权限
> Method: put
> URI: api/rolepriv/{userid}/upd

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |
| roleid | int | yes | 角色id | 无 | 1 |
| node.nodeid | int | 授权节点id | 无 | 对应的nodeid |

#### 4.6 获取用户菜单
> Method: get
> URI: api/usermenu/{role_id}

| 入参 | 类型 | 是否必须 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| alone_roleid | string | yes | 用户角色id | 无 | 1 |
| access_token | string | yes | 令牌 | 无 | xxx.yyy.zzz |

| 出参 | 类型 | 参数说明 | 默认值 | 例子 |
| :--- | :--- | :--- | :--- | :--- |
| menu[0].pid | string | 节点id | 无 | 232 |
| menu[0].menuName | string | 节点名称 | 无 | 新建 |
| menu[0].module | string | 模块 | 无 | new |
| menu[0].controller | string | 控制器 | 无 | Manuscript |
| menu[0].action | string | 动作 | 无 | addPageContactManuscript |
| menu[0].parentId | string | 父级id | 无 | 0 |
| menu[0].isMenu | string | 是否是菜单项 | 无 | 1 |
| menu[0].child[0].pid | int | 子节点id | 无 | 190 |
| menu[0].child[0].menuName | string | 子节点名称 | 无 | 新建稿件 |
| ...... | ....... | ....... | ...... | ......... |
| priv_list[0].child[0].menuName | string | 子节点名称 | 无 | 新建稿件 |
| ...... | ....... | ....... | ...... | ......... |