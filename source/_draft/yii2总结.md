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

### 2.2 响应
```php
$headers = Yii::$app->response->headers;
$headers->add('Pragma', 'no-cache'); //添加响应头字段
Yii::$app->response->redirect('http://www.baidu.com', 301)->send(); //跳转


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
### 2.5数据库
数据库可连接mysql,mariadb,oracle,sqlight,sqlserver等
```php
//1.数据库配置
'db' => [
    'class' => 'yii\db\Connection',
    'driverName' => 'mysql',
    'dsn' => 'odbc:Driver={MySQL};Server=localhost;Database=test',
    'username' => 'root',
    'password' => '',
],

//2.在controller里创建数据库对象
$db = new yii\db\Connection([
    'dsn' => 'mysql:host=localhost;dbname=example',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8',
]);
/**
 * 3.1纯sql查询,利用yii\db\Command
 *
 * 查询方法:
 * queryAll():查询多行
 * queryOne():查询单行
 * queryColumn():查询单列
 * queryScalar():查询标量,如count(1)等
 *
 */
 //bindValue:
$posts = Yii::$app->db->createCommand('SELECT * FROM post WHERE `status`=:status AND `type`=:type')
        ->bindValue(':status', $_GET['status'])
        ->bindValue(':type', 1)
        ->queryAll();
//bindValues:
$params = [
    ':status' => $_GET['status'],
    ':type' => 1
];
$posts = Yii::$app->db->createCommand('SELECT * FROM post WHERE `status`=:status AND `type`=:type')
        ->bindValues($params)
        ->queryAll();

/**
 * 3.2纯sql非查询
 *
 * 执行方法:
 * execute():
 *
 */
 // sql
Yii::$app->db->createCommand('UPDATE post SET status=1 WHERE id=1')
    ->execute();
    
// INSERT (table name, column values),
//columns inserting use batchInsert
Yii::$app->db->createCommand()->insert('user', [
    'name' => 'Sam',
    'age' => 30,
])->execute();

// UPDATE (table name, column values, condition)
Yii::$app->db->createCommand()->update('user', ['status' => 1], 'age > 30')->execute();

// DELETE (table name, condition)
Yii::$app->db->createCommand()->delete('user', 'status = 0')->execute();

//4.读写分离
[
    'class' => 'yii\db\Connection',

    // 主库的配置
    'dsn' => 'dsn for master server',
    'username' => 'master',
    'password' => '',

    // 从库的通用配置
    'slaveConfig' => [
        'username' => 'slave',
        'password' => '',
        'attributes' => [
            // 使用一个更小的连接超时
            PDO::ATTR_TIMEOUT => 10,
        ],
    ],

    // 从库的配置列表
    'slaves' => [
        ['dsn' => 'dsn for slave server 1'],
        ['dsn' => 'dsn for slave server 2'],
        ['dsn' => 'dsn for slave server 3'],
        ['dsn' => 'dsn for slave server 4'],
    ],
]

//5.查询构建器
$rows = (new \yii\db\Query())
    ->select(['id', 'email'])
    ->from('user')
    ->where(['last_name' => 'Smith'])
    ->limit(10)
    ->all();
解释成:
SELECT `id`, `email` 
FROM `user`
WHERE `last_name` = :last_name
LIMIT 10

```
### 2.6其他
```php
链接:yii\helpers\Url::to(["site/index", "id" => 23, "#" => "content"],false); //index.php/site/index?id=23#content
维护配置:'catchAll' => ['site/offline'],
mysql表前缀引用{{%tablename}}
```

## 3.架构图