---
title: go深入
date: 2018-08-15 15:43:17
tags:
---
### 1.main函数和init函数
简介:init函数能够在所有的package使用,main函数只能用于package main,这两个函数在定义时不能有任何参数和返回值,每个package中的init函数都是可选的，但package main就必须包含一个main函数。
<img src="http://resource.cmdapps.com/2018/08/main.init.png">
<center>*main函数引包流程图</center>
例子:
- main.go
```go
package main

import (
	"time"
	"fmt"
	"net/http"
	_ "test/test"
)

const (
	Name = "main"
)

func init() {
	fmt.Printf("I am in %s package.\n", Name)
}
```
- test/test.go
```go
package test

import (
	"path/filepath"
	"os"
	"fmt"
)

var name = "anker"

func init() {
	fmt.Printf("I am in test package and my name is %s.\n", name)
}
```

### 2.路由绑定与session搭配提供web服务

```php
package main

import "net/http"

func SayHello(w http.ResponseWriter, req *http.Request) {
	w.Write([]byte("hello"))
}

func ReadCookieServe(w http.ResponseWriter, req *http.Request) {
	cookie, err := req.Cookie("testcookiename")
	if err == nil {
		cookievalue := cookie.Value
		w.Write([]byte("<b>cookie的值是:"+cookievalue+"</b>\n"))
	} else {
		w.Write([]byte("<b>读取出现错误:"+err.Error()+"</b>\n"))
	}
}

func WriteCookieServe(w http.ResponseWriter, req *http.Request) {
	cookie := http.Cookie{Name: "testcookiename", Value: "testcookievalue", Path: "/", MaxAge: 86400}
	http.SetCookie(w, &cookie)
	w.Write([]byte("<b>设置cookie成功!</b>\n"))
}

func DeleteCookieServe(w http.ResponseWriter, req *http.Request) {
	cookie := http.Cookie{Name: "testcookiename", Path: "/", MaxAge: -1}
	http.SetCookie(w, &cookie)
	w.Write([]byte("<b>删除cookie成功。</b>\n"))
}

func main() {
	http.HandleFunc("/", SayHello)
	http.HandleFunc("/readcookie", ReadCookieServe)
	http.HandleFunc("/writecookie", WriteCookieServe)
	http.HandleFunc("/deletecookie", DeleteCookieServe)

	http.ListenAndServe(":8080", nil)
}
```

### 3.defer深入
```php
package main

import (
	"time"
	"fmt"
)

func trace(funcName string) func() {
	start := time.Now()
	fmt.Printf("function %s enter\n", funcName)
	return func() {
		fmt.Printf("function %s exit (elapsed %s)\n", funcName, time.Since(start))
	}
}

func foo() {
	defer trace("foo()")()  //defer后面的函数值和参数会被求值但实际的函数调用要等到最后
	time.Sleep(5*time.Second)
}

func main() {
	foo()
	foo()
}
```
### 4.表单处理
login.tpl
```php
<html>
<head>
<title></title>
</head>
<body>
<form action="/login" method="post">
	用户名:<input type="text" name="username">
	密码:<input type="password" name="password">
	<input type="submit" value="登录">
</form>
</body>
</html>
```

main.go
```php
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
	"strings"
)

func login(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) //获取请求的方法
	if r.Method == "GET" {
		t, _ := template.ParseFiles("login.gtpl")
		log.Println(t.Execute(w, nil))
	} else {
		//请求的是登录数据，那么执行登录的逻辑判断
		fmt.Println("username:", r.Form["username"][0])
		fmt.Println("password:", r.Form.Get("password"))
	}
}

func main() {
	http.HandleFunc("/login", login)         //设置访问的路由
	err := http.ListenAndServe(":8080", nil) //设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```

### 5.数据库操作

> 说明:
> <1> go语言没有提供数据库驱动,只提供了接口,常用的mysql数据库驱动为github.com/go-sql-driver/mysql
> <2> dsn格式: username:password@protocol(address)/dbname?param=value

> 函数解释:
> sql.Open()函数用来打开一个注册过的数据库驱动，go-sql-driver中注册了mysql这个数据库驱动
> db.Prepare()函数用来返回准备要执行的sql操作，然后返回准备完毕的执行状态。
> db.Query()函数用来直接执行Sql返回Rows结果。
> stmt.Exec()函数用来执行stmt准备好的SQL语句

#### 数据表结构
```mysql
CREATE TABLE `userinfo` (
	`uid` INT(10) NOT NULL AUTO_INCREMENT,
	`username` VARCHAR(64) NULL DEFAULT NULL,
	`department` VARCHAR(64) NULL DEFAULT NULL,
	`created` DATE NULL DEFAULT NULL,
	PRIMARY KEY (`uid`)
);

CREATE TABLE `userdetail` (
	`uid` INT(10) NOT NULL DEFAULT '0',
	`intro` TEXT NULL,
	`profile` TEXT NULL,
	PRIMARY KEY (`uid`)
)
```
#### 代码示例
```go
package test

import (
	_ "github.com/go-sql-driver/mysql"
	"database/sql"
	"fmt"
)

func TestDb() {
	//连接
	db, err := sql.Open("mysql", "root:123456aA@tcp(127.0.0.1:8889)/test1?charset=utf8") //打开一个注册过的驱动
	checkErr(err)

	//插入数据
	stmt, _ := db.Prepare("INSERT `userinfo` SET `username`=?, `department`=?, `created`=?")
	res, _ := stmt.Exec("anker", "研发部", "2018--08-17") //insert,update或delete时使用
	id, _ := res.LastInsertId()
	fmt.Println(id)

	//查询数据
	rows, _ := db.Query("SELECT * FROM `userinfo`")
	for rows.Next() {
		var uid int
		var username string
		var department string
		var created string
		err = rows.Scan(&uid, &username, &department, &created)
		checkErr(err)
		fmt.Println(uid)
		fmt.Println(username)
		fmt.Println(department)
		fmt.Println(created)
	}
}

func checkErr(err error) {
	if err != nil {
		panic(err)
	}
}
```
### 6.orm操作数据库
采用了Go style方式对数据库进行操作，实现了struct到数据表记录的映射。

#### 安装
> go get github.com/astaxie/beego

#### 初始化
```go
func init() {
	//注册驱动
	orm.RegisterDriver("mysql", orm.DRMySQL)
	//设置默认数据库
	orm.RegisterDataBase("default", "mysql", "root:123456aA@tcp(127.0.0.1:8889)/test1?charset=utf8")
	//注册定义的model
	orm.RegisterModel(new(Userinfo))

	//create table
	orm.RunSyncdb("default", false, true)
}
```

#### 完整代码
```go
package test

import (
	"github.com/astaxie/beego/orm"
	"fmt"
	"time"
)

//Model Struct
type Userinfo struct {
	Id int `orm:"pk;column(uid)"` //设置主键
	Username string
	Department string
	Created string
}

func init() {
	//注册驱动
	orm.RegisterDriver("mysql", orm.DRMySQL)
	//设置默认数据库
	orm.RegisterDataBase("default", "mysql", "root:123456aA@tcp(127.0.0.1:8889)/test1?charset=utf8")
	//注册定义的model
	orm.RegisterModel(new(Userinfo))

	//create table
	orm.RunSyncdb("default", false, true)
}

func TestOrm() {
	o := orm.NewOrm() //创建orm对象

	userinfo := Userinfo{
		Username: "lizengcai",
		Department: "企划部",
		Created: time.Now().Format("2006-01-02 15:04:05"),
	}

	//插入表
	o.Insert(&userinfo)

	//更新表
	userinfo.Id = 8
	userinfo.Username = "anker"
	num, err := o.Update(&userinfo, "username")
	fmt.Println("Num:", num, "Err:",err)

	//读取一行
	userinfo := Userinfo{Id: 8}
	o.Read(&userinfo)
	fmt.Println(userinfo)

	//按条件获取
	var userinfoes []Userinfo

	qs := o.QueryTable("userinfo")
	cond := orm.NewCondition()
	cond = cond.And("username", "lizengcai")
	qs = qs.SetCond(cond)
	num,_ := qs.All(&userinfoes)
	fmt.Println(num, userinfoes)
}
```
































