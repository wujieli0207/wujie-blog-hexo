---
title: webpack入门-入口与出口
date: 2021-12-02 07:27:44
tags: webpack
---
## 入口及基础配置
- webpack 该用哪个模块作为构建依赖图的开始
- 在 `entry` 配置入口，可以指定一个或多个入口
	```js
	module.exports = {
	  entry: './path/to/my/entry/file.js'
	};
	```
<!--more-->
- 单页面场景配置
	- 将应用入口和第三方库分离
	- 可以使用 `CommonsChunkPlugin`  从「应用程序 bundle」中提取 vendor 引用到 vendor bundle，把引用 vendor 的部分替换为 ，`__webpack_require__()` 调用
	```js
	const config = {
	  entry: {
		app: './src/app.js',
		vendors: './src/vendors.js'
	  }
	};
	```
- 多页面场景配置
	- 不同页面入口分开配置，进入新页面时独立加载 html 和对应引用
	- 可以使用 `CommonsChunkPlugin` 为每个页面间的应用程序共享代码创建 bundle，实现代码复用的效果
	```js
	const config = {
	  entry: {
		pageOne: './src/pageOne/index.js',
		pageTwo: './src/pageTwo/index.js',
		pageThree: './src/pageThree/index.js'
	  }
	};
	```

## 出口及基础配置
- webpack 输出 bundles 的目录及命名方式配置，默认为  `./dist`
- **可以指定多个入口，但只能有一个出口（output）**
- 在 `output` 字段配置：输出为一个对象，输出路径和文件名
	```js
	const path = require('path');

	module.exports = {
	  output: {
		path: path.resolve(__dirname, 'dist'),
		filename: 'my-first-webpack.bundle.js'
	  }
	};
	```
- 对于多文件入口，使用占位符保持文件名正确
	```js
	{
	  entry: {
		app: './src/app.js',
		search: './src/search.js'
	  },
	  output: {
		filename: '[name].js',
		path: __dirname + '/dist'
	  }
	}

	// 写入到硬盘：./dist/app.js, ./dist/search.js
	```

- 在编译时不知道最终输出文件的 `publicPath` 的情况下，`publicPath` 留空，并且在入口起点设置 `__webpack_public_path__`
	```js
	__webpack_public_path__ = myRuntimePublicPath

	// 剩余的应用程序入口
	```


