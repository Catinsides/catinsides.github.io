---
title: Threejs读取STL文件
date: 2016-11-26 17:42:46
tags: [Hello world,Javascript]
toc: true
description: Threejs是使用JavaScript编写的，运行在浏览器中的3D引擎。
---
>Threejs是使用JavaScript编写的，运行在浏览器中的3D引擎。

　　当初想在我的小项目中添加三维模型展示的功能，于是找到了这个js编写的第三方库。它可以方便的读取图形文件加载到浏览器中，并且提供各式小工具完成交互功能。
　　我在工作中使用的三维软件是Solidworks，可以方便的导出STL格式的文件，这也是通常在3D打印中使用的格式。百度上搜到的threejs教程比较丰富，然而偏偏没有读取STL文件的方法。官方文档中提供了各种Loaders用于加载各种格式的图形文件，然而偏偏也没有读取STL文件的方法……
　　最终神奇的在官方提供的代码包中([github][1]),找到了示例代码。分析代码，结合官方文档生肉，功夫不负有心人，总算是把生肉炒成了熟肉。
　　Talk is cheap,show me the code.

### 添加画布
其实就是在`<body>`标签内添加一个`<div>`,如下
```html
<div id="output"></div>
```
这个区域就是即将插入三维模型的地方，添加id属性方便获取。

### Threejs
本代码中的功能需要引入官方包中的three.min.js, STLLoader.js和OrbitControls.js.
#### 场景
初始化场景类
```js
var scene = new THREE.Scene();
```
为了便于观察，在场景中放置一个三坐标
```js
var axes = new THREE.AxisHelper(20);
scene.add(axes);
```
需要在场景添加其他物件时，使用scene.add()即可，就是这么简单~

#### 照相机
这里的照相机可不是生活中的指示代词，而是图形学中一种抽象概念。它定义了三维空间到二维屏幕的投影方式，照相机又分为正交投影照相机和透视投影照相机。本代码中使用的是透视投影照相机，至于为什么使用这个方式，有兴趣的同学可以wiki两种照相机的区别。
初始化照相机

```js
var camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.x = 150;
camera.position.y = 50;
camera.position.z = 150;
camera.lookAt(scene.position);
```

第一部分，PerspectiveCamera()接收四个参数：

* 45，为照相机竖直方向张角（角度制）；
* window.innerWidth / window.innerHeight,照相机水平与竖直方向长度的比值，取三维模型渲染区域的横纵比例；
* 0.1, 1000，最近点和最远点。

这些参数决定模型投影的形状与大小，一般情况下默认即可。

第二部分，设定照相机的位置，这些数值可根据模型大小进行调整。
第三部分，使照相机指向原点。

#### 读取与渲染
初始化类

```js
var webGLRenderer = new THREE.WebGLRenderer();
```
设定背景颜色与范围，并将渲染结果插入到页面中

```js
webGLRenderer.setClearColor(0xEEEEEE);
webGLRenderer.setSize(window.innerWidth, window.innerHeight);
document.getElementById("output").appendChild(webGLRenderer.domElement);
```
初始化类，并读取模型文件

```js
var loader = new THREE.STLLoader();
loader.load("sample.stl", function (geometry) {
    var mat = new THREE.MeshNormalMaterial();
    mesh = new THREE.Mesh(geometry, mat);
    mesh.scale.set(0.3, 0.3, 0.3);
    scene.add(mesh);
});
```
在匿名函数中，使用了内置的MeshNormalMaterial()材质，颜色分明便于观察。设定材质参数，添加到场景中。


#### 交互
既然是浏览三维模型，所以模型必须能够随心所欲的从各个角度观看。
所以浏览器中的模型必须能够使用鼠标转动。官方包中提供了这一功能。
首先，初始化类

```js
controls = new THREE.OrbitControls( camera, webGLRenderer.domElement );
controls.enableDamping = true;
controls.dampingFactor = 0.25;
controls.enableZoom = true;
```
其实每转动一次，浏览器都在重新绘制模型，所以参数中引入了照相机与渲染类。
下面几个语句是使能拖动和缩放，默认即可。

<b>这里是重点！</b>
以上代码都完成后，我发现模型在浏览器中并不能动！
为什么不动呢？这曾经是困扰了我很长时间的问题。
经过各种的google和stackoverflow后，我才找到了解决方法！
那就是requestAnimationFrame ()，通过递归调用更新画面，使画面动起来。
直接上代码

```js
render();
function render() {
            requestAnimationFrame(render);
            controls.update();
            webGLRenderer.render(scene, camera);
       }
```

代码完成，全部代码以及预览效果可以到我的Repositories中查看。

### 总结
　　通过以上配置就可以将STL图形插入到网页中去了，并且可以使用鼠标拖动和缩放。
代码中的```<div id="output">```可以任意大小，任意位置。STL文件在网上能够搜到各种各样的模型，替换代码中的文件名即可。
　　如果想自己制作STL文件，推荐软件SketchUp.
　　在使用chrome浏览器时，会提示跨域错误，不能加载模型。此时右击浏览器图标->属性，在属性目标后（引号外）添加 --allow-file-access-from-files.
　　FireFox无此问题。

[1]:https://github.com/mrdoob/three.js  "github"
