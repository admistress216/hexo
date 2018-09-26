---
title: laravel总结
date: 2018-09-26 14:19:52
tags:
---

### 1.目录结构

### 2.路由
```php
Route::get/post/put/patch/delete/options($uri, $callback);
Route::match(['get', 'post'], '/test', $callback);
Route::any('/test', $callback);
//参数
Route::get('user/{id}', function($id){
    return "User ".$id;
});
//可选参数(默认值)
Route::get('user/{id?}', function($id=null){
    return "User ".$id;
});
//正则筛选
Route::get('user/{id?}', function($id=null){
    return "User ".$id;
})->where('id', '[0-9]+'); //id为int型

//控制器
Route::get('user/{id}', 'Admin\IndexController@index');
//命令行创建控制器
php artisan make:controller Admin/IndexController

//路由命名
Route::get('user', ['as' => 'profile',  function(){
    echo route('profile'); //http://192.168.200.128/user(地址)
    return "路由命名";
}]);
//在控制器中也可使用route(),view()函数
Route::get('user', ['as' => 'profile',  'uses' => 'Admin\IndexController@index']);
Route::get('user', 'Admin\IndexController@index')->name('profile');

//路由分组，访问http://192.168.200.128/admin/login会路由到Admin\IndexController@login方法
Route::group(['prefix' => 'admin', 'namespace' => 'Admin'], function(){
    Route::get('login', 'IndexController@login');
    Route::get('index', 'IndexController@index');
});

//资源路由,自动生成article/create(index/delete)等方法
Route::resource('article', 'Admin\ArticleController');
```

### 3.中间件(路由前过滤作用)
```php
//命令生成中间件,在app/Http/Middleware中生成AdminLogin.php中间件文件
php artisan make:middleware AdminLogin
//App\Http\Kernel中追加:
protected $routeMiddleware = [
        'admin.login' => \App\Http\Middleware\AdminLogin::class,
];
//路由中使用:
Route::group(['middleware' => ['admin.login']], function(){
    Route::get('/', function () {
        session(['key' => 123]);
        return view('welcome');
    });
});
```

### 4.视图
```php
//1.分配视图并传参(一般传递数组)
return view('index')->with('name', $name)->with('age', $age);
$data = [
    'name' => 'haha',
    'age' => 5
];
return view('index', $data);
//多参数:
$title = "thie is a title";
return view('index', compact('data','title'));
//参数使用(模板中)
<?php echo $data['name'].'----'.$title;  ?>
//使用blade模板引擎(用@屏蔽),默认值(传值为null时)用or创建
{{$data['name']}}---{{title or 'XXX'}}
//不转实体
{!! $script !!}
//判断变量是否存在
{{isset($name) ? $name : 'default'}}

//blade模板引擎 if(unless),foreach,for,forelse,@include
@if()
....
@else
....
@endif
@for()
....
@endfor
@foreach()
.....
@endforeach
@forelse()
...
@empty
....
@endforelse
@include('common.header', ['page' => '首页']) //引入公共部分,第二个参数可没有
@yield('content') //公共模板中引入content部分
    //子视图中
    @extends('layouts.home') //引入公共文件
    @section('content')
    @endsection
```

### 5.配置读取及数据库
```php
config('database.connections.mysql.prefix')

$pdo = DB::connection()->getPdo();
dd($pdo); //类似于print_r()

$users = DB::table('user')->where('user_id', '>', 1)->get();
dd($users);
```

### 6.模型
//新建
php artisan make:model User
//模型中配置表和主键
protected $table = "user";
protected $primaryKey = 'user_id';
//控制器中使用模型
$user = User::find(1);
$user->user_name = 'wangwu';
$user->update(); //更新
//禁用updated_at字段
public $timestamps = false;
