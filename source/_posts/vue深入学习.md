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

### 2. vue的使用
```php
//vue的模板里面,所有的内容要被一个根节点包含起来
<template>
  <div id="app"> //根节点
        <h2>你好vue</h2>
        <br>这是一个跟组件
  </div>
</template>

//绑定属性 v-bind:title或者:title
<div v-bind:title="title">鼠标看一下</div>
<div v-bind:class="{'red': flag,'blue': !flag}">我是一个div</div> //flag设为true

//绑定事件 v-on:click或者@click
<button v-on:click="run1()">执行方法的第一种写法</button>
<button @click="run2()">执行方法的第二种写法</button>
//传值:
<button @click="run3($e)">传值试验</button>
//$e解析
e.srcElement: 获取dom对象
e.srcElement.style.background = 'red';
e.srcElement.dataset: 获取自定义对象
    <button data-aid="123" @click="eventFn($event)">事件对象</button>
    console.log(e.srcElement.dataset.aid)

//绑定html v-html
<div v-html="h"></div>  //h: "<h2>我是h2</h2>"

//绑定数据 {{msg}}或者v-text

//v-for使用:
<ul>
    <li v-for="(item,index) in list" :class="{'red':key==0}">
        {{key}}------{{item}}
    </li>
</ul>

//获取dom节点
<input type="text" ref="userinfo">
console.log(this.$refs.userinfo);
alert(this.$refs.userinfo.value);
```

### 3. v-bind的使用
```php
<!-- 绑定一个属性 -->
<img v-bind:src="imageSrc">

<!-- 缩写 -->
<img :src="imageSrc">

<!-- 内联字符串拼接 -->
<img :src="'/path/to/images/' + fileName">

<!-- class 绑定 -->
<div :class="{ red: isRed }"></div>
<div :class="[classA, classB]"></div>
<div :class="[classA, { classB: isB, classC: isC }]">

<!-- style 绑定 -->
<div :style="{ fontSize: size + 'px' }"></div>
<div :style="[styleObjectA, styleObjectB]"></div>

<!-- 绑定一个有属性的对象 -->
<div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>

<!-- 通过 prop 修饰符绑定 DOM 属性 -->
<div v-bind:text-content.prop="text"></div>

<!-- prop 绑定。“prop”必须在 my-component 中声明。-->
<my-component :prop="someThing"></my-component>

<!-- 通过 $props 将父组件的 props 一起传给子组件 -->
<child-component v-bind="$props"></child-component>

<!-- XLink -->
<svg><a :xlink:special="foo"></a></svg>
```

### 4. 双向数据绑定(MVVM)
> M: model
> V: view
> MVVM: model改变会影响视图view,view视图改变反过来会影响model
> 注: 双向数据绑定必须在表单中使用

```php
<h2>msg</h2>
<input type="text" v-model="msg"/>
<button v-on:click="getMsg()">获取表单里面的数据</button>

<script>
    export default {
        data() {
            return {
                msg: '你好vue'
            }
        },methods:{
            getMsg(){
                alert(this.msg)
            }
        }
    }
</script>
```

### 5. $event相关
```php
e.srcElement.dataset.aid: 自定义数据
e.keyCode: 按键码(enter:13)
```
