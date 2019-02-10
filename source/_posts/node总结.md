---
title: node总结
date: 2019-01-25 15:18:07
tags:
---

### 1. 安装nodejs与npm
```php
//方式一: Linux上安装已经编译的:
# wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz    // 下载
# tar xf  node-v10.9.0-linux-x64.tar.xz       // 解压
# cd node-v10.9.0-linux-x64/                  // 进入解压目录
# ./bin/node -v                               // 执行node命令 查看版本
v10.9.0
# ln -s /usr/software/nodejs/bin/npm   /usr/local/bin/ 
# ln -s /usr/software/nodejs/bin/node   /usr/local/bin/ //设置软连接

//方式二: Centos下源码安装
1. 下载源码
cd /usr/local/src/
wget http://nodejs.org/dist/v0.10.24/node-v0.10.24.tar.gz
2. 解压编译
tar zxvf node-v0.10.24.tar.gz
cd node-v0.10.24
./configure --prefix=/usr/local/node/0.10.24
make
make install
3.配置NODE_HOME
vim /etc/profile
#set for nodejs(在export PATH USER....之前)
export NODE_HOME=/usr/local/node/0.10.24
export PATH=$NODE_HOME/bin:$PATH
```

### 2.npm(node package manage)相关操作
> 在 PHP 中, 包管理使用的 Composer, 
> python 中，包管理使用 easy_install 或者 pip，
> ruby 中我们使用 gem。
> 而在 Node.js 中，对应就是 npm，npm 是 Node.js Package Manager 的意思。

```php
npm init: 初始化npm项目(作者,描述等),并生成package.json和package-lock.json文件
npm install xxx: 当前目录安装xxx模块
npm uninstall xxx: 卸载xxx模块
npm list: npm安装的包列表(当前目录)
npm list -g --depth 0: 全局安装的包列表(依赖深度为0)
```

### 3.nvm(node版本管理工具)
```php
//安装:
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
//nvm有输出则代表安装成功
nvm ls-remote:查看远程支持的版本
nvm install node: 默认安装最新的版本
nvm install versionid: 安装对应版本
nvm uninstall versionid: 卸载对应版本
nvm ls: 查看本地安装的node版本
nvm use versionid: 使用对应版本的node

//用nvm安装node之后，我每次启动终端 的时候都要重新nvm use v7.4 
//解决办法输入命令
nvm alias default stable
```

### 4. [课程](https://gitee.com/lizengcai/node-lessons)
#### 4.11 express框架
```php
//官网地址
http://expressjs.com/ 
$ mkdir lesson1 && cd lesson1
# 这里没有从官方 npm 安装，而是使用了大淘宝的 npm 镜像
$ npm install express --registry=https://registry.npm.taobao.org
$ ls node_modules
#里面如果出现 express 文件夹则说明安装成功。npm list --depth 0更直观
```

- 编写代码
```php
# app.js

// 这句的意思就是引入 `express` 模块，并将它赋予 `express` 这个变量等待使用。
var express = require('express');
// 调用 express 实例，它是一个函数，不带参数调用时，会返回一个 express 实例，将这个变量赋予 app 变量。
var app = express();

// app 本身有很多方法，其中包括最常用的 get、post、put/patch、delete，在这里我们调用其中的 get 方法，为我们的 `/` 路径指定一个 handler 函数。
// 这个 handler 函数会接收 req 和 res 两个对象，他们分别是请求的 request 和 response。
// request 中包含了浏览器传来的各种信息，比如 query 啊，body 啊，headers 啊之类的，都可以通过 req 对象访问到。
// res 对象，我们一般不从里面取信息，而是通过它来定制我们向浏览器输出的信息，比如 header 信息，比如想要向浏览器输出的内容。这里我们调用了它的 #send 方法，向浏览器输出一个字符串。
app.get('/', function (req, res) {
  res.send('Hello World');
});

// 定义好我们 app 的行为之后，让它监听本地的 3000 端口。这里的第二个函数是个回调函数，会在 listen 动作成功后执行，我们这里执行了一个命令行输出操作，告诉我们监听动作已完成。
app.listen(3000, function () {
  console.log('app is listening at port 3000');
});
//执行node app.js
```

#### 4.12 学习使用外部模块
- 知识点
1. 学习 req.query 的用法
2. 学习建立 package.json 来管理 Node.js 项目。

> 简单说来呢，这个 package.json 文件就是定义了项目的各种元信息，包括项目的名称，git repo 的地址，作者等等。最重要的是，其中定义了我们项目的依赖，这样这个项目在部署时，我们就不必将 node_modules 目录也上传到服务器，服务器在拿到我们的项目时，只需要执行 npm install，则 npm 会自动读取 package.json 中的依赖并安装在项目的 node_modules 下面，然后程序就可以在服务器上跑起来了。

```php
//我们来新建一个 lesson2 项目，并生成一份它的 package.json。
$ mkdir lesson2 && cd lesson2
$ npm init
//OK，这时会要求我们输入一些信息，乱填就好了，反正这个地方也不用填依赖关系。
//npm init 这个命令的作用就是帮我们互动式地生成一份最简单的 package.json 文件，init 是 initialize 的意思，初始化。
//当乱填信息完毕之后，我们的目录下就会有个 package.json 文件了。
//这时我们来安装依赖，这次的应用，我们依赖 express 和 utility 两个模块。
$ npm install express utility --save
//这次的安装命令与上节课的命令有两点不同，一是没有指定 registry，没有指定的情况下，默认从 npm 官方安装，上次我们是从淘宝的源安装的。二是多了个 --save 参数，这个参数的作用，就是会在你安装依赖的同时，自动把这些依赖写入 package.json。命令执行完成之后，查看 package.json，会发现多了一个 dependencies 字段，如下：
  "dependencies": {
    "express": "^4.16.4",
    "utility": "^1.15.0"
  }
$npm list --depth 0
    lesson2@1.0.0 /project/lesson2
    ├── express@4.16.4
    └── utility@1.15.0
```

- 编写代码
```node
#app.js

// 引入依赖
var express = require('express');
var utility = require('utility');

// 建立 express 实例
var app = express();

app.get('/', function (req, res) {
  // 从 req.query 中取出我们的 q 参数。
  // 如果是 post 传来的 body 数据，则是在 req.body 里面，不过 express 默认不处理 body 中的信息，需要引入 https://github.com/expressjs/body-parser 这个中间件才会处理，这个后面会讲到。
  // 如果分不清什么是 query，什么是 body 的话，那就需要补一下 http 的知识了
  var q = req.query.q;

  // 调用 utility.md5 方法，得到 md5 之后的值
  // 之所以使用 utility 这个库来生成 md5 值，其实只是习惯问题。每个人都有自己习惯的技术堆栈，
  // 我刚入职阿里的时候跟着苏千和朴灵混，所以也混到了不少他们的技术堆栈，仅此而已。
  // utility 的 github 地址：https://github.com/node-modules/utility
  // 里面定义了很多常用且比较杂的辅助方法，可以去看看
  var md5Value = utility.md5(q);

  res.send(md5Value);
});

app.listen(3000, function (req, res) {
  console.log('app is running at port 3000');
});
//node app.js
//访问 http://localhost:3000/?q=alsotang，完成。
```


> 如果直接访问 http://localhost:3000/ 会抛错
> 可以看到，这个错误是从 crypto.js 中抛出的。
> 这是因为，当我们不传入 q 参数时，req.query.q 取到的值是 undefined，utility.md5 直接使用了这个空值，导致下层的 crypto 抛错。

#### 4.13 使用 superagent 与 cheerio 完成简单爬虫
当在浏览器中访问 http://localhost:3000/ 时，输出 CNode(https://cnodejs.org/ ) 社区首页的所有帖子标题和链接，以 json 的形式。
输出示例：
```php
[
  {
    "title": "【公告】发招聘帖的同学留意一下这里",
    "href": "http://cnodejs.org/topic/541ed2d05e28155f24676a12"
  },
  {
    "title": "发布一款 Sublime Text 下的 JavaScript 语法高亮插件",
    "href": "http://cnodejs.org/topic/54207e2efffeb6de3d61f68f"
  }
]
```
> 我们这回需要用到三个依赖，分别是 express，superagent 和 cheerio。
> superagent(http://visionmedia.github.io/superagent/ ) 是个 http 方面的库，可以发起 get 或 post 请求。
> cheerio(https://github.com/cheeriojs/cheerio ) 大家可以理解成一个 Node.js 版的 jquery，用来从网页中以 css selector 取数据，使用方式跟 jquery 一样一样的。

> 1.npm init
> 2.npm install --save express superagent cheerio
> 3.写应用逻辑

- 编写代码
```php
var express = require('express');
var superagent = require('superagent');
var cheerio = require('cheerio');

var app = new express();
app.get('/', function (req, res, next) {
  // 用 superagent 去抓取 https://cnodejs.org/ 的内容
  superagent.get('https://cnodejs.org/')
    .end(function (err, sres) {
      // 常规的错误处理
      if (err) {
        return next(err);
      }
      // sres.text 里面存储着网页的 html 内容，将它传给 cheerio.load 之后
      // 就可以得到一个实现了 jquery 接口的变量，我们习惯性地将它命名为 `$`
      // 剩下就都是 jquery 的内容了
      var $ = cheerio.load(sres.text);
      var items = [];
      $('#topic_list .topic_title').each(function (idx, element) {
        var $element = $(element);
        items.push({
          title: $element.attr('title'),
          href: $element.attr('href')
        });
      });

      res.send(items);
    });
});

app.listen(3000, function(){
        console.log('app is listen at port 3000');
});
```


### 5. package.json理解
#### 5.1 dependencies解释

```php
//npm完整版本号
【主版本 . 次要版本 . 补丁版本】
一般情况下，次要版本号发生改变的话，表示程序有重大更新。

//各符号意义:
^: 安装指定版本号以上的版本(包括自己),允许最左边不为0的版本号升级
~: 安装指定版本号以上的版本(包括自己),如果指定了次要版本，允许补丁版本升级。如果没有指定次要版本，允许次要版本升级。
```

| 标识示例 | 版本范围 | 说明 |
| :-----: | :----: | :-----: |
| ~1.2.3 | 1.2.3 <= version < 1.3.0 |  |
| ~1.2 | 1.2.0 <= version < 1.3.0 |  |
| ~1 | 1.0.0 <= version < 2.0.0	|  |
| ~0.2.3 | 0.2.3 <= version < 0.3.0 |  |
| ~0.2 | 0.2.0 <= version < 0.3.0 |  |
| ~0 | 0.0.0 <= version < 1.0.0	|  |
| ~1.2.3-beta.2 | 1.2.3-beta.2 <= version < 1.3.0 | 1.2.3版允许高于beta.2的beta版，但1.2.4-beta.2不被允许，因为是属于另一个版本号组的beta版本。 |
| --- | --- | ---  |
| ^1.2.3 | 1.2.3 <= version < 2.0.0 |  |
| ^0.2.3 | 0.2.3 <= version < 0.3.0 |  |
| ^0.0.3 | 0.0.3 <= version < 0.0.4 |  |
| ^1.2.3-beta.2 | 1.2.3-beta.2 <= version < 2.0.0 | 允许1.2.3 版的高于beta-2 的beta版本 |
| ^0.0.3-beta.2 | 0.0.3-beta.2 <= version < 0.0.4 | 只允许0.0.3 版的高于beta-2 的版本 |
| ^0.0 | 0.0.0 <= version < 0.1.0 |  |
| ^0 | 0.0.0 <= version < 1.0.0 | 无 |

#### 5.2 devDependencies与dependencies的区别
- devDependencies用于本地环境开发时候。
- dependencies用户发布环境

> - 通过NODE_ENV=developement或NODE_ENV=production指定开发还是生产环境。 
> - devDependencies是只会在开发环境下依赖的模块，生产环境不会被打入包内,而dependencies依赖的包不仅开发环境能使用，生产环境也能使用。
> - 通过--save或--save-dev添加到package.json相应的位置

### 6. JavaScript和Nodejs的区别
> JavaScript=ECMAScript(语言基础，如：语法、数据类型结构以及一些内置对象) + DOM(一些操作页面元素的方法) + BOM(一些操作浏览器的方法)
> Nodejs=ECMAScript(语言基础，如：语法、数据类型结构以及一些内置对象) + os(操作系统) + file(文件系统) + net(网络系统) + database(数据库)























