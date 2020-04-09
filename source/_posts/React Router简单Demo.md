---
title: React Router简单Demo
date: 2018-04-07 13:39
tags: [前端, react, react-router]
---


### 简介
react router是使用react的时候首选的一个路由工具。
<!--more-->
### 安装
react router包含react-router,react-router-dom和react-router-native这三个包，分别是路由核心组件和浏览器端组件和native端组件，所以我们需要安装react-router-dom

```
npm install --save react-router-dom
```
安装后就可以直接使用了

https://codepen.io/pshrmn/pen/YZXZqM?editors=1010

这里有一个只有一个js文件的demo，我是根据这个demo来改成用多个文件的demo的

### 正式开始

##### index.js配置
首先你需要在index.js里面引入BrowserRouter组件，这个组件是作为路由器的选择。

你的选择可以有两种BrowserRouter和HashRouter

当存在服务区来管理动态请求时，需要使用<BrowserRouter>组件，而<HashRouter>被用于静态网站。

引入后index.js为

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './assets/css/index.css';
import App from './pages/App';
import registerServiceWorker from './registerServiceWorker';
import {BrowserRouter} from 'react-router-dom'

ReactDOM.render((
	<BrowserRouter>
		<App/>
	</BrowserRouter>
	), document.getElementById('root'));
registerServiceWorker();

```
##### 添加被路由的组件
添加两个测试用的组件内容随意 test和test2

```javascript
import React, { Component } from 'react';
import '../assets/css/App.css';

class App extends Component {
	render() {
		return (
			<div className="App">
				<p className="App-intro">
					this is test
				</p>
			</div>
		);
	}
}
export default App;

```

##### 添加路由的组件router.js
先上代码吧，很简单

```javascript
import React from 'react';
import test2 from './pages/Test2.js'
import test from './pages/Test.js'
import {Switch,Route} from 'react-router-dom'


const router = () => (
	<Switch>
		<Route exact path='/test' component={test}/>
		<Route path='/test2' component={test2}/>
	</Switch>
)

export default router;
```

首先引入react，不引入的话不会识别jsx语法

然后引入两个测试组件和react-router-dom

其中所有的路由需要用Switch来包括

然后所有的路由需要用Route组件来写

path属性是路由地址，component是地址的组件

这些就可以定义我们路由跳转的页面了

##### app.js中设置跳转

先看代码

```javascript
import React, { Component } from 'react';
import {Link} from 'react-router-dom'
import Router from './../router.js'

class App extends Component {
  render() {
    return (
      <div>
        <li><Link to='/test'>test</Link></li>
        <li><Link to='/test2'>test2</Link></li>
        <Router></Router>
      </div>
    );
  }
}

export default App;

```
引入react-router-dom的Link组件，然后引入我们编写好的Router

然后用Link去跳转到对应的路由地址

然后在Router组件里面展示当前路由的页面

Router是用来显示对于路由的页面的

### 总结
这里只是简单的路由的使用

总结一下就是只需要在入口处定义路由类型，然后设置路由的对应组件，然后再跳转到对应的路由就可以显示不同路由的组件了

#### 参考资料
https://segmentfault.com/a/1190000010174260#articleHeader16