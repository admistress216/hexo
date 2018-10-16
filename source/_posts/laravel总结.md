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

### 3.中间件(请求之前/之后对数据进行过滤)
#### 3.1 中间件位置及创建
```php
app/Http/Middleware  //位置
php artisan make:middleware CheckAge //生成app/Http/Middleware/CheckAge.php中间件
```
#### 3.2 定义中间件规则
仅允许提供的参数 age 大于 200 的请求访问该路由。否则，我们会将用户重定向到 home 。
```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckAge
{
    /**
     * 处理传入的请求
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }

        return $next($request);
    }

}
```

#### 3.3 注册中间件
```php
//App\Http\Kernel中追加:

protected $middleware = [
        \App\Http\Middleware\CheckAge::class, //注册全局中间件
    ];

protected $routeMiddleware = [  //注册路由中间件
        'age.check' => \App\Http\Middleware\CheckAge::class,
];
```

#### 3.4 路由中使用
```php
Route::group(['middleware' => ['age.check']], function(){
    Route::get('/', function () {
        session(['key' => 123]);
        return view('welcome');
    });
});
```

为路由配置多个中间件
```php
Route::get('/', function () {
    //
})->middleware('first', 'second');
```
分配中间件时，你还可以传递完整的类名：
```php
use App\Http\Middleware\CheckAge;

Route::get('admin/profile', function () {
    //
})->middleware(CheckAge::class);
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
```php
//接收数据
/$input = Input::all();

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
//非递增主键
public $incrementing = false;
//主键为字符串而非int型
protected $keyType = string;
//禁用updated_at,created_at字段
public $timestamps = false;
//自定义时间戳
const CREATED_AT = 'creation_data';
const UPDATED_AT = 'last_update';

//可以被批量赋值的属性,只有批量赋值的属性才可通过create()添加数据
protected $fillable = ['name']; //允许name属性可以批量赋值
//不可以批量赋值的属性
protected $guarded = ['price'];
$flight = Flight::create(['name' => 'Flight 10']);
//已有模型
$flight->fill(['name' => 'Flight 22']);

$user = User::all(); //返回模型表中所有的结果

$user = User::where('active', 1)
              ->orderBy('name', 'desc')
              ->take(10)  //take相当于limit,skip相当于offset
              ->get();

$user = User::where('active', 1)->first(); //first相当于find
$user = User::find([1,2,3]);

//聚合函数
$count = User::where('active', 1)->count();
$price = User::where('active', 1)->max('price');

//插入
$user = new User;
$user->name = $name;
$user->save();

//更新
$flight = Flight::find(1);
$flight->name = 'New';
$flight->save();
Flight::where('active', 1)
   ->where('destination', 'San Diego')
   ->update(['delay' => 1]); //需要传入键值对数组

//删除
$flight = Flight::find(1);
$flight->delete();
Flight::destroy(1,2,3);
Flight::destroy([1,2,3]);
$deletedRows = Flight::where('active', 0)->delete();
```


### 7.清除缓存
#### 7.1 清除视图缓存
php artisan view:clear
#### 7.2 清除运行缓存
php artisan cache:clear
#### 7.3 清除配置缓存
php artisan config:clear

### 8.方法
//引入静态文件
asset()
//引入地址
url()
//返回到前一个页面
back()
//参数传递
with()
//加解密
Crypt::encrypt($str)
Crypt::decrypt($str)
//重定向
redirect($url);

### 9.request

```php
/**
*基于Illuminate\Http\Request
*/
//获取所有参数并放于数组中
$request->all();
//获取相应字段,get,post等均可
$request->input('name', 'lizengcai'); //第二个参数用于默认值
$request->input('products.*.name');  //获取数组的值
$request->has('name');  //数据存在返回true
$request->only(['name', 'post']);
$request->except('name');  //接收部分参数,参数可以为数组或字符串

//路由参数接收
	$api->post('test/{id}', [ //路由
                'as' => 'test.index',
                'uses' => 'TestController@index',
        ]);
	public function test(Request $request, $id){}	//controller方法

//以http://laravel_api.com/api/test/3?b=c为例
//返回请求的uri
$uri = $request->path();  //  /api/test/3
//对uri进行匹配
$request->is('api/*');   //  true

//完整网址
$url = $request->url();   //  http://laravel_api.com/api/test/3
$fullUrl = $request->fullUrl();   //http://laravel_api.com/api/test/3?b=c

//方法
$method = $request->method();    //post
$request->isMethod('post');    //true


```

### 10.Eloquent orm
```php
php artisan make:model User; //新建模型

protected $table = 'my_users'; //模型中定义表(默认为模型小写复数)
protected $primaryKey = 'user_id'; //定义主键(默认为id)
protected $connection = 'mysql2'; // 在config/database.php中设定
public $timestamps = false;  // 数据表中不设置updated_at和created_at字段
protected $fillable = ['first_name', 'last_name', 'email'];  //批量赋值白名单
protected $guarded = ['id', 'password'];  //批量赋值黑名单,*为阻止所有

/**
 *查询相关
 */
$users = User::all();  //取出所有数据
$user = User::find(1);  //去除一条 $user->name;
$user = User:findOrFail(1); //是否存在id为1的记录,可定义异常
$users = User::where('id', '>', 10)->firstOrFail(); //是否存在一条,存在则取出,可定义异常
$users = User::where('id', '>', 1)->take(10)->get(); //结合条件查询,id大于1的记录取出10条,where和get()搭配使用
$count = User::where('id', '>', 1)->count();  //统计条数
$users = User::whereRaw('age > ? and votes = 100', [25])->get();
$user = User::on('connection-name')->find(1); //指定数据库

/**
 * 新增
 */
$user = new User;
$user->name = 'John';
$user->save(); //新增或存储(create)
$insertedId = $user->id; //新增或存储的id
$user = User::create(['name' => 'John']);  //在数据库中建立一个新用户
$user = User::firstOrCreate(['name' => 'John']);  // 以属性找用户，若没有则新增并取得新的实例...
$user = User::firstOrNew(['name' => 'John']);  // 以属性找用户，若没有则建立新的实例...

/**
 * 更新,删除
 */
$user = User::find(1);
$user->email = 'john@foo.com';
$user->save();  //更新取出的模型
$affectedRows = User::where('votes', '>', 100)->update(['status' => 2]); //您可以结合查询语句，批次更新模型：

/**
 * 删除
 */
 $user = User::find(1);
 $user->delete();
 
 User::destroy(1);
 User::destroy([1, 2, 3]);
 User::destroy(1, 2, 3); //根据主键值删除
 
 $affectedRows = User::where('votes', '>', 100)->delete(); //批次删除
```