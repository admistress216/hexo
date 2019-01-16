---
title: composer总结
date: 2018-10-12 15:59:04
tags: 
categories:
- php 
---

### 1. 下载与配置
#### 1.1 下载
```php
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
//或者
wget https://getcomposer.org/composer.phar
mv composer.phar /usr/local/bin/composer
```
#### 1.2 中国镜像全局配置
```php
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

#### 1.3 使用composer完成自动加载器
```php
//1.首选新建一个PHP项目文件夹,可以手动写一个composer.json，内容如下：
{
        "autoload": {
                "files": [
                        "comm/functions.php"
                ]
        }
}

//2.从上面json信息，我们可以大致猜测，这是要做文件的自动加载。同时，我们新建好comm目录和functions.php文件。完成上面操作，我们打开终端，cd到 test目录下面，执行命令：
composer dump-autoload

//3.然后在看我们的项目，多出来一个vendor目录，里边就是composer的东西,至此，我们应该来测试一下，composer到底怎么做自动加载的？我们在comm目录下的functions.php写了一个函数：
function showName() {
        echo "我的名字";
}

//4.然后我们要在index.php中，调用这个函数。常规的方法是先要require 'comm/functions.php'， 然后才能调用funcitons.php中定义的函数。下面我们看composer的方式：
require __DIR__.'/vendor/autoload.php';
showName();
# php index.php

//5.我们继续在comm目录下，新建一个test.php文件：
function test(){
        echo 'test';
}

//6.这个时候，我们要想在index.php中能调用test()函数，需要：在composer.json中增加：
{
        "autoload": {
                "files": [
                        "comm/functions.php",
                        "comm/test.php"
                ]
        }
}
composer dump-autoload //终端，同样还是在项目目录下执行,完成上面2步，我们就可以在index.php中，调用test()函数了

//7.类的自动加载:我们新建一个Class目录，里面新建一个User.php
class User{

}
//然后修改composer.json文件：
{
        "autoload": {
                "files": [
                        "comm/functions.php",
                        "comm/test.php"
                ],
                "classmap": [
                        "Class/"
                ]
        }
}
//完成上面操作，同样是需要在终端下执行：
composer dump-autoload
//最后，我们在index.php中测试：
$user = new User();
var_dump($user);
```

### 2. 要点
#### 2.1 ~和^的区别

```php
 ~和^的意思很接近，在x.y的情况下是一样的都是代表x.y <= 版本号 < (x+1).0，但是在版本号是x.y.z的情况下有区别，举个例子吧：

~1.2.3 代表 1.2.3 <= 版本号 < 1.3.0

^1.2.3 代表 1.2.3 <= 版本号 < 2.0.0
```

