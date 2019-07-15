---
title: vue深入学习
date: 2018-11-15 16:34:47
tags:
---

### 1. cnpm包安装与vue环境搭建

```php
cnpm  下载包的速度更快一些。
地址：http://npm.taobao.org/	
安装cnpm:
npm install -g cnpm --registry=https://registry.npm.taobao.org

搭建vue的开发环境：
https://cn.vuejs.org/v2/guide/installation.html
1、必须要安装nodejs
2、搭建vue的开发环境 ，安装vue的脚手架工具   官方命令行工具
npm install --global vue-cli  /   cnpm install --global vue-cli         （此命令只需要执行一次）
3、创建项目   必须cd到对应的一个项目里面
方式一:
vue init webpack vue-demo01
cd  vue-demo01 
cnpm install   /  npm install          如果创建项目的时候没有报错，这一步可以省略。如果报错了  cd到项目里面运行  cnpm install   /  npm install
npm run dev

方式二:
vue init webpack-simple vuedemo02
cd  vuedemo02
cnpm install   /  npm install
npm run dev
```

### 2. vue问题总结
### 2.1 vue axios数据(data)赋值问题
```php
1.问题描述:
使用axios获取接口数据,然后将返回后的数据赋值给data()中定义的属性,执行后前端报错,undefined

2.原因:
在请求执行成功后执行回调函数中的内容，回调函数处于其它函数的内部this不会与任何对象绑定，为undefined

3.解决方案:
一) 将指向vue对象的this赋值给外部方法定义的属性,然后在内部方法中使用该属性
    mounted: function(){
        var _this = this;
        _this.$axios.get('api_url').then(function(response){
            _this.prop = response.data;
        })
    }

二) 使用箭头函数
mounted: function(){
    this.$axios.get('api_url').then(response => {
        this.prop = response.data;
    })
}
```

### 2.1 vue axios数据(data)赋值问题
```php
1.问题描述:
点击子组件函数不生效

2.原因:
需要加原始修饰符

3.解决方案:
<product-card :product='product' @click.native="showProduct(product.id)" />

showProduct(id){
    this.$router.push('/product/'+id);
},
```














