---
title: js总结
date: 2018-08-28 13:53:51
tags:
---

### 1. 页面加载完加载js
```php
1、$(function(){ 
　　$("#a").click(function(){ 
　　　　//adding your code here 
　　}); 
}); 
//同(function(){})(jQuery)
2、$(document).ready(function(){ 
　　$("#a").click(function(){ 
　　　　//adding your code here　　 
　　}); 
}); 
3、window.onload = function(){ 
　　$("#a").click(function(){ 
　　　　//adding your code here 
　　}); 
} 
```

### 2.函数
```php
trigger('select'): 出发select事件
```