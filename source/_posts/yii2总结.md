---
title: yii2总结
date: 2018-07-05 15:56:58
tags:
---
## 1.应用的静态结构图和目录结构
<img src="http://resource.cmdapps.com/2018/07/static_struct.png"/>
<center>静态结构图</center>
<img src="http://resource.cmdapps.com/2018/07/contents.png"/>

<center>目录结构</center>

## 2.基础知识点
### 2.1 请求
```php
$request = Yii::$app->request;
$id = $request->get('id', 1); //相当于$id = isset($_GET['id']) ? $_GET['id'] : 1;
$method = Yii::$app->request->method;
$headers = Yii::$app->request->headers; //获取头部
$userHost = Yii::$app->request->userHost;
$userIP = Yii::$app->request->userIP;
```
### 2.2控制器示例
```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public function actionIndex(){
        $test = "hah";
        $this->render("index", ['test' => $test]);
    }
}
```

### 2.3模型示例
```php
namespace app\models;

use yii\base\Model;


class EntryForm extends Model
{
    public $name;
    public $email;

    /**
    *设定规则 
    *safe:不验证,required:必须验证,compare:对比,in:范围验证,
    *double:双精度验证,email:邮箱验证,integer:数字验证,match:正则验证,string:字符串验证,
    *unique:唯一验证,capture:验证码(这两个都需要数据库)
    */
    public function rules() {
        return [
            [['name', 'email'], 'required'],
            ['email', 'email']
        ];
    }

    /**
    * 设置标签
    */
    public function attributeLabels() {
        return [
            'name' => 'Your Name',
            'email' => 'Your Email'
        ];
    }
}
```

### 2.4视图关键点讲解
#### 2.4.1布局
默认情况下,@app/views/layouts/main.php文件中放置公共布局文件,在公共文件中$content
用于调用控制器中render方法渲染内容视图后的结果.
#### 2.4.2视图和控制器变量连结
通过extract方法分配变量.
### 2.5数据库示例

### 2.6其他
```php
跳转:yii\helpers\Url::to(["site/index", "id" => 23, "#" => "content"],false); //index.php/site/index?id=23#content
维护配置:'catchAll' => ['site/offline'],
```

## 3.架构图