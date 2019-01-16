---
title: mysql总结
date: 2018-10-05 00:34:56
tags:
category: [cache]
---
### 1. 更改密码及导出
```php
'mysqladmin -uroot -proot password + 新密码', //更改密码
'mysql -uroot -p +数据库 < +sql路径',//导入sql文件
'mysqldump -uroot -p [-d] +数据库 > sql文件', //导出sql文件,-d:no data(仅结构)
```

### 2. linux下mysql安装
```php
groupadd mysql
useradd -g mysql -s /sbin/nologin -M mysql, //-s:定义shell,-M : 不建立根目录,-g:指定组
mkdir -p /data/mysql/data,
chown -R mysql:mysql /data/mysql,

yum -y install gcc gcc-c++ cmake ncurses-devel bison,
wget --no-check-certificate https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.39.tar.gz, //下载

# cmake ./

# -DCMAKE_INSTALL_PREFIX=/usr/local/mysql          \    #安装路径

# -DMYSQL_DATADIR=/usr/local/mysql/data            \    #数据文件存放位置

# -DSYSCONFDIR=/etc                                \    #my.cnf路径

# -DWITH_MYISAM_STORAGE_ENGINE=1                   \    #支持MyIASM引擎

# -DWITH_INNOBASE_STORAGE_ENGINE=1                 \    #支持InnoDB引擎

# -DWITH_MEMORY_STORAGE_ENGINE=1                   \    #支持Memory引擎

# -DWITH_READLINE=1                                \    #快捷键功能(我没用过)

# -DMYSQL_UNIX_ADDR=/tmp/mysqld.sock               \    #连接数据库socket路径

# -DMYSQL_TCP_PORT=3306                            \    #端口

# -DENABLED_LOCAL_INFILE=1                         \    #允许从本地导入数据

# -DWITH_PARTITION_STORAGE_ENGINE=1                \    #安装支持数据库分区

# -DMYSQL_USER=mysql                                \   #mysqld运行用户

# -DEXTRA_CHARSETS=all                             \    #安装所有的字符集

# -DDEFAULT_CHARSET=utf8                           \    #默认字符

# -DDEFAULT_COLLATION=utf8_general_ci
# make && make install

cmake ./ \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql         \
-DMYSQL_DATADIR=/data/mysql/data                \
-DSYSCONFDIR=/etc                                \
-DWITH_MYISAM_STORAGE_ENGINE=1                   \
-DWITH_INNOBASE_STORAGE_ENGINE=1                 \
-DMYSQL_UNIX_ADDR=/data/mysql/mysqld.sock        \
-DMYSQL_TCP_PORT=3306                            \
-DENABLED_LOCAL_INFILE=1                         \
-DWITH_PARTITION_STORAGE_ENGINE=1                \
-DMYSQL_USER=mysql                               \
-DEXTRA_CHARSETS=all                             \
-DDEFAULT_CHARSET=utf8                           \
-DDEFAULT_COLLATION=utf8_general_ci

make&&make install,

chown -R mysql:mysql /usr/local/mysql/,
mv /usr/local/src/mysql-5.6.39/support-files/my-default.cnf /etc/my.cnf //选择覆盖

mv /usr/local/src/mysql-5.6.39/support-files/mysql.server /etc/init.d/mysqld //服务端启动放入服务中service mysqld start
chmod a+x /etc/init.d/mysqld,
chkconfig --level 345 mysqld on, //开机启动

echo "export PATH=/usr/local/mysql/bin/:$PATH" >> /etc/profile
source /etc/profile //设置环境变量,方便客户端启动

//初始化设置(user,配置文件,基础目录,数据目录)
/usr/local/mysql/scripts/mysql_install_db \
--user=mysql \
--defaults-file=/etc/my.cnf \
--basedir=/usr/local/mysql \
--datadir=/data/mysql/data

vim /etc/my.cnf
[mysqld]
user=mysql
basedir=/usr/local/mysql
default-storage-engine=Innodb
datadir=/data/mysql/data //放置于mysql模块
socket=/data/mysql/mysql.sock
[mysql]
socket=/data/mysql/mysql.sock //注意放在最下方

service mysqld reload
service mysqld start

/usr/bin/mysqladmin -u root password \'z\' //修改密码
mysql -u root -p  -P port //登录

说明:
参数说明:
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql //安装目录
-DMYSQL_DATADIR=/usr/local/mysql/data //数据库存放目录
-DWITH_MYISAM_STORAGE_ENGINE=1 //安装myisam存储引擎
-DWITH_INNOBASE_STORAGE_ENGINE=1 //安装innodb存储引擎
-DWITH_ARCHIVE_STORAGE_ENGINE=1 //安装archive存储引擎
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 //安装blackhole存储引擎
-DENABLED_LOCAL_INFILE=1 //允许从本地导入数据
-DDEFAULT_CHARSET=utf8 　　//使用utf8字符
-DDEFAULT_COLLATION=utf8_general_ci //校验字符
-DEXTRA_CHARSETS=all 　　//安装所有扩展字符集
-DMYSQL_TCP_PORT=3306 //MySQL监听端口
-DMYSQL_USER=mysql //MySQL用户名
其他参数:
-DWITH-EMBEDDED_SERVER=1 //编译成embedded MySQL library (libmysqld.a)
-DSYSCONFDIR=/etc //MySQL配辑文件
-DMYSQL_UNIX_ADDR=/tmp/mysqld.sock //Unix socket 文件路径
-DWITH_READLINE=1 //快捷键功能
-DWITH_SSL=yes //SSL
-DWITH_MEMORY_STORAGE_ENGINE=1 //安装memory存储引擎
-DWITH_FEDERATED_STORAGE_ENGINE=1 //安装frderated存储引擎
-DWITH_PARTITION_STORAGE_ENGINE=1 //安装数据库分区
-DINSTALL_PLUGINDIR=/usr/local/mysql/plugin //插件文件及配置路径
```

### 3. 设置权限
```php
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456aA' WITH GRANT OPTION;
flush privileges;
```

### 4.pdo
#### 4.1 pdo类
```php
<?php
/**
 * Created by PhpStorm.
 * User: tc-net
 * Date: 2016/8/11 0011
 * Time: 16:32
 */
class MysqlPDO{
    //获取对象句柄
    protected $message = 'Unknown exception'; // 异常信息
    protected static $_instance = null;
    protected $dbh;
    protected $where = '';
    protected $wheredata = array();
    protected $order = '';
    private function __construct($param) {
        try {
            //echo 'mysql:host=' . $param['hostname'] . ';port=' . $param['port'] . ';dbname=' . $param['database'], $param['username'], $param['password'];exit;
            $this->dbh = new PDO ( 'mysql:host=' . $param['hostname'] . ';port=' . $param['port'] . ';dbname=' . $param['database'], $param['username'], $param['password'], array (PDO::ATTR_PERSISTENT => false,PDO::MYSQL_ATTR_INIT_COMMAND => "set names utf8mb4;") );
//            self::$PDOInstance->query("SET client_encoding='UTF-8';");
//            self::$PDOInstance->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
//            self::$PDOInstance->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch ( PDOException $e ) {
            throw new Exception('MySQL Error: '.$e->getMessage ());
            exit;
        }
    }
    public function clearWhere(){
        $this->where = '';
        $this->wheredata = array();
    }
    private function error($stmt){
        $erorr = $stmt->errorInfo();
        $this->message = "mysql ERROR:".$erorr[1]." ".$erorr[2];
        throw new Exception($this->message);
    }
    /**
     * 防止克隆
     *
     */
    private function __clone() {}

    /**
     * 单例
     *
     */
    public static function getInstance($param,$instance='default'){
        if (!isset(self::$_instance[$instance])||self::$_instance[$instance] === null) {
            self::$_instance[$instance] = new self($param);
        }
        return self::$_instance[$instance];
    }
    public function where($field,$val){
        $this->where[] = preg_match('/[=><]/i',$field)?"`{$field}`":"`{$field}`=";
        if(is_array($val)){
            $this->wheredata = array_merge($this->wheredata,$val);
        }else{
          array_push($this->wheredata,$val);
        }
    }
    public function like_where($field,$val){
        $this->where[] = "`{$field}` like ";
        if(is_array($val)){
            $this->wheredata = array_merge($this->wheredata,$val);
        }else{
            array_push($this->wheredata,$val);
        }
    }
//    public function orderby($field,$sort){
//        $this->order = "ORDER BY `{$field}` {$sort}";
//    }
//    public function select($tablename,$fields='*'){
//        $wheresql = !empty($this->where)?" WHERE ".implode("? AND ",$this->where)."?":'';
//        $sql = "SELECT {$fields} FROM {$tablename} {$wheresql}";
//        $stmt =  $this->dbh->prepare($sql);
//        $r = $stmt->execute();
//        return $r!==false?true:false;
//    }
    /**
     * @param $tablename 表名
     * @param $data array 数据
     * @return bool
     */
    public function update($tablename,$data){
        //防止全表更改
        if(empty($this->where)){
            return false;
        }
        if(empty($data)||!is_array($data)){
            return false;
        }
        $setsql = '';
        foreach($data as $k=>$v){
            $setsql .= "`{$k}`=?,";
        }
        $setsql = trim($setsql,',');
        $wheresql = implode("? AND ",$this->where)."?";
        $sql = "UPDATE {$tablename} SET {$setsql} WHERE {$wheresql}";
        $data = array_values($data);
        $data = array_merge_recursive($data,$this->wheredata);
        $stmt =  $this->dbh->prepare($sql);
        $r = $stmt->execute($data);
        return $r!==false?true:false;
    }

    /**
     * @param $tablename
     * @param $data array()
     * @return bool|string
     */
    public function replace($tablename,$data){
        if(empty($data)||!is_array($data)){
            return false;
        }
        $setsql = '';
        $vals = '';
        foreach($data as $k=>$v){
            $setsql .= "`{$k}`,";
            $vals .="?,";
        }
        $setsql = trim($setsql,',');
        $vals = trim($vals,',');
        $sql = "REPLACE into {$tablename}({$setsql}) VALUES ($vals)";
        $stmt =  $this->dbh->prepare($sql);
        if($stmt->execute(array_values($data))){

            return $this->dbh->lastInsertId();
        }else{
//            var_dump($stmt->errorInfo());exit;
            $this->error($stmt);
        }
        return false;
    }
    public function insert($tablename,$data){
        if(empty($data)||!is_array($data)){
            return false;
        }
        $setsql = '';
        $vals = '';
        foreach($data as $k=>$v){
            $setsql .= "`{$k}`,";
            $vals .="?,";
        }
        $setsql = trim($setsql,',');
        $vals = trim($vals,',');
        $sql = "INSERT INTO {$tablename}({$setsql}) VALUES ($vals)";
       // echo $sql;exit;
        $stmt =  $this->dbh->prepare($sql);
        $data = is_array($data)?array_values($data):array($data);
        if($stmt->execute(array_values($data))){
            return $this->dbh->lastInsertId();
        }else{
//            var_dump($stmt->errorInfo());exit;
            $this->error($stmt);
        }
        return false;
    }

    /**
     * 获取总数
     * @param $sql
     * @param $param
     * @return bool
     */
    public function getTotal($sql,$param=''){
        $stmt =  $this->dbh->prepare($sql);
        $param = !empty($param)&&is_array($param)?array_values($param):array($param);
        //var_dump($stmt->execute($param));
        if ($stmt->execute($param)) {
            $data = $stmt->fetch(PDO::FETCH_NUM);
            return $data[0];
        }else{
//            var_dump($stmt->errorInfo());exit;
            $this->error($stmt);
        }
        return false;
    }

    /**
     * 执行非查询操作
     * @param $sql
     * @param string $param
     * @return bool|string
     */
    public function exec($sql,$param=''){
        $stmt =  $this->dbh->prepare($sql);
        $param = !empty($param)&&is_array($param)?array_values($param):array($param);
        if ($stmt->execute($param)) {
            $id = $this->dbh->lastInsertId();
            return $id?$id:TRUE;
        }else{
//            var_dump($stmt->errorInfo());exit;
            $this->error($stmt);
        }
        return false;
    }
    public function query($sql,$param=''){
        $stmt =  $this->dbh->prepare($sql);
        if(!empty($param)){
            $param = !empty($param)&&is_array($param)?array_values($param):array($param);
        }
        if ($param?$stmt->execute($param):$stmt->execute()) {
            $data = $stmt->fetchAll(PDO::FETCH_OBJ);
            return $data?$data:array();
        }else{
//            var_dump($stmt->errorInfo());exit;
            $this->error($stmt);
        }
        return false;
    }
    public function getOne($sql,$param=''){
        $data = array();
        $stmt =  $this->dbh->prepare($sql);
        if(!empty($param)){
            $param = !empty($param)&&is_array($param)?array_values($param):array($param);
        }
        if ($param?$stmt->execute($param):$stmt->execute()) {
            $data = $stmt->fetch(PDO::FETCH_OBJ);

        }else{
//            var_dump($stmt->errorInfo());exit;
            $this->error($stmt);
        }
        return $data?$data:array();
    }

    /**
     * beginTransaction 事务开始
     */
    public function beginTransaction()
    {
        $this->dbh->beginTransaction();
    }

    /**
     * commit 事务提交
     */
    public function commit()
    {
        $this->dbh->commit();
    }

    /**
     * rollback 事务回滚
     */
    public function rollback()
    {
        $this->dbh->rollback();
    }
    /**
     * checkFields 检查指定字段是否在指定数据表中存在
     *
     * @param String $table
     * @param array $arrayField
     */
    public function checkFields($table, $arrayFields){
        $fields = $this->getFields($table);
        foreach ($arrayFields as $key => $value) {
            if (!in_array($key, $fields)) {
                return false;
            }else{
                return true;
            }
        }
    }
    /**
     * getFields 获取指定数据表中的全部字段名
     *
     * @param String $table 表名
     * @return array
     */
    private function getFields($table){
        $fields = array();
        $recordset = $this->dbh->query("SHOW COLUMNS FROM $table");
        $this->getPDOError();
        $recordset->setFetchMode(PDO::FETCH_ASSOC);
        $result = $recordset->fetchAll();
        foreach ($result as $rows) {
            $fields[] = $rows['Field'];
        }
        return $fields;
    }
    /**
     * destruct 关闭数据库连接
     */
    public function __destruct(){
        $this->dbh = null;
        self::$_instance = null;
    }
}
```

#### 4.2 配置

```php
<?php
$active_group = 'default';
$query_builder = TRUE;
$db['default'] = array(
//    'hostname' => '172.31.16.8',
//    'username' => 'cctvnewsplatform',
//    'password' => '!Q@W#E$R',
    'hostname' => 'localhost',
    'username' => 'root',
    'password' => '123456aA',
    'database' => 'test',
    'dbprefix' => '',
    'port'     => '3306',
    'pconnect' => FALSE,
    'cache_on' => FALSE,
    'cachedir' => '',
    'char_set' => 'utf8',
    'dbcollat' => 'utf8_general_ci',
);
$db['db_slave'] = array(
    'hostname' => '',
    'username' => '',
    'password' => '',
    'database' => 'cctvnewsplatform',
    'dbprefix' => 'cctvnewsplatform_',
    'port'     => '3306',
    'pconnect' => FALSE,
    'cache_on' => FALSE,
    'cachedir' => '',
    'char_set' => 'utf8mb4',
    'dbcollat' => 'utf8mb4_general_ci',
);
$db['vgc'] = array(
    'hostname' => '',
    'username' => '',
    'password' => '',
    'database' => 'cctvnews',
    'dbprefix' => '',
    'port'     => '3306',
    'char_set' => 'utf8mb4',
    'dbcollat' => 'utf8mb4_general_ci',
);
$db['log'] = array(
    'hostname' => '',
    'username' => 'log',
    'password' => '',
    'database' => 'log',
    'dbprefix' => 'cctvnewsplatform_',
    'port'     => '3306',
    'pconnect' => FALSE,
    'cache_on' => FALSE,
    'cachedir' => '',
    'char_set' => 'utf8',
    'dbcollat' => 'utf8_general_ci',
);
```
### 5.mysqli类
#### 5.1 成员变量
```php
affected_rows: 返回先前操作受影响的行数(-1:error,0:受影响行数0)

```
#### 5.2 mysqli方法
```php
mysqli_affected_rows($link): 同成员变量affected_rows
```

#### 5.1 mysqli_result方法
```php
fetch_all(): 获取所有数据,生成数组[选项:MYSQLI_ASSOC,MYSQLI_NUM,MYSQLI_BOTH]

```

### 6.pdo
#### 6.1 pdo连接
```php
$dsn = 'mysql:host=localhost;dbname=testdb;port=3306(默认);unixsocket=/path/to/abc.socket;charset=utf8';
$options = [
    PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8', //同dsn中的charset
];
$dbh = new PDO($dsn, $username, $password, $options);
```