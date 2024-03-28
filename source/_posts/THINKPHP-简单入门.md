---
title: THINKPHP 简单入门
date: 2016-11-25 17:45:47
tags: [Hello world,PHP]
toc: true
description: THINKPHP是一个快速简单的轻量级国产PHP开发框架。
---

>THINKPHP是一个快速简单的轻量级国产PHP开发框架。

本文介绍了其基础的功能，方便自己查阅和回顾。
闲话少叙，下面开始:)

### 一、环境
建议使用wamp、lamp搭建测试环境，下载安装即可。
从官网下载THINKPHP 3.2版本的压缩包，解压后放到wamp(lamp)下的www文件夹内。
在浏览器地址栏输入http://localhost, 即可看到欢迎界面。

### 二、路由
THINKPHP采用模块化设计，每个模块文件下都有相应的配置文件。
本文只配置一个模块Home，为了后面方便，仅需在Application/Common/Conf/config.php
配置即可。即此后提到的配置文件语句，均写到此文件内。
打开config.php, 保留语句为

```php
return array();
```
在括号内写入，

```php
'URL_ROUTER_ON' => true,              //开启路由
'URL_ROUTE_RULES' => array(           //CURD操作
    'create' => 'Home/Index/cre',
    'update' => 'Home/Index/upd',
    'retrieve' => 'Home/Index/ret',
    'delete' => 'Home/Index/del',
    ),                               
```
此处路由规则的含义是，当在浏览器地址栏输入
http://localhost/index.php/create 
时，相当于访问
http://localhost/index.php/Home/Index/cre,
即Home模块下Index控制类下的cre方法，以此类推。
当路由中包含参数时，在参数前添加':'，匹配时$_GET[]会获取相应的参数，例

```
'create/:id\d/:code\d' => 'Home/Index/cre',
```
\d为使用正则限制路由参数。

### 三、控制器
控制器类文件命名的规则是首字母大写，驼峰命名法，并以.class.php结尾。
在路径Application/Home/Controller/下，新建文件重命名为IndexController.class.php
打开文件添加代码，

```php
namespace Home\Controller;
use Think\Controller;
class IndexController extends Controller {
    public function ret(){
        $Test = D('test');               //使用内置D方法初始化模型类Test
        $temp = $Test->get_coupon();     //使用模型类中的get_coupon()方法
        $this -> assign('temp',$temp);   //模板赋值
        $this -> display('main');        //模板渲染
    }
    public function upd(){
        $data['id'] = 99;
        $data['code'] = 123456789;
        $Test = M('test');              //内置M方法
        $Test -> add($data);
    }
    public function cre(){
        $_GET['id'] = $data['id'];      //获取url中的参数
        $_GET['code'] = $data['code'];
        $Test = M('test');
        $Test -> where('id=2') -> save($data);
    }
    public function del(){
        $data['id'] = 99;
        $data['code'] = 123456789;
        $Test = M('test');
        $Test -> where('id=1') -> delete();
    }
}
```
以上代码的方法对应着CURD操作，当路由匹配到方法时，对数据表进行相应的操作。

当不使用$_GET，而是直接将url内的参数传入到控制器内的方法时，
首先，在配置文件中添加

```php
'URL_PARAMS_BIND'       =>  true,
```
开启参数绑定。
以上述cre()函数为例,

```php
public function cre($id,$code){
        $data['id'] = $id;
        $data['code'] = $code;
        $Test = M('test');
        $Test -> where('id=2') -> save($data);
}
```
访问 http://localhost/index.php/Home/Index/ret/id/01/code/12345 时，即可传入相应的参数。
当然，路由规则也需要加入相应的关键字。

这里简单说一下内置D方法和M方法的区别。

* 当使用THINKPHP模型中的自动验证、关联模型等复杂业务逻辑功能时，使用D方法。
* 当仅使用CURD操作时，使用M方法。

### 四、模型
模型类文件创建在Application/Home/Model/下，同样采用首字母大写，驼峰规则，以
.class.php结尾命名。注意文件名要与控制器类的名称一致。
打开文件添加代码，

```php
namespace Home\Model;
use Think\Model;
class TestModel extends Model {
    public function get_coupon(){
        $arr = $this -> select();
        return $arr;
    }
}
```
因为在控制器类中使用了get_coupon()方法，所以这里要把它加进去。如果有其他对模型操作的方法都可以写进来，当然写在控制器类也可以，但是为了代码功能明晰，建议分开写。

数据库连接参数写入配置文件，

```php
'DB_TYPE'  => 'mysql',
'DB_USER'  => 'root',
'DB_PWD'   => '1234',
'DB_HOST'  => '127.0.0.1',
'DB_PORT'  => '3306',
'DB_NAME'  => 'coupon',  //这是测试用的数据表
```
开发文档中提到了可以将配置信息生成为数组，但是不在模型类调用这一数组时，不要将配置信息组合为数组。

测试表coupon内容如下：

id  |code     
----|------
0   |SUFFLzS
1   |hZxakCl
2   |xD2gz0N
3   |6WwLt7U
4   |xLSxY1n
5   |bn422Ka
6   |55BaQ0C
7   |Ui1Jc8P

### 五、模板
模型类文件创建在Application/Home/View/Index/下,与控制器类内代码相对应，新建main.html文件，添加以下代码：

```html
<html>
<head>
<title>测试主页</title>
</head>
<body>
<foreach name='temp' item='coupon'>
{$coupon.id}:{$coupon.code}<br/>
</foreach>
</body>
</html>
```

`<foreach>`是内置的标签，可以将数据循环输出。其中temp是控制器类内的数组名称，coupon为自定义名称，id和code为数组内的键名。

在浏览器地址栏访问 http://localhost/index.php/Home/Index/cre 时，
页面显示为

0:SUFFLzS
1:hZxakCl
2:xD2gz0N
3:6WwLt7U
4:xLSxY1n
5:bn422Ka
6:55BaQ0C
7:Ui1Jc8P

### 六、总结与吐槽
以下是总结：
　　至此，THINKEPHP 3.2的快速入门教程已经完成。本文对路由规则分配，控制器操作模型、渲染模板进行了简单的介绍。如果想深入了解框架的其他知识，还请移步官方文档。至于MySQL的相关知识，还是另开一篇文章好了。

以下是吐槽：
　　“什么AUIMD方法真的是反人类啊，不能顾名思义的方法都不是好方法！”，一位绞尽脑汁想不到新变量名称的程序员说道。