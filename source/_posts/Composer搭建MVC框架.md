---
title: Composer搭建MVC框架
date: 2017-03-30 11:11:28
tags: [PHP]
toc: true
description: Composer是一个依赖管理工具，利用PSR标准和PHP命名空间的特性，构造出繁荣的PHP生态系统。
---
>Composer是一个依赖管理工具，利用PSR标准和PHP命名空间的特性，构造出繁荣的PHP生态系统。

### Composer是什么？
**先从别处摘抄一段简介，感受一下Composer的功能。**
Composer不是一个包管理器。它涉及"pachages"和"libraries"，但它在每个项目的基础上进行管理，在你项目的某个目录中（例如vendor）进行安装。默认情况下它不会在全局安装任何东西。因此，这仅仅是一个依赖管理。

这种想法并不新鲜，Composer受到了node's npm和ruby's bundler的强烈启发。而当时PHP下并没有类似的工具。

Composer将这样为你解决问题：

1.你有一个项目依赖于若干个库。
2.其中一些库依赖于其他库。
3.你声明你所依赖的东西。
4.Composer会找出哪个版本的包需要安装，并安装它们（将他们下载到你的项目中）。

好了，了解功能之后，lets coding！

### 准备工作
#### Composer安装
[百度一下](http://docs.phpcomposer.com/00-intro.html#Installation-Windows)

#### 建立项目
在一个目录下建立文件夹，名字就叫smvc(simple mvc)吧。
在文件夹下新建文件composer.json:
```json
{
	"require":{

	}
}
```
在此目录下，开启命令行运行：
```bash
composer update
```
几秒钟后会生成以下文件：
```plaintext
|--vendor/
|	|--scrapy.cfg
|	|	|--autoload_classmap.php
|	|	|--autoload_namespaces.php
|	|	|--autoload_psr4.php
|	|	|--autoload_real.php
|	|	|--autoload_static.php
|	|	|--ClassLoader.php
|	|	|--installed.json
|	|	|--LICENSE
|	|--autoload.php
|--composer.json
```
OK,准备工作完成，可以开工了。

### MVC搭建
在上一篇文章中[如何编写自己的MVC框架](https://catinsides.github.io/catinsides.github.io/2017/01/26/%E5%A6%82%E4%BD%95%E7%BC%96%E5%86%99%E8%87%AA%E5%B7%B1%E7%9A%84MVC%E6%A1%86%E6%9E%B6/)已经简单介绍了MVC框架的结构和运作流程。所以不多废话，直接按照MVC的思路逐步搭建出一个小型的MVC框架。

#### 路由
既然用了Composer，就不需要像之前一样自己手动撸代码了，而是直接到全球最大同性交友平台Github搜索关键字router，[搜一下](https://github.com/search?l=PHP&q=router&type=Repositories&utf8=%E2%9C%93)
选择一个契(xi)合(huan)项目的包，使用composer进行下载。这里我选择的是klein/klein.php(a fast&flexible router)。没有什么特殊的理由，只是因为出现在了首页，并且stars数量最多而已。
然后打开composer.json，添加安装信息（一般情况下会列在开发者文档中）:
```json
{
	"require":{
		"klein/klein": "dev-master"
	}
}
```
然后再命令行中运行`composer update`,等待安装下载完成，速度视网络情况而定。
安装完成后，会在`vender`文件夹生成相关目录。
在根目录下建立入口文件`index.php`：
```php
<?php
//自动加载
require_once './vendor/autoload.php';

//载入路由文件
require './core/routes.php';
```
然后新建文件夹core，在里面新建routes.php，使用刚才下载的路由包:
```php
<?php
$klein = new \Klein\Klein();

$klein->respond('GET', '/hello', function () {
   return "hello world!";
});

$klein->dispatch();
```
打开wamp在本地测试一下能否成功运行。点击Localhost后自动打开浏览器，在地址栏地址后输入`/hello`会出现hello world！字样，OK运行成功。其实每个包的作者都会在项目中写出详细的开发文档，只要按照给定的API设定好路由即可。在这里以能够实现数据库常用的增删改查为目标构建出这个小型框架。

根据逻辑重构路由文件：
```php
<?php
$klein = new \Klein\Klein();

//设定默认主页，各项请求在这里完成
$klein->respond(function () {
    include './tpl/index.php';
});

//将路由指向控制器文件
$klein->respond('GET', '/create', function () {
   Controller::create();
});

$klein->respond('GET', '/update', function () {
   Controller::update();
});

$klein->respond('GET', '/retrieve', function () {
   Controller::retrieve();
});

$klein->respond('GET', '/delete', function () {
   Controller::delete();
});

$klein->dispatch();
```

#### 模型
在根目录创建文件夹config，新建config.php文件写入数据库配置信息：
```php
<?php
define("DB_HOST", "localhost");
define("DB_USER", "root");
define("DB_PASSWORD", "1234");
define("DB_NAME", "test");
```
在core文件夹创建基本的模型类Model.php,在里面使用PDO编写基本的数据库操作。
```php
<?php 
//引入数据库配置文件
require './config/config.php';

class Model
{
	protected $_dbHandle;
	protected $_table;

	public function __construct($table)
	{
		$this ->connect(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME);
		$this ->_table = $table;
	}

    public function connect($host, $user, $pass, $dbname)
    {
        try {
            $dsn = sprintf("mysql:host=%s;dbname=%s;charset=utf8", $host, $dbname);
            $option = array(PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC);
            $this->_dbHandle = new PDO($dsn, $user, $pass, $option);
        } catch (PDOException $e) {
            exit('Wrong: ' . $e->getMessage());
        }
    }

    public function select()
    {
        $sql = sprintf("SELECT * FROM %s", $this->_table);
        $sth = $this->_dbHandle->prepare($sql);
        $sth->execute();

        return $sth->fetch();
    }

    public function add($id, $data)
    {
        $sql = sprintf("INSERT INTO %s(id, name) VALUES (%d, '%s')", $this->_table, $id, $data);
        $this ->_dbHandle->query($sql);
    }

    public function delete($id)
    {
    	$sql = sprintf("DELETE FROM %s WHERE `id` = %d", $this->_table, $id);
        $sth = $this->_dbHandle->prepare($sql);
        $sth->execute();
    }

    public function update($id, $data)
    {
    	$sql = sprintf("UPDATE %s SET name = '%s' WHERE `id` = %d", $this->_table, $data, $id);
        $this ->_dbHandle->query($sql);
    }
}
```
数据库文件编写完成。

#### 控制器
在core文件夹新建控制器类Controller.php,在其内部使用Model.php与数据库进行连接:
```php
<?php 
class Controller{
	public static function create()
	{
		$model = new Model('test');
		$model ->add('1', 'Mike');
		echo "插入成功！";
	}

	public static function update()
	{
		$model = new Model('test');
		$model ->update('1', 'John');
		echo "更新成功！";
	}

	public static function retrieve()
	{
		$model = new Model('test');
		$results = $model ->select();
		extract($results);
		require "./tpl/index.php";
	}

	public static function delete()
	{
		$model = new Model('test');
		$model ->delete('1');
		echo "删除成功！";
	}
}
```
#### 数据库
```sql
CREATE TABLE `test`(
	`id` int(1),
	`name` varchar(5)
)ENGINE=Innodb DEFAULT CHARSET=utf8;

```
#### 自动加载
类文件编写完成后，如果在首页测试会出现找不到相关类的错误。因为还没有在composer.json中设置为自动加载，所以在文件中添加字段，最终代码为：
```json
{
	"require":{
		"klein/klein": "dev-master"
	},
	"autoload":{
		"classmap":[
			"core"
		]
	}
}

```
这个MVC框架的视图层没有特别复杂的实现，而是直接采用了加载文件，输出变量的方式。
至此，这个框架的基本要求已经全部实现了。

### 小结
在本文中，简单利用了composer的下载和自动加载类的功能，实现了MVC框架。虽然这个框架瘦骨嶙峋到几乎只有骨骼……代码还有非常大的优化和拓展空间。而这些就可以用composer强大的依赖管理来实现。比如项目中需要一个验证码的功能，只需要下载并注册到composer.json中，然后正确引入即可使用。Composer这些功能特性极大方便了PHP开发者的快速开发。