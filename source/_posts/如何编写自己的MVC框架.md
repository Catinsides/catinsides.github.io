---
title: 如何编写自己的MVC框架
date: 2017-01-26 14:22:56
tags: PHP
toc: true
description: MVC是当下Web开发中流行的设计模式。从自己动手编写框架开始学习，能够有效的学习MVC结构和提高自身的面向对象编程能力。
---
MVC是一种软件设计模式。顾名思义，这些字母分别代表着模型（Model）、视图（View）和控制器（Controller）。PHP中的MVC旨在分离业务与逻辑层，使得开发过程中的结构清晰，大大提高了效率。当下有很多流行的PHP框架均使用了MVC模式。如果是新手入门，查看这些框架的源代码肯定会一头雾水，毫无思路。上手新框架等于重新开始学习，这必然会消耗不必要的精力和时间。而学习MVC框架最好的方式就是自己动手写一个框架，既学习了设计模式，又巩固了编程能力，何乐而不为。更重要的是，在开发过程中能够将自己的想法融入到框架中去，从而实现各种功能。

### 准备工作
这篇文章是我[Google](https://www.google.com.hk/#newwindow=1&safe=active&q=code+your+own+php+mvc+framework)学习很多文章后的总结。将各路大神的代码汇总，然后择取出MVC必要的元素，根据个人需要编写出的小型框架，而非使用Composer组装出的框架。这个框架包含以下特点：

* MVC结构
* 单一入口
* 自动加载类
* 数据库封装

根据这个思路，开始coding.

### MVC流程
日常上网过程中可以简化理解为人与服务器中数据库交互的过程。
每一次网页上的请求发送到服务器，服务端将请求通过控制器处理后，再传送到模型。
模型内包含业务逻辑，负责处理如何与数据库交换数据，数据处理完成后，发送到视图。
视图将传来的数据渲染后，最终返还用户。这样便完成了一次请求。
熟悉MVC流程后，就有了框架的宏观印象，方便理清结构。

### 编写框架
#### 建立目录
首先，在项目目录下建立如下文件和文件夹:
```
|--application
|    |--controllers
|    |--models
|    |--views
|--config
|--framework
|--.htaccess
|--index.php
```

`application` 内放置用户编写的MVC文件；
`config` 内放置数据库设置文件；
`framework` 内放置框架的核心文件；
`.htaccess` 为重定向文件；
`index.php` 为入口文件。

#### 重定向
`.htaccess`是服务器的配置文件，它用来将所有的请求转到入口文件处理。这样一来，网页程序便有了单一入口，掌控用户请求，避免熊孩子的无理取闹（笑。还可以生成对搜索引擎友好的URL。配置内容如下：
Apache服务器：
```
RewriteEngine on
 RewriteCond %{REQUEST_FILENAME} !-f
 RewriteCond %{REQUEST_FILENAME} !-d
 RewriteRule ^(.+)$ index.php/$1 [L]
```
nginx服务器：
```nginx
location / {
    if(!-e $request_filename){
        rewrite ^(.+)$ /index.php/$1 break;
    }
}
```

#### 入口文件
打开同目录下的index.php文件，输入以下代码：
```php
<?php
require './framework/framework.php';
```
内容很简单，就是包含核心文件夹内的入口文件。其实在这里可以定义一些预定义常量方便调用，例如：
```php
define('APP_PATH', __DIR__.'/');
define('APP_DEBUG', true);
define('APP_URL', 'http://localhost');
```

#### 核心文件
外入口文件(index.php)将请求传递给核心文件的内入口文件(framework.php)后，接下来的工作由核心文件完成。framework.php的内容如下：

```php
    <?php
    defined('FRAME_PATH') or define('FRAME_PATH', __DIR__.'/');
    defined('APP_PATH') or define('APP_PATH', dirname($_SERVER['SCRIPT_FILENAME']).'/');
    defined('APP_DEBUG') or define('APP_DEBUG', false);
    defined('CONFIG_PATH') or define('CONFIG_PATH', APP_PATH.'config/');
    defined('RUNTIME_PATH') or define('RUNTIME_PATH', APP_PATH.'runtime/');
    
    require APP_PATH . 'config/config.php';
    
    require FRAME_PATH . 'Core.php';
    
    $framework = new Core;
    $framework->run();
```

同样，在这里进行预定义常量和包含文件的工作，并实例化核心文件，让网页程序得以运行。题外话：看了一些外国人写的教程，大部分都喜欢用`$framework->bootstrap()`, 而这个`bootstrap()`并非代表着前端框架Bootstrap（原谅我脑洞大，第一个想到的就是这个），而是一种命名习惯吧。
从上述代码可以看出，包含了两个文件，其中`config.php`是数据库配置文件，要把它存放在`config`文件夹内，其内容如下：
```php
    <?php
    define('DB_NAME', 'test_db');
    define('DB_USER', 'root');
    define('DB_PASSWORD', '1234');
    define('DB_HOST', 'localhost');
```
根据个人情况修改即可。
回到`framework`文件夹，建立`Core.php`，这是框架核心文件，内容如下：
```php
    <?php
    class Core
    {
        public function run()
        {
            spl_autoload_register(array($this, 'loadClass'));
            $this->setReporting();
            $this->removeMagicQuotes();
            $this->unregisterGlobals();
            $this->route();
        }
       
        public function route()
        {
            $controllerName = 'Index';
            $action = 'index';
            $param = array();
            $url = isset($_GET['url']) ? $_GET['url'] : false;
                if ($url) {      
                    $urlArray = explode('/', $url);
                    $urlArray = array_filter($urlArray);
                    $controllerName = ucfirst($urlArray[0]);
                    array_shift($urlArray);
                    $action = $urlArray ? $urlArray[0] : 'index';
                    array_shift($urlArray);
                    $param = $urlArray ? $urlArray : array();
                }
            
            $controller = $controllerName . 'Controller';
            $dispatch = new $controller($controllerName, $action);
            
            if ((int)method_exists($controller, $action)) {
                call_user_func_array(array($dispatch, $action), $param);
            } else {
                exit($controller . "控制器不存在");
            }
        }
        
        public function setReporting()
        {
            if (APP_DEBUG === true) {
                error_reporting(E_ALL);
                ini_set('display_errors','On');
            } else {
                error_reporting(E_ALL);
                ini_set('display_errors','Off');
                ini_set('log_errors', 'On');
                ini_set('error_log', RUNTIME_PATH. 'logs/error.log');
            }
        }
        
        public function stripSlashesDeep($value)
        {
            $value = is_array($value) ? array_map(array($this, 'stripSlashesDeep'), $value) : stripslashes($value);
            return $value;
        }
        
        public function removeMagicQuotes()
        {
            if (get_magic_quotes_gpc()) {
                $_GET = isset($_GET) ? $this->stripSlashesDeep($_GET ) : '';
                $_POST = isset($_POST) ? $this->stripSlashesDeep($_POST ) : '';
                $_COOKIE = isset($_COOKIE) ? $this->stripSlashesDeep($_COOKIE) : '';
                $_SESSION = isset($_SESSION) ? $this->stripSlashesDeep($_SESSION) : '';
            }
        }
        
        public function unregisterGlobals()
        {
            if (ini_get('register_globals')) {
                $array = array('_SESSION', '_POST', '_GET', '_COOKIE', '_REQUEST', '_SERVER', '_ENV', '_FILES');
                foreach ($array as $value) {
                    foreach ($GLOBALS[$value] as $key => $var) {
                        if ($var === $GLOBALS[$key]) {
                            unset($GLOBALS[$key]);
                        }
                    }
                }
            }
        }
        
        public static function loadClass($class)
        {
            $frameworks = FRAME_PATH . $class . '.class.php';
            $controllers = APP_PATH . 'application/controllers/' . $class . '.class.php';
            $models = APP_PATH . 'application/models/' . $class . '.class.php';
            if (file_exists($frameworks)) {
                include $frameworks;
            } elseif (file_exists($controllers)) {
                include $controllers;
            } elseif (file_exists($models)) {
                include $models;
            } else {
                echo "控制器类或模型类不存在！"
            }
        }
    }
```
从`run()`中可以理解这一核心文件的运行机制，首先`spl_autoload_register()`用于自动加载类文件，在以后的类实例化中，可以方便的直接调用，而不是一个一个的require;接下来`setReporting()`用来开启调试模式，并记录到日志中;`removeMagicQuotes()`用于移除敏感字符;`unregisterGlobals()`用于检测并移除系统全局变量，避免如`_GET`不到变量的问题。准备工作完成后，`route()`方法截取URL，将形如：
`localhost/?url=controller/action/parameters`
的URL分离为`controller`,`action`和`parameters`，其中`controller`即`application`文件夹下用户自定义的控制器类文件，`action`为相应类文件内的方法，`parameters`为传入方法内的变量。实例化控制器后，并调用view方法。
这里的URL有点古怪，因为使用是`GET`方法获取的。如果想去掉`?url`也未尝不可，使用`$_SERVER['PATH_INFO']`可获取`index.php`后的内容，以`/`分离可获取控制器和方法；而`？`后的内容可以使用`$_SERVER['QUERY_STRING']`获取。`PATHINFO`的URL模式对搜索引擎更加友好。

#### MVC基类
接下来，在核心文件夹内创建`Controller.class.php`,`Model.class.php`和`View.class.php`三个基类文件，用户创建的子类文件需要继承基类文件，所以可以把一些常用的方法写入到这个文件内，日后使用时直接调用即可。同时，因为这些基类文件负责着整体的MVC流程，所以其中要包含一些总体调度方法。
`Controller.class.php`内容如下：
```php
    <?php 
    class Controller
    {
        protected $_controller;
        protected $_action;
        protected $_view;
        
        public function __construct($controller, $action)
        {
            $this->_controller = $controller;
            $this->_action = $action;
            $this->_view = new View($controller, $action);
        }
        
        public function assign($name, $value)
        {
            $this->_view->assign($name, $value);
        }
        
        public function render()
        {
            $this->_view->render();
        }
    }
```
从代码内容可知，控制器基类用于分配变量和传递变量给视图，所以在控制器子类中，可以直接使用`$this->render()` 进行渲染。
`Model.class.php`内容如下：
```php
    <?php
        class Model extends Sql
        {
            protected $_model;
            protected $_table;
            public function __construct()
            {
                $this->connect(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME);
                $this->_model = get_class($this);
                $this->_model = substr($this->_model, 0, -5);
                $this->_table = strtolower($this->_model);
            }   
        }
```
此类继承自数据库基类`Sql.class.php`,其内容如下：
```php
    <?php 
    class Sql
    {
        protected $_dbHandle;
        protected $_result;
        private $filter = '';
        public function connect($host, $user, $pass, $dbname)
        {
            try {
                $dsn = sprintf("mysql:host=%s;dbname=%s;charset=utf8", $host, $dbname);
                $option = array(PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC);
                $this->_dbHandle = new PDO($dsn, $user, $pass, $option);
            } catch (PDOException $e) {
                exit('错误: ' . $e->getMessage());
            }
        }
        
        public function select($id)
        {
            $sql = sprintf("select * from `%s` where `id` = '%s'", $this->_table, $id);
            $sth = $this->_dbHandle->prepare($sql);
            $sth->execute();
            return $sth->fetch();
        }     
    }
```
这里只封装了普通查询的方法，可以以这个方法为模板编写其他的CURD方法。再以后的使用过程中，就可以直接使用封装后的方法操作数据库
`View.class.php`内容如下：
```php
    class View
    {
        protected $variables = array();
        protected $_controller;
        protected $_action;
    
        function __construct($controller, $action)
        {
            $this->_controller = $controller;
            $this->_action = $action;
        }
        
        public function assign($name, $value)
        {
            $this->variables[$name] = $value;
        }
        
        public function render()
        {
            extract($this->variables);
            //加载自定义模板
        }
    }
```
这样，框架的核心部分就完成了。测试框架时，像使用其他框架一样，在`application`文件夹内的子文件夹分别建立用户类文件，并继承基类文件，写入测试方法和语句即可。

### 总结
当我翻阅了很多编写入门级MVC框架教程之后，发现很多代码内容都是一样的。比如截取URL和自动加载类的方法。查看这些源码很容易理解其原理和过程。由于目前能力有限，我还不能倒背如流般的coding出这些代码，只得借鉴大神的代码，让自己学习和巩固。在开发过程中，也可以使用其他人造好的轮子，利用`Composer`下载到项目文件夹内，然后根据命名空间规则引用，就可以使用了。这样便极大的加快了项目的开发速度。但是我不喜欢黑箱操作，单纯的调用API反而更容易让人摸不着北。创造，才是一件令人开心的事。