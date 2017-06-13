---
title: WebGL基础简明教程1-简介
tags:
  - FE
  - WebGL
categories: 前端技术
date: 2017-05-17 13:14:42
---
我也是个初学WebGL的人，这部分的内容是我在看完《WebGL编程指南》一书后的精简教程。看完之后我对三维世界重建了一些观念，这篇文章是尽量在有限的内容中，一下介绍几个重要的基本的概念，后面我会分几篇再详细介绍几个重要的概念。

WebGL是利用HTML5的canvas绘制和渲染三维图形，再现代的浏览器中均支持。WebGL是从OpenGL ES中继承过来的。

代码存储在我的[GitHub](https://github.com/zrysmt/data-viz/tree/master/webgl/demo)中。
> https://github.com/zrysmt/data-viz/tree/master/webgl/demo

首先我们来绘制一个二维的实例，点击的时候绘制一个点。
示例程序:[https://zrysmt.github.io/demo/webgl-demo/demo/0-simple.html](https://zrysmt.github.io/demo/webgl-demo/demo/0-simple.html).

# 1.WebGL二维：一次绘制一个点

## html片段

注意WebGL canvas的坐标（右，z轴垂直屏幕向外）和二维canvas（左）不一样。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webgl/1-1.jpg)
html片段很简单，我们使用就是canvas元素。
```html
<body onload="main()">
    <canvas id="webgl" width="400" height="400">
        Please use a browser that supports "canvas"
    </canvas>
</body>
```
## WebGL执行流程

![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webgl/1-2.jpg)
## 从main函数开始

```javascript
function main() {
    var canvas = document.getElementById('webgl');
    //初始化WebGL
    var gl = getWebGLContext(canvas);
    if (!gl) {
        console.log('Failed to get the rendering context for WebGL');
        return;
    }
    // 初始化着色器
    if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
        console.log('Failed to intialize shaders.');
        return;
    }
    // 获取a_Position的存储位置
    var a_Position = gl.getAttribLocation(gl.program, 'a_Position');
    if (a_Position < 0) {
        console.log('Failed to get the storage location of a_Position');
        return;
    }
    // 注册鼠标点击事件
    canvas.onmousedown = function(ev) {
        click(ev, gl, canvas, a_Position);
    };

    // 设置<canvas>背景色
    gl.clearColor(0.0, 0.0, 0.0, 1.0);

    // 清除<canvas>
    gl.clear(gl.COLOR_BUFFER_BIT);
}
```
## 顶点着色器

用来描述顶点的特性（如位置、颜色等）
```javascript
var VSHADER_SOURCE =
  'attribute vec4 a_Position;\n' +
  'void main() {\n' +
  '  gl_Position = a_Position;\n' +
  '  gl_PointSize = 10.0;\n' +
  '}\n';
```
是一种类似C的语言。a_Position是一个attribute变量，vec4表示有四个浮点数组成的矢量。
## 片元着色器

进行逐片处理的过程如光照
```javascript
var FSHADER_SOURCE =
   'void main() {\n' +
   '  gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);\n' +
   '}\n';
```
## 使用顶点着色器和片元着色器

![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webgl/1-3.jpg)

## 初始化着色器

这部分的代码也是通用的，流程如下,具体代码我们在我的github中查看
```
 * 1.创建着色器对象(gl.createShader())
 * 2.向着色器对象中填充着色器程序的源代码(gl.shaderSource())
 * 3.编译着色器(gl.compileShader())
 * 4.创建程序对象(gl.createProgram())
 * 5.为程序对象分配着色器(gl.attachShader())
 * 6.连接程序对象(gl.linkProgram())
 * 7.使用程序对象(gl.useProgram())
```
## 注册鼠标点击事件

注册鼠标事件，然后对坐标进行处理
```javascript
 canvas.onmousedown = function(ev) {
        click(ev, gl, canvas, a_Position);
 };
```
注意要转化为WebGL的坐标。
```javascript
function click(ev, gl, canvas, a_Position) {
    var x = ev.clientX; // 鼠标的x坐标
    var y = ev.clientY; // 鼠标的y坐标
    var rect = ev.target.getBoundingClientRect();

    x = ((x - rect.left) - canvas.width / 2) / (canvas.width / 2); //处理后相得canvas的x坐标
    y = (canvas.height / 2 - (y - rect.top)) / (canvas.height / 2); //处理后相得canvas的y坐标
    g_points.push(x);
    g_points.push(y);

    gl.clear(gl.COLOR_BUFFER_BIT);

    var len = g_points.length;
    for (var i = 0; i < len; i += 2) {
        // 将顶点位置传给attribute变量a_Position
        gl.vertexAttrib3f(a_Position, g_points[i], g_points[i + 1], 0.0);
        // 绘制
        gl.drawArrays(gl.POINTS, 0, 1);
    }
}
```
## 绘制


```javascript
gl.drawArrays(gl.POINTS, 0, 1);
```
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webgl/1-4.jpg)
指定第一个参数可以绘制线或三角形，具体的这几个有什么意义，可以查看[github](https://github.com/zrysmt/data-viz/tree/master/webgl/demo)中示例的源码。

# 2.WebGL二维：绘制多个点

一次性的将全部的点传给顶点着色器，这个时候就需要用到了**缓冲对象**。
我们以绘制个三角形为例（一次性至少传入三个点），示例程序：[https://zrysmt.github.io/demo/webgl-demo/ch03/HelloTriangle.html](https://zrysmt.github.io/demo/webgl-demo/ch03/HelloTriangle.html)。
效果如下所示：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webgl/1-5.png)
对应的源码位置：[https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/HelloTriangle.js](https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/HelloTriangle.js)

## 创建缓冲区对象

```javascript
function initVertexBuffers(gl) {
  var vertices = new Float32Array([
    0, 0.5,   -0.5, -0.5,   0.5, -0.5
  ]);
  var n = 3; // The number of vertices

  // 创建缓冲区对象
  var vertexBuffer = gl.createBuffer();
  if (!vertexBuffer) {
    console.log('Failed to create the buffer object');
    return -1;
  }

  // 绑定
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  // 写入数据
  gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);

  var a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  if (a_Position < 0) {
    console.log('Failed to get the storage location of a_Position');
    return -1;
  }
  // 缓冲区对象传给a_Position变量
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);

  // 启用
  gl.enableVertexAttribArray(a_Position);
 //顶点个数
  return n;
}
```

# 3.WebGL二维-变换与动画

## 变换

学过线性代数的都知道矢量的变化是可以通过矩阵（4 X 4的，可以容纳下三种变化）完成的。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webgl/1-6.jpg)

我们来看顶点着色器代码
```javascript
var VSHADER_SOURCE =
  'attribute vec4 a_Position;\n' +
  'uniform mat4 u_xformMatrix;\n' +
  'void main() {\n' +
  '  gl_Position = u_xformMatrix * a_Position;\n' +
  '}\n';
```
u_xformMatrix就是变化的矩阵。
旋转示例：[https://zrysmt.github.io/demo/webgl-demo/ch03/RotatedTriangle_Matrix.html](https://zrysmt.github.io/demo/webgl-demo/ch03/RotatedTriangle_Matrix.html)
旋转示例源码：[https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/RotatedTriangle_Matrix.js](https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/RotatedTriangle_Matrix.js)
缩放示例：[https://zrysmt.github.io/demo/webgl-demo/ch03/ScaledTriangle_Matrix.html](https://zrysmt.github.io/demo/webgl-demo/ch03/ScaledTriangle_Matrix.html)
缩放示例源码：[https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/ScaledTriangle_Matrix.js](https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/ScaledTriangle_Matrix.js)
平移示例：[https://zrysmt.github.io/demo/webgl-demo/ch03/TranslatedTriangle.html](https://zrysmt.github.io/demo/webgl-demo/ch03/TranslatedTriangle.html)
平移示例源码：[https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/TranslatedTriangle.js](https://github.com/zrysmt/data-viz/blob/master/webgl/ch03/TranslatedTriangle.js)

**注意：**以后关于矩阵的运算我们使用源码提供的库。旋转使用`setRotate`,`rotate`;平移使用`setTranslate`，`translate`;缩放使用`setScale`,`scale`.
这部分的使用
```javascript
var formatMatrix = new Matrix4();
    formatMatrix.setRotate(ANGLE,0, 0, 1); //绕z轴旋转ANGLE度数
    formatMatrix.translate(Tx,Ty,Tz);   //x,y,z轴上平移
```
第一个都要是带z的方法，后面的都不带即可。

> [https://github.com/zrysmt/data-viz/blob/master/webgl/lib/cuon-matrix.js](https://github.com/zrysmt/data-viz/blob/master/webgl/lib/cuon-matrix.js)

## 动画

动画的原理是使用HTML5 requestAnimationFrame()方法重绘WebGL图形。
示例：[https://zrysmt.github.io/demo/webgl-demo/ch04/RotatingTriangle_withButtons.html](https://zrysmt.github.io/demo/webgl-demo/ch04/RotatingTriangle_withButtons.html)
示例源码：[https://github.com/zrysmt/data-viz/blob/master/webgl/ch04/RotatingTriangle_withButtons.js](https://github.com/zrysmt/data-viz/blob/master/webgl/ch04/RotatingTriangle_withButtons.js)

# 4.WebGL二维-颜色与纹理

## 颜色

示例：[https://zrysmt.github.io/demo/webgl-demo/ch05/ColoredTriangle.html](https://zrysmt.github.io/demo/webgl-demo/ch05/ColoredTriangle.html)
示例源码：[https://github.com/zrysmt/data-viz/blob/master/webgl/ch05/ColoredTriangle.js](https://github.com/zrysmt/data-viz/blob/master/webgl/ch05/ColoredTriangle.js)

**大概流程**是这样的：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webgl/1-7.jpg)
顶点坐标==>图形装配==>光栅化==>执行片元着色器
```
  var verticesColors = new Float32Array([
    //  坐标x,坐标y，颜色r，g，b
     0.0,  0.5,  1.0,  0.0,  0.0, 
    -0.5, -0.5,  0.0,  1.0,  0.0, 
     0.5, -0.5,  0.0,  0.0,  1.0, 
  ]);
```
initVertexBuffers函数对整个矩阵的处理,位置和颜色分别分配给a_Position和a_Color
```
 gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, FSIZE * 5, 0);
 gl.vertexAttribPointer(a_Color, 3, gl.FLOAT, false, FSIZE * 5, FSIZE * 2);
```
```javascript
function initVertexBuffers(gl) {
  var verticesColors = new Float32Array([
    // Vertex coordinates and color
     0.0,  0.5,  1.0,  0.0,  0.0, 
    -0.5, -0.5,  0.0,  1.0,  0.0, 
     0.5, -0.5,  0.0,  0.0,  1.0, 
  ]);
  var n = 3;

  // Create a buffer object
  var vertexColorBuffer = gl.createBuffer();  
  if (!vertexColorBuffer) {
    console.log('Failed to create the buffer object');
    return false;
  }

  // Bind the buffer object to target
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexColorBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, verticesColors, gl.STATIC_DRAW);

  var FSIZE = verticesColors.BYTES_PER_ELEMENT;
  //Get the storage location of a_Position, assign and enable buffer
  var a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  if (a_Position < 0) {
    console.log('Failed to get the storage location of a_Position');
    return -1;
  }
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, FSIZE * 5, 0);
  gl.enableVertexAttribArray(a_Position);  // Enable the assignment of the buffer object

  // Get the storage location of a_Position, assign buffer and enable
  var a_Color = gl.getAttribLocation(gl.program, 'a_Color');
  if(a_Color < 0) {
    console.log('Failed to get the storage location of a_Color');
    return -1;
  }
  gl.vertexAttribPointer(a_Color, 3, gl.FLOAT, false, FSIZE * 5, FSIZE * 2);
  gl.enableVertexAttribArray(a_Color);  // Enable the assignment of the buffer object

  // Unbind the buffer object
  gl.bindBuffer(gl.ARRAY_BUFFER, null);

  return n;
}
```
## 纹理

示例：[https://zrysmt.github.io/demo/webgl-demo/ch05/TexturedQuad.html](https://zrysmt.github.io/demo/webgl-demo/ch05/TexturedQuad.html)
示例源码：[https://github.com/zrysmt/data-viz/blob/master/webgl/ch05/TexturedQuad.js](https://github.com/zrysmt/data-viz/blob/master/webgl/ch05/TexturedQuad.js)

```javascript
var image = new Image();  // Create the image object
  if (!image) {
    console.log('Failed to create the image object');
    return false;
  }
  // Register the event handler to be called on loading an image
  image.onload = function(){ loadTexture(gl, n, texture, u_Sampler, image); };
  // Tell the browser to load an image
  image.src = '../resources/sky.jpg';
```
逻辑函数
```javascript
function loadTexture(gl, n, texture, u_Sampler, image) {
  gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, 1); // Flip the image's y axis
  // Enable texture unit0
  gl.activeTexture(gl.TEXTURE0);
  // Bind the texture object to the target
  gl.bindTexture(gl.TEXTURE_2D, texture);

  // Set the texture parameters
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
  // Set the texture image
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGB, gl.RGB, gl.UNSIGNED_BYTE, image);
  
  // Set the texture unit 0 to the sampler
  gl.uniform1i(u_Sampler, 0);
  
  gl.clear(gl.COLOR_BUFFER_BIT);   // Clear <canvas>

  gl.drawArrays(gl.TRIANGLE_STRIP, 0, n); // Draw the rectangle
}
```

三维单独写一篇介绍。

**参考阅读：**
- [WebGL权威指南]()
- [webgl开源三维引擎的选择](http://blog.csdn.net/lh1162810317/article/details/50827948)
- [MDN-WebGl API](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API)
- [MDN-WebGL 中文教程](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial) 

