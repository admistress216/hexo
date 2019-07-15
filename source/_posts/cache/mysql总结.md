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

### 7.1DCL(data control language)语句
#### 7.11 创建新用户并授权
```php
//新建lizengcai用户,并授权book数据库所有表select,insert,delete权限(对象授权)
grant select,insert,delete on book.* to 'lizengcai'@'localhost' identified by '123';
//收回lizengcai在book表上的insert权限
revoke insert on book.* from 'lizengcai'@'localhost'
```

#### 7.12 查看帮助
```php
? contents:支持的所有命令
? show: show相关命令
? alter table: 修改表相关命令
```

#### 7.13 常用命令
```php
show databases: 展示数据库(SCHEMATA表)
show tables from schemaname: 表信息(TABLES表)
show columns from schemaname.tablename: 列信息(COLUMNS表)
show index from schemaname.tablename: 索引信息(STATISTICS表)
```

#### 7.14 函数使用
```php
//字符串函数
concat(s1,s2...): 链接字符串
    select concat('aa','bb', 'cc'),concat('aa',null);  //aabbcc,null
insert(str,x,y,instr): 将字符串str从x位置开始,y字符长度的子串替换为instr
    select insert('beijing2008you', 12, 3, 'me'); //beijing2008me
lower(),upper(): 转换大小写
    select lower('ME'),upper('me');//me,ME
left(str,x),right(str,y): 返回左边x字符/右边y字符(x/y为null时返回空)
lpad(str,n,pad),rpad(str,n,pad): 使用pad对str左边/右边进行填充,直至总共满n个字符
    select lpad('2008', 15, 'beijing'),rpad('beijing',15,'2008');//beijingbeij2008,beijing20082008
ltrim(str),rtrim(str),trim(str): 去除字符串左边/右边/两边的空格
    select length(ltrim('   |beijing')),length(rtrim('beijing|   '));//8,8
repeat(str, n): 重复str n次
    select repeat('mysql ',3); //mysql mysql mysql
replace(str, a, b): //用b替换str中的a
    select replace('beijing_2010', '_2010', '2008'); //beijing2008
strcmp(s1,s2): 比较字符串s1和s2的ASCII码值大小(小,返回-1,相等,返回0,大,返回1)
    select strcmp('a','b'),strcmp('b','b'),strcmp('c','b'); //-1,0,1
substring(str,x,y): 返回字符串x位置起y长度的字符串(下标从1开始)
    select substring('beijing2008',8,4),substring('beijing2008',1,7); //2008,beijing
    
//数值函数
abs(x): x的绝对值
ceil(x)/floor(x): 大于x的最小整数/小于x的最大整数
    select ceil(-0.8),ceil(0.8); //0,1
    select floor(-0.8),floor(0.8); //-1,0
mod(3,2): 求模
rand(): 返回0-1之间的随机值
    //产生0-100之间的数
    select ceil(100*rand()),ceil(100*rand());
round(x,y): x的四舍五入(保留y位小数)
    select round(1.1),round(1.1,2),round(1,2); //1,1.10,1
truncate(x,y): 截断,不进行四舍五入
    select round(1.235,2),truncate(1.235,2); //1.24,1.23

//时间函数
curdate()/curtime()/now(): 当前日期/时间/日期和时间
unix_timestamp()/unix_timestamp(now())/unix_timestamp(date): 当前时间戳/日期的时间戳
from_unixtime(unixtime): 由时间戳转换为时间
    select from_unixtime(unix_timestamp(now())) //当前时间相当于now()
week(curdate()/now()): 返回日期是当年的第几周
year(curdate()/now()): 返回所给日期是哪一年
hour/minute(curtime()/now()): 返回时间的小时/分钟
monthname(curdate()/now()): 返回月份
date_format(date,fmt): 按字符串fmt格式化日期date,date取值:now(),curdate(),curtime()
date_add(date, interval exp type): 返回与所给日期date相差interval时间段的日期(可用负数表示之前的日期)
    select now() current,date_add(now(),interval 31 day) after31days,date_add(now(),interval '1_2' year_month) after_oneyear_twomonth;
date_diff(date1,date2): 表示date1和date2之间相差的天数
    select datediff('2019-6-26', now());

//流程函数

//其他函数
last_insert_id(): 返回最后插入id,如果一次插入多条,则显示第一条记录插入时的id
```

#### 7.21 存储引擎
```php
show engines: 展示存储引擎
alter table ai engine=innodb;
alter table ai auto_increment=10;  //更改自增值(注:存放在内存中)

//创建组合索引
create table autoincre_demo(
d1 smallint not null auto_increment,
d2 smallint not null,
name varchar(10),
index(d2,d1)
)engine=myisam;

//myisam表(分三种存储格式: 静态表,动态表,压缩表)
    静态表: 默认的,固定长度,存储时填充空格,取出时去除空格,原有的数据中空格也会去掉(后面的),数据易恢复
        create table myisam_char(name char(10)) engine =myisam;
        insert into myisam_char values('abcde'),('abcde   '),('  abcde'),('  abcde  ');
        select name,length(name) from myisam_char;
    动态表: 空间占用相对较小,频繁更新和删除会产生碎片,需定期执行optimize table或myisamchk -r来改善性能,数据不易恢复
    压缩表: 占用非常小的空间
    
    存储:
        每个myisam表在磁盘上被存储成3个文件
            .frm:存储表定义
            .MYD(MY DATA):存储数据
            .MYI(MY INDEX):存储索引
    备份与恢复:
        直接复制三个文件


//innodb表
    #创建父表
    CREATE TABLE `country` (
      `country_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,
      `country` varchar(50) NOT NULL,
      `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
      PRIMARY KEY (`country_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
    
    #创建子表(外键)
    //对父表进行删除和更新操作时,字表的对应的操作含义(默认为restrict):
        restrict(限制)/no action:子表有数据则不允许更新/删除父表,
        cascade(级联):更新/删除父表时,同时更新/删除子表记录
        set null:更新/删除父表时,对子表响应的字段做set null操作
    //注意:
        在导表/alter table/load data过程中,关闭外键检测set foreign_key_checks=0;处理完再打开set foreign_key_checks=1;
        
    CREATE TABLE `city` (
      `city_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,
      `city` varchar(50) NOT NULL,
      `country_id` smallint(5) unsigned NOT NULL,
      `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
      PRIMARY KEY (`city_id`),
      KEY `idx_fk_country_id` (`country_id`),
      CONSTRAINT `fk_city_country` FOREIGN KEY (`country_id`) REFERENCES `country` (`country_id`) ON UPDATE CASCADE
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
    
    存储: 
        .ibd: 数据,
        .frm: 定义,
        共享空间的数据字典
    备份与恢复:
        alter table table_name discard table_space
        alter table table_name import table_space
```
### 7.2 选择合适的存储引擎
#### 7.21 char和varchar
1. 不同的存储引擎对char和varchar的使用原则有所不同,这里概述如下:
> - _myisam存储引擎_: 建议使用固定长度数据列代替可变长度的数据列.
> - _innodb存储引擎_: 建议使用varchar类型,内部的行存储没有区分固定长度和可变长度列(所有数据行都使用指向数据列值的头指针),因此,在本质上使用固定长度列的char性能不一定比可变长度列
性能要好,因此,主要的性能因素是数据行使用的存储总量,因为固定长度char所用空间大于varchar,因此使用varchar来最小化需要处理的数据行和磁盘I/O是比较好的

![测试一下](/images/1560437658.png)






























