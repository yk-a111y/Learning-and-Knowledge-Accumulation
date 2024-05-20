## 三个阶段
bebel的转换过程可以分为三阶段：解析阶段（Parsing）、转换阶段（Transformation）、生成阶段（Code Generation）。其作用就是将源代码转换为浏览器可以识别的另一段源码。

详细过程如图：

parse =>  parser 把源码转成抽象语法树（AST）
transform => 遍历 AST，调用各种 transform 插件对 AST 进行增删改
generate => 转换后的 AST 打印成目标代码，并生成 sourcemap

![[基础 2024-02-13 20.03.24.excalidraw]]
### 简单实例：说明三个过程如何工作
首先，我们需要通过parse生成源代码的AST，该API由@babel/parse提供
```js
const ast = parse(sourceCode, {
	sourceType: 'unambiguous',
	plugin: 'jsx'
})
```
parse接收两个参数，sourceCode源码和options配置。源码为需要转换为AST的代码内容，options是转换过程中以什么方式parse的配置。

其次，将得到的AST使用@babel/traverse中的traverse方法遍历，这一阶段也是处理逻辑的核心。调用方式有两种：
```js
traverse(ast, {
	FunctionDecleration:
		enter(path, state) {}, // 进入节点时调用
		exit(path, state) {} // 离开节点时调用
})

traverse(ast, {
	// 仅指定一个函数时，enter阶段调用
	FunctionDecleration(path, state) {
		...
	}
})
```
FunctionDecleration的内部逻辑，即为visitor函数。vistor函数有两个参数，path记录了遍历到路径，state则是遍历过程中不同节点传递信息的机制。

通过traverse我们可以将AST修改为我们期望的样子

