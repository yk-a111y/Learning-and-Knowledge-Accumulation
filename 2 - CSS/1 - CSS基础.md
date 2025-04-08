## 盒模型
浏览器渲染时，将元素表示为一个个矩形盒子，由四部分组成: content、padding、border、margin；content为实际内容，padding为内边距，控制内容在盒子内的位置，border为边框，margin为外边距控制不同盒子间的位置关系。
### 标准盒模型
`box-sizing: content-box`
为浏览器默认的盒子模型。盒子总宽度/高度 = width/height + padding + border + margin。width/height只是内容高度，并不包含padding和border。
### 怪异盒模型
`box-sizing: border-box`
盒子总宽度/高度 = width/height + margin。即盒子的内容宽高包括了padding和border在内。

## flexbox弹性盒模型
由（`Flex Container`容器/`Flex item`项目成员）构成。flex布局的元素称为`Flex Container`容器，它的所有子元素都是`Flex item`。容器有两个轴线排列，水平轴和垂直轴，默认为水平轴排列
### flex-grow、flex-shrink、flex-basis 属性
- flex-grow: 放大比例，默认为0，即如果存在剩余空间，也不放大。
- flex-shrink: 项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。
- flex-basis: 定义了在分配多余空间之前，项目占据的主轴空间。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，也就是项目的本来大小。
flex属性即三者的简写，默认为`flex: 0 1 auto`
## BFC
BFC是一个独立的渲染区域，让处于 `BFC` 内部的元素与外部的元素相互隔离，使内外元素的定位不会相互影响。

**渲染规则：**
- 内部盒子在垂直方向排列，并且在同一BFC内部相邻盒子的margin会发生重叠(垂直and水平)
- BFC区域不与外部浮动区域重叠，计算height时内部浮动元素也参与
- BFC为独立容器，内外元素互不影响

触发条件：
- display: flex、grid、inline-block
- position: absolute、fixed
- overflow: hidden、auto、scroll

## 重排 & 重绘 & 合成
**重排:** 对DOM结构的修改，引发DOM几何尺寸发生变化会引起重排过程。又称为回流。
**引发重排的操作:** width、padding等几何属性的修改；DOM节点发生增减或移动；读写offset、client等属性；
![[Pasted image 20240307172041.png]]

**重绘:** 页面中元素样式的改变不影响它在文档流中的位置时，浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。由于跳过了布局树和CSSOM树的生成，故效率会比重排要高。
**引发重绘的操作:** color、backgroundColor、visibility等。
![[Pasted image 20240307172030.png]]

**合成:** 更改了一个既不要重新布局、也不要重新绘制的属性，直接执行合成操作。合成的动作交由GPU和合成线程加速处理，不涉及到CPU和主线程，所以即使主线程发生卡顿，也不会影响合成动作的效果。
**触发合成的属性:** CSS3的transform、opacity等。

>故在动画方面，倾向于使用效率更高的transform；而不是引发重排的绝对定位、固定定位等。

![[Pasted image 20240307172015.png]]
```ad-cmt
will-change属性：用于提前告知浏览器元素将要发生哪些变化，让浏览器有在处理复杂动画时进行优化；不要过度使用，会增加内存消耗；可以在变化前赋予，变化后删除。

可接受的值：
- auto（默认值），不做处理
- scroll-position: 元素滚动位置将发生变化
- contents: 元素内容变化
- CSS属性名: transform、opacity等
```
## 伪类 & 伪元素
**伪类:** 利用特殊的伪类选择器，为满足某一规律的元素添加特殊样式。由于是在已有的元素上对样式进行修改，故不产生新元素
```css
a:hover {color: #FF00FF}
p:first-child {color: red}
```
**伪元素:** 在元素前方或后方插入的内容，但实际并不在文档中生成，只是在网页的显示上可见。
```css
p::before/after/first-letter/first-line
```

伪类是通过伪类选择器对元素**按某种规则进行样式的修改**。伪元素则是对元素**添加附属内容**，但不在文档中显示
## CSS选择器权重
根据选择范围大小，优先级与其成反比。id > 类 > 标签 > 通配符
- **属性**选择器 ( a[ref="11"]) 、**伪类**选择器 ( li: last-child ) 、**类选择器**权重相同
- **伪元素**选择器 (p : : after) 与**标签**选择器权重相同
- 相邻兄弟选择器: li+p, 子元素选择器: li>p, 后代选择器: li p 的权重 = 通配符权重
## 隐藏元素的方法和区别
### display: none
**渲染树**不会包含该对象，页面中也不会占据位置，绑定事件不会响应
DOM树构建的时候会包含display: none的元素，但与CSS样式规则合并为渲染树时，不会包含。
### visibility: hidden
元素不显示但占据页面空间，绑定事件不响应

>与display: none的区别
>1. none元素在渲染树上消失，不占用页面空间。hidden的元素依然在渲染树上，所占空间依旧存在。
>2. none不可继承，但子元素也会被删除，自然不会显示。hidden可继承，故其子元素也会消失
>3. none引发重排，hidden仅是重绘
>
### opacity: 0
透明度为0不显示但占据空间，可以响应绑定的事件
## 文本超出显示省略号
```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```
## 小于12px的字体
默认的最小字体为12px。
可以使用`transform: scale()`来缩放，但由于其只作用于可以定义高度的元素，故要将行内元素使用`display: inline-block`转为可设置宽高的元素。
## 像素单位
px: 绝对单位，页面按精确像素展示

em: 相对单位，`基准点为父节点字体的大小`，如果自身定义了`font-size`按自身来计算（浏览器默认字体是`16px`），整个页面内`1em`不是一个固定的值。

rem：`以根元素的字体大小为基准`。例如`html`的`font-size: 16px`，则子级`1rem = 16px`

vw和vh: `1vw` 就等于可视窗口宽度的百分之一; `1vh` 就等于可视窗口高度的百分之一
## 居中方案
### 水平居中方案
- 行内元素，给其父元素设置`text-align:center`
- 定宽元素：
	- `margin: 0 auto`
	- 子绝父相，left: 50%  &  margin-left: -1/2 width
- 不定宽元素
	- 父flex布局，子元素设置margin: 0 auto
	- 父flex布局, 并设置justify-content: center
	- 子绝父相，`left: 50%` `transformX(-50%)`
### 垂直居中方案
- 单行文本，设置line-height = 父元素高度
- 定高元素：
	- `margin: auto 0`
	- 子绝父相, top: 50% + margin-top: -1/2 height
- 不定高元素
	- 父flex布局，子元素设置margin: auto 0
	- 父flex布局，子元素设置align-items: center
	- 子绝父相，`top: 50%` `transformY(-50%)`
### 水平垂直居中
- 父flex布局: justify-content & align-items: center
- 父flex布局，子元素设置`margin: auto`
- 子绝父相: 子元素设置上下左右为0 + margin: auto
- 子绝父相(知道宽高的前提下)：`left: 50%` & `top: 50%`，然后用margin或transform负宽高的一半
## 布局方案
### 两栏布局
### 三栏布局
## 图形绘制
### 三角形
![[Pasted image 20240229223954.png]]
```css
.div {
	width: 0;
	height: 0;
	border: 50px solid transparent;
	border-bottom-color: blue;
}
```
### 扇形
![[Pasted image 20240229224020.png]]
```css
.div {
	width: 0; 
	height: 0;
	border-radius: 50%;
	border: 50px solid transparent;
	border-top-color: blue;
}
```