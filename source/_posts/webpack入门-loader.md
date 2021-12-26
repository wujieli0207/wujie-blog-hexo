---
title: webpack入门-loader
date: 2021-12-05 11:01:46
categories: 前端
tags: webpack
img: https://notesimgs.oss-cn-shanghai.aliyuncs.com/img/20211226142337.jpg
---
## loader 基础
- 将所有文件转换成 webpack 能够处理的模块，用于处理非 JS 文件
- 在 module.rule 配置，必须包含 test 和 use 属性
	- `test`： 标识出应该被对应的 loader 进行转换的某个或某些文件
	- `use`：进行转换时，应该使用哪个 loader
	```javascript
	const path = require('path');

	const config = {
	  module: {
		rules: [
		  { test: /\.css$/, use: 'css-loader' },
		  { test: /\.ts$/, use: 'ts-loader' }
		]
	  }
	};

	module.exports = config;
	```

- 使用 loader 的三种方式
	- 配置（推荐）：在 webpack.config.js 中配置
	- 内联：在每个 import 语句中显式指定 loader
		`import Styles from 'style-loader!css-loader?modules!./styles.css';`
	- CLI：在 shell 命令中指定它们
		`webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'`

- loader 特性
	- loader 支持链式传递，反向执行，loader 链中的第一个 loader 返回值给下一个 loader
	- loader 可同步也可异步
	- loader 可以在 Node.js 环境运行
	- loader 能够接受查询参数
	- loader 可以使用 `opotions` 对象进行配置
	- 可以将普通 npm 模块导出一个 loader