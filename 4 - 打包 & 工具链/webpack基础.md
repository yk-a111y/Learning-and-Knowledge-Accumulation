### 【综述】如何看待Webpack
Webpack 本质上是一个函数，它接受一个配置信息（config）作为参数，执行后返回一个 compiler 对象，调用 compiler 对象中的 run 方法就会启动编译。run 方法接受一个回调，可以用来查看编译过程中的错误信息或编译信息。

【一个简单的例子】

![[webpack基础 2024-11-11 15.20.19.excalidraw]]
【核心思想】
- 第一步：根据webpack.config.js找到入口文件entry；
- 第二步：找到entry所依赖的模块，并收集关键信息：比如`路径、源代码、它所依赖的模块`等；
- 第三步：最终输出到硬盘中的文件（dist）： 包括 modules 对象、require 模版代码、入口执行文件等。

这一过程中，由于浏览器并不认识除 `html、js、css` 以外的文件格式，所以我们还需要对源文件进行转换 —— **`Loader 系统`**
### 基本概念
#### entry
打包时第一个被访问的源码文件，默认是src/index.js，可以通过配置文件指定。Webpack以入口文件为准，通过层层的依赖关系形成的依赖图来加载整个项目的依赖
#### output
打包之后输出的文件，默认是dist/main.js，也可通过配置更改。
```js
module.exports = {
	...
	output: {
		...
		clean: true // 打包时自动清理上次打包产物
	}
}
```
#### loader
Webpack只识别JS文件，对于非JS文件需要一个翻译官将文件编译为Js文件进行处理。xxx-loader：css-loader，style-loader，less-loader等。使用顺序为entry -> loader -> output，在打包文件之前运行。
#### plugin
在loader之外的其他扩展功能由Plugin实现，例如: 打包优化、资源管理、环境变量注入等。通常以xxx-webpack-plugin为后缀。plugins的运行贯穿了整个webpack的编译周期。

> loader和plugin本质上都是npm包

#### chunk
根据文件之间的依赖关系形成依赖树，组成依赖树的这些文件合称为一个chunk
#### mode
区分打包环境的关键字。
- development: 自动优化打包速度，添加调试过程中的辅助
- production: 自动优化打包结果
- none: 最原始的打包，不做任何额外处理

### 认识sourceMap
通过.map文件（本质上是JSON格式的文件，包含了映射信息）建立源代码与构建后代码之间的映射关系。
#### map文件
在webpack.config.js中配置devtool，打包时会生成xxx.js.map（bundle.js.map代表bundle.js的source-map）文件，其内容如下：
- version: source-map的版本(目前是3)
	> 最初source-map生成的文件大小是原始文件的10倍，version2减少50%体积，version3又减少50%体积。
- sources: 当前.map文件通过哪些文件转化而来
- names: 转换前变量和属性的名称
- mappings: 一串base64 VLQ（可变长度值）编码。记录映射关系。
- file: 打包后的文件（浏览器加载的文件），如：bundle.js。
- sourceContent: 转换前具体的代码信息。
- sourceRoot: sources对应的根目录。默认值为src
#### 如何配置devtool

![[Pasted image 20240213185325.png]]
### 认识babel
Babel是一个工具链，可将ES6+语法、TS语法、JSX语法转换为浏览器能够识别的普通ES5代码，本质上是编译器（将一种语言转化为另一种语言）的一种。其功能包括但不限于：语法转换、源代码转换、polyfill实现目标环境缺少的功能。

#### babel使用指南
与postcss一样，babel可以作为独立工具使用。只不过日常开发时，使用babel作为webpack的其中一环。

安装：@babel/core（核心代码）& @babel/cli（命令行工具）

使用命令：`npx babel ./src --out-dir ./build --plugins=${...}`
> src是指源文件目录、--out-dir后面指定输出的文件夹、plugin指定转换过程使用的插件。

比如：想要转换箭头函数代码为ES5
`npx babel src --out-dir dist -plugins=@babel/plugin-transform-arrow-functions`

如果使用大量plugin，这么执行命令很麻烦，可以安装babel预设的转换插件@babel/preset-env，该插件支持普遍的JS高级语法转换。
`npx babel src --out-dir dist --presets=@babel/preset-env`
> 常见的babel预设：preset-env、preset-react、preset-typescript
#### babel底层原理
[[babel原理综述]]
#### babel结合webpack
**安装依赖**
`npm i babel-loader @babel/core @babel/preset-env @babel/polyfill`
> @bebel/polyfill => preset-env只能转换基本语法，promise等高级语法需要靠polyfill转换。
> 
> polyfill虽然可以转换所有高级语法，但文件会变得冗余，体积扩大近百倍。故使用core-js进行按需转译 (官方不建议直接安装polyfill，而是core-js/stable--解决高级特性，regenerator-runtime/runtime--解决高级函数，因为@babel/polyfill体积很大)

webpack.config.js中配置
```js
module.exports = {
	module: {
		rules: [
			{
				test: /\.js$/,
				exclude: /node_modules/,
				use: {
					loader: 'babel-loader',
					options: {
						presets: [
							['@babel/preset-env'],
							{
								useBuiltIns: 'usage', // 按需加载
								corejs: 3, // 指定core-js版本
								targets: 'last 2 version' // 覆盖browserlist的配置
							}
						]
					}
				}
			}
		]
	}
}
```

![[Pasted image 20240214195556.png]]
![[Pasted image 20240214200720.png]]
![[Pasted image 20240214195828.png]]
![[Pasted image 20240214200153.png]]
#### babel编译React
#### babel编译Typescript
##### ts-loader
首先必须创建tsconfig.json文件，告知ts如何进行编译。`tsc --init`

安装ts-loader后，配置webpack如下：
```js
module.exports = {
	module: {
		rules: {
			...
			{
				test: /\.ts$/,
				use: 'ts-loader'
			}
		}
	}
}
```
##### babel-loader
一般开发选择使用babel-loader进行ts的编译。因为平常编写ts时会用到polyfill中的内容，使用babel-loader编译，集成性更好。而ts-loader仅支持将ts转化为js，此外还会对**类型进行校验**（babel-loader不对类型进行校验）。
```js
module.exports = {
	module: {
		rules: {
			...
			{
				test: /\.ts$/,
				use: {
					loader: 'babel-loader',
					options: {
						presets: [
							['@babel/preset-env', {
								useBuiltIns: 'usage', // 按需加载
								corejs: 3, // 指定core-js版本
								targets: 'last 2 version' // 覆盖browserlist
							}],
							['@babel/preset-typescript', {
								useBuiltIns: 'usage', // 按需加载
								corejs: 3, // 指定core-js版本
							}]
						]
					}
				}
			}
		}
	}
}
```
##### 最佳实践
当项目仅需要将ts代码转化为js，中间不涉及polyfill填充时，使用ts-loader编译。

当项目用到一些高级函数和特性，涉及到polyfill时，使用babel-loader进行代码转译，tsc进行类型校验。`tsc --noEmit (--watch)`（进行校验但不输出内容）
### webpack性能优化
#### 分包处理
项目中所有内容打包到一个包：
- 难以管理，分不清运行时代码是哪部分。
- 资源过大，首屏加载速度慢。

Code Splitting: 将项目中不同部分的代码放入不同bundle中，之后就可以按需加载或者并行加载。这一过程，可以通过代码加载的优先级提升加载性能。常见的Code Splitting有3种形式：多入口起点、动态导入和SplitChunks去重分离代码。
##### 多入口起点
配置entry多入口
```js
entry: {
	index: {
	  import: './src/index.js',
	  dependOn: 'shared'
	},
	main: {
	  import: './src/main.js',
	  dependOn: 'shared'
	},
	// 多包之间共享的包
	shared: ['axios']
}
```
output通过占位符实现输出多个打包文件
```js
output: {
    path: path.resolve(__dirname, './build'),
    // placeholder
    filename: '[name]-bundle.js',
    clean: true
},
```
##### 动态导入
两种方式实现：ECMAScript的import方法和require.ensure，目前推荐import的方法。

首屏只加载首屏所需资源，切换路由时，才将对应资源请求到本地。
```js
btn1.onclick = function() {
	import(/* webpackChunkName: "about" */'./router/about').then(res => {
		res.about()
		res.default()
	})
}

btn2.onclick = function() {
	import(/* webpackChunkName: "category"" */'./router/category"')
}
```
对于分包后的命名，可以在output中如下设置：
```js
output: {
    clean: true,
    path: path.resolve(__dirname, './build'),
    // placeholder
    filename: '[name]-bundle.js',
    // 单独针对分包的文件进行命名
    chunkFilename: '[name]_chunk.js' // 其name对应webpackChunkName
  },
```
##### SplitChunks
基本概念：chunk是webpack的区块概念，有三种方式可以产生。
- 每个entry产生一个区块
- 每个动态加载 => import(./xxx)产生一个区块
- 代码分割会产生区块
基于SplitChunksPlugin实现的分包功能，默认只对异步请求接收到的文件做chunks打包。
```js
module.exports = {
	...,
	optimization: {
		// 生成chunkID的算法 => development默认为named; production默认为deterministic
		chunkIDS: "deterministic"
		// 分包的配置
		splitChunks: {
			chunks: 'all', // async 表示只对异步请求拿到的数据分包；initial 表示只对非动态加载的数据分包；防止在配置路由懒加载是，命中对应的懒加载包使其不生效
			maxSize: 20000, // 大于指定大小(20KB) => 继续拆包
			minSize: 10000, // 将包拆分为不小于minSize的包
		}
		// 自己配置拆包后的内容，如：哪些拆包，哪些不拆；拆后的包叫什么名字
		cacheGroups: {
			utils: {
				test: /utils/,
				filename: "[id]_utils.js"
			}
			// node_modules 中的内容打包，进行如下配置
			vendors: {
				test: /node_modules/,
				// webpack可以根据id（文件内容不变，id也不变），识别该包是否可以复用，下次打包的时候不做处理
				filename: "[id]_vendors.js" // mode 为 development的环境下 id 特别长
			}
		}
	}
}
```
splitChunks的最佳实践 -- 细粒度代码分割
```js
module.exports = {
	mode: 'production',
	optimization: {
		splitChunks: {
			maxInitialRequests: MAX_REQUEST_NUM,
			maxAsyncRequests: MAX_REQUEST_NUM,
			minSize: MIN_LIB_CHUNK_SIZE,
			cacheGroups: {
				defaultVendors: false,
		        default: false,
		        // node_modules中的包打包进lib
		        lib: {
		          chunks: 'all',
		          test(module) {
		            return (
		              module.size() > MIN_LIB_CHUNK_SIZE &&
		              /node_modules[/\]/.test(module.identifier())
		            );
		          },
		          name(module) {
		            const hash = crypto.createHash('sha1');
		            // ...
		            return `lib.${hash.digest('hex').substring(0, 8)}`;
		          },
		          priority: 3,
		          minChunks: 1,
		          reuseExistingChunk: true,
		        },
		        // 被多个模块共用的包打包进shared
		        shared: {
		          chunks: 'all',
		          name(module, chunks) {
		            return `shared.${crypto
		              .createHash('sha1')
		              .update(
		                chunks.reduce((acc, chunk) => {
		                  return acc + chunk.name;
		                }, ''),
		              )
		              .digest('hex')
		              .substring(0, 8)}${isModuleCSS(module) ? '.CSS' : ''}`;
		          },
		          priority: 1,
		          minChunks: 2, // 被两个以上模块共用的包
		          reuseExistingChunk: true,
		        },
			}
		}
	}
}
```
其中
- lib缓存组用于把体积较大的NPM包模块，拆分为独立区块，产生独立产物文件，从而在多次打包发版、更新哈希版本号文件名的同时，避免让用户再次下载这些大体积模块，提高缓存命中率，减少资源下载体积。
- shared文件保持独立的哈希版本号文件名，以便于在页面A和页面B之间复用，既能减少2个页面的JS体积，又能提高多次打包发版后的缓存命中率
#### prefetch 与 preload
prefetch: 在父chunk加载结束后的浏览器空闲时间加载，下载内容作用于对未来某一时刻的资源请求的立即响应。更推荐使用，因为不占用当前主包的下载、执行时间。
preload：在父chunk加载时，并行下载。

具体做法：
webpack魔法注释
```js
btn1.onclick = function() {
	import(
		/* webpackChunkName: "about" */
		/* webpackPrefetch: true */
	'./router/about').then(res => {
		res.about()
		res.default()
	})
}
```
#### CDN服务器
通过互联网，利用最靠近用户的服务器向用户发送资源，尽量避免网络请求中途的不可控性。CDN服务器大致分为三层: 源服务器、父级服务器、边缘服务器。应用了CDN网络的客户端发送请求时，首先检查边缘节点有无资源，其次是父级服务器，一层一层查找，直至请求到资源。资源 **(CDN主要针对项目中的静态资源如图片等)** 在响应时，也会在各级CDN做缓存方便下次请求的快速响应。

开发的主要用途：静态资源和第三方包利用CDN加速下载。

**应用：**
对于现有的CDN服务器，直接在output中设置publicPath，这样html文件的资源都会加上这个服务器前缀，请求CDN
```js
output: {
	filename: 'js/[name].[contenthash:8].bundle.js',
	path: path.resolve(__dirname, 'dist'),
	// CDN服务器地址
	publicPath: 'https://yksown/cdn'
}
```
对于一些配置了CDN的npm包，打包时将它们排除，然后在index.html文件中设置script标签src的属性为其CDN的URL进行引入
```js
module.exports = {
  entry: ...,
  output: ...,
  externals: {
    // 包名: '从CDN地址请求下来的js中提供的对应名称'
    lodash: '_',
    react: 'React'
  }
}
```

```html
<script src="lodash的CDN服务器"></script>
<script src="React的CDN服务器"></script>
```
#### MiniCssExtractPlugin
功能：打包时将CSS提取到一个共同的JS文件中。

前置loader：
- css-loader: 将css转为JS文件
- MiniCssExtractPlugin.loader => 生产阶段（生产是提取css到公共js文件中，故不需要挂载style标签）
- style-laoder: 将包含css内容的JS文件挂载到style标签中 => 开发阶段

应用：

```js
module.exports = {
	plugins: [
		...,
		new MiniCssExtractPlugin({
			filename: '[name].css',
			chunkFilename: '[name].css' // 针对动态导入的css => import('./xxx.css')
		})
	]
}
```
#### Terser
Terser: 集JS解释、Mangler/Compressor为一体的工具集。默认情况下，bundle.js是没有任何压缩的，是webpack底层使用TerserPlugin实现的压缩效果。

单独安装Terser工具：`npm i terser -G/D`
命令行中使用：`terser [input_filename] [options]`
如：`npx terser abc.js -o abc.min.js -c -m`(-c: compress; -m: mangle)

webpack中配置Terser
mode为production是，webpack会自动开启TerserPlugin，如果对其默认配置不满意，可以自定义来覆盖默认配置。
CSSMinizerPlugin: 压缩css代码内容 `npm i css-minimizer-webpack-plugin`
```js
module.exports = {
	optimization: {
		...,
		minize: true, // 告知webpack使用TerserPlugin或其他代码压缩插件
		minizer: [
			new TerserPlugin ({
				extractComments: false, // 注释抽取到一个单独文件中
				parallel: true, // 压缩的并行数量，默认值为true。默认数量为(进程数 - 1)
				// 对插件的配置
				terserOptions: {
					compress: {
						arguments: true // 函数中arguments[0]等转化为形参
						unused: false // 未被使用的代码不进行删除；默认为true（删除无用代码）
					},
					mangle: true,
					toplevel: false,
					keep_fnames: true // 压缩时，保持原函数名字
				}
			}),
			// 压缩CSS代码
			new CSSMinimizerPlugin({
				parallel: true
			})
		]
	}
}
```

#### Tree Shaking
TreeShaking 的核心在于对代码的`静态分析`，即在不执行程序的情况下，通过分析源代码或编译后的代码来理解程序行为、发现问题或提取信息；首先，TreeShaking会进行`代码 -> AST`的转换，之后分析转换后的AST，移除相关的无用代码。

移除的规则：如果是`纯函数`的导入，未被使用会被移除；如果是`副作用函数`的导入（如：console.log在控制台打印），该模块会被视为有副作用而得以保留。

webpack实现JS的Tree Shaking的两种方案：usedExports 和sideEffects。实现CSS TreeShaking的方案：purgeCss插件。

##### usedExports
设置usedExports为true，webpack会帮我们分析模块中哪些函数未被使用，从而删除这些无用代码。仅设置该属性，打包产物内会将无用代码标记（即只在打包产物中用注释标记哪些代码未被使用），还需要与terser联动，进行删除。
```js
module.exports = {
	...,
	optimization: {
		usedExports: true
	}
}
```
该配置需要与terserOptions中的compress的`unused = true`属性配合使用，不然webpack只是不导出无用代码，打包内容还是会包含无用代码。
##### sideEffects -- 副作用
sideEffects用于告知webpack compiler哪些模块是有副作用的：副作用是指某模块内的代码有执行一些特殊任务，不能仅仅通过export来判断这段代码的意义。

package.json中配置`sideEffects: false`表示所有文件都没有副作用，在treeshaking的基础上，不止会删除代码，无用的文件也会删除（TreeShaking时为防止文件内有特殊操作，通常会保留文件）。

sideEffects也可配置哪些文件含有副作用，打包的时候要保留。
```js
{
	sideEffects: [
		"*.css", // 所有css文件
		"./src/test.js" // 指定路径的文件
	]
}
```
##### CSS的TreeShaking -- Purgecss
```js
const glob = require('glob');
const { PurgeCssPlugin } = require('purgescc-webpack-plugin');

module.exports = {
	plugins: [
		// purgeCss也可以对less文件进行处理，所以它是对打包后的css做TreeShaking操作
		
		new PurgeCssPlugin({
			// 检测哪些目录下的内容
			paths: glob.sync(`${path.resolve(__dirname, '../src')}/**/*`, {nodir: true}),
			// 默认情况下，purgeCss会将html标签的样式移出，safeList可以设置安全名单
			safeList: function () {
				return {
					standard: ["html"]
				}
			}
		})
	]
}
```
#### Scope Hoisting
webpack打包时会有很多不同的函数作用域，作用域提升将其全部提升到最外层，以减小体积，这个操作在mode为production时是默认的。
```js
module.exports = {
	plugins: [
		new webpack.optimize.ModuleConcatenationPlugin()
	]
}
```
#### 文件压缩
不同于Terser，文件压缩是基于某种压缩算法，将HTML文件压缩成指定格式以减小体积。
`npm i compression-webpack-plugin -D`

```js
new CompressionPlugin({
	test: /\.(css|js)$/,
	minRatio: 0.7, // 至少压缩比例
	algorithm: "gzip", // 压缩算法
})
```
#### 模块联邦

### 源码相关

从一段简单的代码开始 => webpack执行的入口
```js
const webpack = require('../lib/webpack');
const config = require('./webpack.config'); // 我们写的配置信息

// 创建compiler对象
const compiler = webpack(config);

// 执行run方法，对代码编译和打包
compiler.run((err, stats) => {
	if (err) {
		console.log(err);
	} else {
		console.log(stats);
	}
})
```
webpack方法中，传入config，内部通过createCompiler创建compiler对象。创建过程中会注册config文件中plugins数组内设置的插件。
![[Pasted image 20240308132450.png]]