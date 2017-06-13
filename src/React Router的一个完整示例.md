---
title: React Router的一个完整示例
tags:    
- React
- React Router   
categories: 前端技术
---
本博文提供一个单网页结构网页（SPA）使用React Router路由控制跳转的完整例子。
> 可以在我的[github](https://github.com/zrysmt/react-demo/tree/master/demo03) 中clone或者fork
  https://github.com/zrysmt/react-demo/tree/master/demo03

关于配置可以查看我之前的一篇博客：[一步一步进入React的世界（React+Webpack+ES6组合配置）](https://zrysmt.github.io/2016/10/31/%E4%B8%80%E6%AD%A5%E4%B8%80%E6%AD%A5%E8%BF%9B%E5%85%A5React%E7%9A%84%E4%B8%96%E7%95%8C%EF%BC%88React+Webpack+ES6%E7%BB%84%E5%90%88%EF%BC%89/)。
# 1.整个目录结构
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/react/1.png)
- build是编译后的文件夹
- src 放入源码
  + components组件
     + global 通用组件和SCSS
	 + ... 分模块
  + app.js入口
- index.html 

# 2.源码
关于源码可以在开头给出的github中找到详细的完整例子，这里就介绍重要的几个文件源码
记住要安装react-router

```bash
npm i react-router -S
```
## 2.1 index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Our Home,Our Heart</title>
    <meta name="viewport" content="width=device-width,initial-scale = 1.0,user-scalable=no">
</head>

<body>
    <div id="content">
    </div>
    <script src="build/bundle.js"></script>
</body>
</html>
```
## 2.2 入口文件app.js
关于react router的基础知识我们可以参考阮一峰老师的博客作为入门指导。

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import {Router,Route,IndexRoute,hashHistory} from 'react-router';

import './components/global/global.scss';

import Nav from './components/global/menu';
import Home from './components/home/home';
import Story from './components/story/story';


class App extends React.Component{
	render(){
		return(
			<div>	
				<Nav/>
				{this.props.children}
			</div>				
		)
	}
}

ReactDOM.render((
	<Router history={hashHistory}>
		<Route path="/" component={App}>
			<IndexRoute component={Home}/>
			<Route path="/Story" component={Story}/>
		</Route>
	</Router>
	),document.body
);
```
**简单解释下:**
组件App除了包含Nav组件，还应该包括主体内容
当使用index.html访问的时候，是在项目根目录下，这样会先加载APP组件，APP组件包含`{this.props.children}`，便会加载`<IndexRoute/>`里面定义的组件Home。用户访问'/'相当于：

```
<App>
  <Nav/>
  <Home/>
</App>
```

## 2.3 Nav组件
/components/global/menuLi.jsx
/components/global/menu.jsx

- 最小一块组件menuLi.jsx

```javascript
import React from 'react';
import {Link} from 'react-router';
class MenuLi extends React.Component{
	render(){
		let linkTo = this.props.name =="Home"?"/":"/"+this.props.name;
		return (
			<li>
				<Link to={linkTo}>
					{this.props.name}
				</Link>
			</li>
		);
	}
}
export default MenuLi;
```
`Link`组件用于取代`<a>`元素，生成一个链接，允许用户点击后跳转到另一个路由
- Nav组件 menu.jsx

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import MenuLi from './menuLi';
import './menu.scss';

let menuLis = ["Home","Story","Travel","TimeLine","Future"];
class MenuUl extends React.Component{
	render(){
		return(
			<ul>
			{
				menuLis.map(function(menuLi) {
				    return <MenuLi name={menuLi}/>
				})
			}
			</ul>
		);	
	}
} 
class Nav extends React.Component{
	render(){
		return(
			<nav>
				<div id="menu">
					<MenuUl/>			
				</div>
			</nav>
		)
	}
}
export default Nav;
```
## 2.4 Home组件
/components/home/home.jsx,示例比较简单

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import "./home.scss";
class Home extends React.Component{
	render(){
		return (
			<h5>这是home</h5>
		);
	}
}
export default Home;
```
## 2.5 Story组件

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import "./story.scss";

class Story extends React.Component{
	render(){
		return (
			<h5>这是story</h5>
		);
	}
}
export default Story;
```
其余几个组件不一一列出了

> 可以在我的[github](https://github.com/zrysmt/react-demo/tree/master/demo03) 中clone或者fork,查看完整的例子代码

参考阅读:

- [React Router 使用教程--阮一峰](http://www.ruanyifeng.com/blog/2016/05/react_router.html?utm_source=tool.lu)