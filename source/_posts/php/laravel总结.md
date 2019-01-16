---
title: laravel总结
date: 2018-09-26 14:19:52
tags:
categories:
- php 
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

- 为路由配置多个中间件
```php
Route::get('/', function () {
    //
})->middleware('first', 'second');
```
- 分配中间件时，你还可以传递完整的类名：
```php
use App\Http\Middleware\CheckAge;

Route::get('admin/profile', function () {
    //
})->middleware(CheckAge::class);
```
- 中间件组: 中间件组只是更方便地实现了一次为路由分配多个中间件
```php
Route::get('/', function () {
    //
})->middleware('web');

Route::group(['middleware' => ['web']], function () {
    //
});
```
> 注:web中间件会自动应用于routes/web.php

#### 3.5 中间件参数
- 通过$next之后传递参数
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
    public function handle($request, Closure $next, $role)
    {
        if ($request->age <= 200) {
            echo '年轻'."<br>";
            echo $role;
        }

        return $next($request);
    }

}
```
- 定义路由时通过一个 : 来隔开中间件名称和参数来指定中间件参数。多个参数就使用逗号分隔：
```php
Route::put('/index', function ($id) {
    //
})->middleware('age.check:editor');
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

### 6.用户认证系统
#### 6.1 数据导入与创建路由,视图和HomeController
```php
php artisan migrate //数据导入
php artisan make:auth //生成身份认证的路由,视图(resources/views/auth下)和HomeController
```

#### 6.2 自定义跳转路径
- 你可以通过在 LoginController、RegisterController 和 ResetPasswordController中设置 redirectTo 属性来自定义重定向的位置：
```php
protected $redirectTo = '/';

//自定义逻辑
protected function redirectTo()
{
    return '/path';
}
```
> 注: 方法优先于属性

#### 6.3 自定义认证用户名(LoginController)
```php
    public function username() {  //默认为email
        return 'email';
    }
```

#### 6.4 自定义看守器(暂无)

#### 6.5 检索已认证用户信息
- 方式一:
```php
use Illuminate\Support\Facades\Auth;

$user = Auth::user();  //获取当前已认证用户信息
$id = Auth::id();  //获取认证id
```
- 方式二:
```php
use Illuminate\Http\Request;

public function index(Request $request) {
    $user = $request->user();  //只有获取user信息的,没有获取id方法
}
```

#### 6.6 确认用户是否认证(认证返回true)
```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // 用户已登录...
}
```

#### 6.7 别名
- 别名的定义
```php
//位置: /config/app.php
'aliases' => [
        'Auth' => Illuminate\Support\Facades\Auth::class,
    ],
```
- 别名的使用
```php
//控制器中
use Auth;  //不用写全命名空间
```

#### 6.8 路由保护
> 路由中间件 用于只允许通过认证的用户访问指定的路由
> Laravel 自带了在 Illuminate\Auth\Middleware\Authenticate 中定义的 auth 中间件。
> 由于这个中间件已经在 HTTP 内核中注册(App\Http\Kernel中路由中间件)，所以只需要将中间件附加到路由定义中

- 路由中添加:
```php
Route::get('profile', function () {
    // 只有认证过的用户可以...
})->middleware('auth');
```
- 控制器中添加:
```php
public function __construct()
{
    $this->middleware('auth');
}
```
> 将 auth 中间件添加到路由时，还需要指定使用哪个看守器来认证用户。
> 指定的看守器对应配置文件 auth.php 中 guards 数组的某个键：

- 指定看守器:
```php
public function __construct()
{
    $this->middleware('auth:api');
}
```
- 频路限制
```php
Route::get('/api/users', ['middleware' => 'throttle:60,1'], function(){  //限制某个 IP 地址每分钟只能访问某个路由 60 次
    //
});
```
#### 6.9手动认证用户等相关操作
- 认证用户:
> 不采用laravel内置的认证控制器,直接调用laravel的认证类来认证(Illuminate\Support\Facades\Auth)
> 代码中的email字段为用户名,可为其他字段,attempt()方法,接收数组,认证成功则返回true,失败则返回false
> redirect('home')和redirect()->intended('home')作用一样(都有return)

```php
namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * 处理身份认证尝试.
     *
     * @return Response
     */
    public function authenticate()
    {
        if (Auth::attempt(['email' => $email, 'password' => $password])) {
            // 认证通过...
            return redirect()->intended('dashboard');
        }
    }
}
```

- 注销用户
```php
Auth::logout();
```

- 访问指定的看守器实例
> 可以通过 Auth facade 的 guard 方法来指定要使用哪个看守器实例。这允许你使用完全独立的可认证模型或用户表来管理应用的抽离出来的身份验证。
> 传递给 guard 方法的看守器名称应该与 auth.php 配置文件中 guards 中的其中一个值相对应：

```php
if (Auth::guard('api')->attempt($credentials)){

}
```
- 看守器详解:
```
'guards' => [
    'api' => [
        'driver' => 'jwt',  //看守器驱动,可自定义(session,token,jwt)
        'provider' => 'users',  //所使用的提供器名称
    ],
],
```

- 提供器详解:
```
'providers' => [
        'users' => [                       //提供器名称
            'driver' => 'eloquent',       //驱动
            'model' => App\User::class,  //使用的模型或表名
        ],

        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],
```

- HTTP基础认证
暂时不用



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
//接收数据
/$input = Input::all();

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

protected $table = 'my_users'; //模型中定义表(默认表为模型小写复数)
protected $primaryKey = 'user_id'; //定义主键(默认主键为id)
protected $connection = 'mysql2'; // 在config/database.php中设定
public $timestamps = false;  // 数据表中不设置updated_at和created_at字段
//可以被批量赋值的属性,只有批量赋值的属性才可通过create()添加数据
protected $fillable = ['first_name', 'last_name', 'email'];  //批量赋值白名单
$flight = Flight::create(['first_name' => 'Flight 10']);  //允许first_name赋值
protected $guarded = ['id', 'password'];  //批量赋值黑名单,*为阻止所有
public $incrementing = false;  //非递增主键
protected $keyType = string;  //主键为字符串而非int型
const CREATED_AT = 'creation_data';  //自定义时间戳
const UPDATED_AT = 'last_update';  //自定义时间戳

/**
 *查询相关
 */
$users = User::all();  //取出所有数据
$user = User::find(1);  //去除一条 $user->name;
$user = User::where('active', 1)->first(); //first相当于find
$user = User::find([1,2,3]);
$user = User:findOrFail(1); //是否存在id为1的记录,可定义异常
$users = User::where('id', '>', 10)->firstOrFail(); //是否存在一条,存在则取出,可定义异常
$users = User::where('id', '>', 1)->take(10)->get(); //结合条件查询,id大于1的记录取出10条,where和get()搭配使用
$count = User::where('id', '>', 1)->count();  //统计条数
$users = User::whereRaw('age > ? and votes = 100', [25])->get();
$user = User::on('connection-name')->find(1); //指定数据库
$user = User::where('active', 1)
              ->orderBy('name', 'desc')
              ->take(10)  //take相当于limit,skip相当于offset
              ->get();
              
/**
 *聚合函数
 */
 $count = User::where('active', 1)->count();
 $price = User::where('active', 1)->max('price');

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
//已有模型
$flight->fill(['name' => 'Flight 22']);

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
