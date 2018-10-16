---
title: jwt总结
date: 2018-10-15 16:46:53
tags:
---

### 1. 小问题

#### 1.1 中间件解释
```php
Route::group(['middleware' => 'auth:api'], function () {
  Route::get('/t', function () {
      return 'ok';
  });
});
```
在此处,使用的是auth中间件,:api的意思是说,我们指定了auth中间件的driver为api,而如果不指定的话,
driver默认使用的是web

### 2. jwt使用例子
#### 2.1 创建用户
```php
//Controller
public function register(Request $request) {
    $newUser = [
        'email' => $request->input('email'),
        'name' => $request->input('name'),
        'password' => bcrypt($request->input('password'))
    ];
    
    $user = User::create($newUser);
    $token = JWTAuth::fromUser($user);
    
    return responseSuccess($token);
}
```

#### 2.2 用户登录
```php
//Controller
public function authenticate(UserLoginRequest $request)
{
    $credentials = $request->only('email', 'password');

    try {
        if (! $token = JWTAuth::attempt($credentials)) {
            return responseError(-1, "邮箱或者密码错误");
        }
    } catch (JWTException $e) {
        return responseError(-3, "token 无法生成");
    }

    return responseSuccess(compact('token'));
}
```

#### 2.3 获取用户
```php
$user = JWTAuth::parseToken()->authenticate();
```

#### 2.4 用户登出
```php
public function logout()
    {
        JWTAuth::setToken(JWTAuth::getToken())->invalidate();

        return responseSuccess();
    }

```