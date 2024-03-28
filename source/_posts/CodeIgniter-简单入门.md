---
title: CodeIgniter 简单入门
date: 2016-11-26 17:42:56
tags: [Hello world,PHP]
toc: true
description: CodeIgniter 是一套给 PHP 网站开发者使用的应用程序开发框架和工具包。它的目标是让你能够更快速的开发，它提供了日常任务中所需的大量类库， 以及简单的接口和逻辑结构。通过减少代码量，CodeIgniter 让你更加专注于你的创造性工作。
---
>CodeIgniter 是一套给 PHP 网站开发者使用的应用程序开发框架和工具包。 它的目标是让你能够更快速的开发，它提供了日常任务中所需的大量类库， 以及简单的接口和逻辑结构。通过减少代码量，CodeIgniter 让你更加专注于你的创造性工作。

本文介绍该框架（简称CI）的基本功能，方便自己查阅和回顾。
闲话少叙，下面开始:)

### 环境
测试环境建议使用wamp或lamp，下载后安装即可。
从CI官网下载压缩包，解压到wamp(lamp)目录下的www文件夹内，
在浏览器地址栏输入 http://localhost ，即可看到欢迎界面。

### 路由
路由配置文件routes.php在框架根目录下application/config文件夹内，打开后看到一堆注释和保留语句。注释部分算是一部分教程，方便理解框架内容。
保留语句是路由名称数组，左面是匹配规则，右面是对应的控制器类。
仅保留这一条规则，其他规则注释掉。
```php
$route['default_controller'] = 'welcome';
```
这行代码的含义是，当打开 http://localhost 时，默认使用Welcome控制器。
噫，这里怎么只有控制器名没有方法名呢？
这时打开application/controller下的Welcome.php，会发现里面有一个index()方法，通过上面的注释可以了解到，访问
http://localhost/index.php/welcome
或
http://localhost/index.php/welcome/index
都会匹配到index()方法。
即当url中没有指明方法名称时，CI会自动寻找当前控制器类下的index()方法执行。

好了，上面都是官方的范例，现在轮到自己动手了。
情景设定为，从数据库中获取所有的优惠码显示在页面中。
新建路由规则，
```php
$route['codes'] = 'main/get';
```
访问 http://localhost/index.php/codes 时，即可使用Main控制类下的get()方法。

路由规则也可以使用正则限定参数，例
```php
$route['codes/([a-z]+)/(\d+)'] = 'main/$1/$2';
```
访问 http://localhost/index.php/codes/get/1 时，会定向到main类下的get()方法，多余的url参数会传入到函数中。

### 控制器
在目录application/controller下，新建文件Main.php。
控制器类文件名可根据编程规范命名，这里为了方便仅采用了首字母大写的规则。
打开文件，键入代码
```php
<?php
class Main extends CI_Controller {
 }
```
因为要从数据库中获取信息，所以可以把读取模型的代码写入到构造方法中，减少后面的代码量。
```php
public function __construct(){
        parent::__construct();
        $this->load->model('coupon_model');
    }
```
在get()方法中，直接使用模型类中的方法即可。
同时，把将要渲染模板的代码写入方法中，
```php
public function get(){
    $data['codes'] = $this->coupon->get_codes();
    $this->load->view('templates/header');
    $this->load->view('pages/content', $data);
    $this->load->view('templates/footer');
}
```
上面的view()方法会在<b>模板</b>章节介绍。

### 模型
数据库配置文件在目录application/config下，
打开database.php可以看相关配置参数，根据实际情况填写即可。
配置完成后，在目录application/models下，新建模型类文件Coupon_model.php,
打开文件输入以下代码即可完成配置，
```php
<?php
class Coupon_model extends CI_Model{
    function __construct(){
         $this->load->database();
    }
    function get_codes(){
         $query = $this->db->get('test');
         return $query->result_array();
    }
}
```
返回数据格式为数组。
这里用到的数据库信息与THINKPHP快速入门中一致。
这样，框架的模型配置已经完成了。

### 模板
这里介绍第三节提到的渲染模板。根据代码可以看出view()方法接收了两个参数，其中一个是模板文件的路径，另一个则是传入的变量。
模板文件在目录application/views下，其中有官方模板可以借用。
新建两个文件夹pages和templates，与代码相对应。
顾名思义，templates是模板，可以将html通用代码分为header.html和footer.html两个文件，如`<body>`标签以上的部分写到header内，`</body>`以下的部分写到footer内。引用时不需要写后缀名。
在pages文件夹内创建content.html，输入代码

```php
<h2>这里是优惠码:</h2>
<ul>
<?php foreach ($data as $date_item):?>
    <li><?php echo $date_item['id'].":".$date_item['code']; ?></li>
<?php endforeach; ?>
</ul>
```
这样的代码很是让人头疼。CI提供了模板解析类，使用更优美的模板语言渲染模板。当然这部分内容不在本文的范围内，具体内容可以参考官方文档。
当模板渲染时，会按照view()的顺序依次渲染。

现在，在浏览器地址栏输入  http://localhost/index.php/codes ，会看到从数据库中获取的优惠码信息。

0:SUFFLzS
1:hZxakCl
2:xD2gz0N
3:6WwLt7U
4:xLSxY1n
5:bn422Ka
6:55BaQ0C
7:Ui1Jc8P

### 总结
　　CodeIgniter的开发遵循了MVC的设计模式，这种将逻辑层与表现层分离的软件方法，让开发人员更专注于自己所属的工作，同时代码的维护也方便了很多。
　　个人非常喜欢CI框架，不仅是其简单自由高效的特性，更因为其语法风格。开发时配置明确，思路清晰，总之写起来很爽。