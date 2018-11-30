---
title: 使用React+Three.js 封装一个三维地球
tags:
  - FE
  - Three.js
categories: 前端技术
date: 2017-09-23 17:43:15
---
良久没有写过博客了，最近忙的焦头烂额，忽略了博客，罪过罪过。今天补充一篇，前一段时间研究过的技术，使用React+Three.js 封装一个三维地球，支持鼠标的交互行为。其实也实现了对有坐标的json数据展示在地球上的功能，以后会有补充。

[github仓库地址](https://github.com/zrysmt/react-threejs-app):
> https://github.com/zrysmt/react-threejs-app

整体做完之后的效果图：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/react%2Bthreejs/1.jpg)
废话少说，直接上环境
## 1.环境
使用facebook给出的脚手架工具[create-react-app](https://github.com/facebookincubator/create-react-app).

```bash
npm install -g create-react-app

create-react-app react-threejs-app
cd react-threejs-app/
```
执行
```bash
npm start
```
浏览器会自动打开`localhost:3000`。

## 2.背景知识
Three.js简单来说就是封装了WebGL一些易用的API接口，我们知道只使用WebGL比较低效。具体的关于WebGL的技术给出两篇博客的入口,关于Three.js可以参考文章最后给出的参考阅读部分。
- [WebGL基础简明教程1-简介](https://zrysmt.github.io/2017/05/17/WebGL%E5%9F%BA%E7%A1%80%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B1-%E7%AE%80%E4%BB%8B/)
- [WebGL基础简明教程2-基础知识](https://zrysmt.github.io/2017/05/17/WebGL%E5%9F%BA%E7%A1%80%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B2-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/)

如果不是很了解WebGL技术也没有关系，不妨现在先看看Three.js创建模型的整体过程。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/react%2Bthreejs/2.png)

安装需要的库,`three`是Three.js的库，` three-orbitcontrols`用来支持鼠标的交互行为的库。
```bash
npm i three three-orbitcontrols --save
```

## 3.一步一个脚印
### 3.1 准备一张高清的世界地图
这里在github仓库中已经给出。
### 3.2 定义一个组件`ThreeMap`
在`ThreeMap.js`定义组件`ThreeMap`，并且创建改组件的样式`ThreeMap.css`。css定义三维地球的容器的宽度和高度。
```css
#WebGL-output{
	width: 100%;
	height: 700px;
}
```
并且该组件在`App.js`引用。
### 3.3 引入库和样式
```js
import './ThreeMap.css';
import React, { Component } from 'react';
import * as THREE from 'three';
import Orbitcontrols from 'three-orbitcontrols';
import Stats from './common/threejslibs/stats.min.js';
```
### 3.4 初始化方法入口和要渲染的虚拟DOM
```js
componentDidMount(){
	this.initThree();
}
```
要渲染的虚拟DOM设定好
```js
render(){
	return(
		<div id='WebGL-output'></div>
	)
}
```
### 3.4 initThree方法
- 创建场景

```js
let scene;
scene = new THREE.Scene();
```
- 创建Group

```js
let group;
group = new THREE.Group();
scene.add( group );
```
- 创建相机

```js
camera = new THREE.PerspectiveCamera( 60, width / height, 1, 2000 );
camera.position.x = -10;
camera.position.y = 15;
camera.position.z = 500;
camera.lookAt( scene.position );
```
- 相机作为`Orbitcontrols`的参数，支持鼠标交互

```js
let orbitControls = new Orbitcontrols(camera);
orbitControls.autoRotate = false;
```
- 添加光源:环境光和点光源

```js
let ambi = new THREE.AmbientLight(0x686868); //环境光
scene.add(ambi);
let spotLight = new THREE.DirectionalLight(0xffffff);  //点光源
spotLight.position.set(550, 100, 550);  
spotLight.intensity = 0.6;
scene.add(spotLight);
```
- 创建模型和材质

```js
let loader = new THREE.TextureLoader();
let planetTexture = require("./assets/imgs/planets/Earth.png");

loader.load( planetTexture, function ( texture ) {
	let geometry = new THREE.SphereGeometry( 200, 20, 20 );
	let material = new THREE.MeshBasicMaterial( { map: texture, overdraw: 0.5 } );
	let mesh = new THREE.Mesh( geometry, material );
	group.add( mesh );
} );
```
- 渲染

```js
let renderer;
renderer = new THREE.WebGLRenderer();
renderer.setClearColor( 0xffffff );
renderer.setPixelRatio( window.devicePixelRatio );
renderer.setSize( width, height );
container.appendChild( renderer.domElement );
```
- 增加监控的信息状态

```js
stats = new Stats();
container.appendChild( stats.dom );
```
**将以上封装到`init`函数中**
- 动态渲染，地球自转

```js
function animate() {
	requestAnimationFrame( animate );
	render();
	stats.update();
}
function render() {
	group.rotation.y -= 0.005;  //这行可以控制地球自转
	renderer.render( scene, camera );
}
```
调用的顺序是：
```js
init();
animate();
```
大功告成，一个可交互的三维地球就可以使用了。

## 4.`ThreeMap.js`整体代码
```js
import './ThreeMap.css';
import React, { Component } from 'react';
import * as THREE from 'three';
import Orbitcontrols from 'three-orbitcontrols';
import Stats from './common/threejslibs/stats.min.js';

class ThreeMap extends Component{
 	componentDidMount(){
		this.initThree();
	}
	initThree(){
		let stats;
		let camera, scene, renderer;
		let group;
		let container = document.getElementById('WebGL-output');
		let width = container.clientWidth,height = container.clientHeight;

		init();
		animate();

		function init() {
			scene = new THREE.Scene();
			group = new THREE.Group();
			scene.add( group );

			camera = new THREE.PerspectiveCamera( 60, width / height, 1, 2000 );
			camera.position.x = -10;
        	camera.position.y = 15;
			camera.position.z = 500;
			camera.lookAt( scene.position );
			
			//控制地球
			let orbitControls = new /*THREE.OrbitControls*/Orbitcontrols(camera);
        	orbitControls.autoRotate = false;
        	// let clock = new THREE.Clock();
        	//光源
        	let ambi = new THREE.AmbientLight(0x686868);
        	scene.add(ambi);

        	let spotLight = new THREE.DirectionalLight(0xffffff);
        	spotLight.position.set(550, 100, 550);
        	spotLight.intensity = 0.6;

        	scene.add(spotLight);
			// Texture
			let loader = new THREE.TextureLoader();
			let planetTexture = require("./assets/imgs/planets/Earth.png");

			loader.load( planetTexture, function ( texture ) {
				let geometry = new THREE.SphereGeometry( 200, 20, 20 );
				let material = new THREE.MeshBasicMaterial( { map: texture, overdraw: 0.5 } );
				let mesh = new THREE.Mesh( geometry, material );
				group.add( mesh );
			} );

			renderer = new THREE.WebGLRenderer();
			renderer.setClearColor( 0xffffff );
			renderer.setPixelRatio( window.devicePixelRatio );
			renderer.setSize( width, height );
			container.appendChild( renderer.domElement );
			stats = new Stats();
			container.appendChild( stats.dom );  //增加状态信息 

		}
		
		function animate() {
			requestAnimationFrame( animate );
			render();
			stats.update();
		}
		function render() {		
			group.rotation.y -= 0.005;  //这行可以控制地球自转
			renderer.render( scene, camera );
		}
	}
	render(){
		return(
			<div id='WebGL-output'></div>
		)
	}
}

export default ThreeMap;
```
**参考阅读：**
- [WebGL基础简明教程1-简介](https://zrysmt.github.io/2017/05/17/WebGL%E5%9F%BA%E7%A1%80%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B1-%E7%AE%80%E4%BB%8B/)
- [WebGL基础简明教程2-基础知识](https://zrysmt.github.io/2017/05/17/WebGL%E5%9F%BA%E7%A1%80%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B2-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/)
- [图解WebGL&Three.js工作原理](http://www.cnblogs.com/wanbo/p/6754066.html)
