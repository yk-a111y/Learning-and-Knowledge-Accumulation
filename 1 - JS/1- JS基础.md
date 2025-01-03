### JS基础
#### 类型判定

1. **typeof**：判断数据类型的二进制码，**无法区分null和Object**；
基本数据类型除了null都可以准确判定
引用数据类型除了Function都会显示Object
>typeof null =\=\= 'object'
typeof NaN =\=\= 'number'

2. **instanceof**：x instanceof y 判断构造函数y的Prototype是否在x的原型链上(能准确判断引用类型，不能精准判断原始数据类型)
```jsx
function instanceof (x, y) {
  let proto = Object.getPrototypeOf(x); // 获取x原型
  let prototype = y.prototype; // 获取构造函数y的prototype

  while (true) {
    if (!proto) return false;
    if (proto === prototype) return true;
    proto = Object.getPrototypeOf(proto) // 如果不等于，沿着原型链继续查找
  }
}

2 instanceof Number // false
function() {} instanceof Function // true
```

3. **Object.prototype.toString.call()**
```jsx
var a = 'abc'
Object.prototype.toString.call(a) // [object String]
```

4. **constructor/constructor.name: **指向当前对象的构造函数，由于constructor是原型上属性且可被更改，故通过它判断不安全
```jsx
var a = 'abc'
console.log(a.constructor) // f String() { [native code] }
console.log(a.constructor.name) // 'String'
a.__proto__.hasOwnProperty(constructor) // true
```
#### null 和 undefined 的区别
- null代表的含义是空对象。用于初始化可能会返回对象的变量。`typeof null`会返回`object`
- undefined的含义是未定义。声明后的变量如果未定义，返回undefined。
#### new 对象的过程
```js
function myNew(fn, ...args) {
	// 1. 新建对象
	let obj = {};
	// 2. 新对象的隐式原型指向构造函数的显式原型
	obj.__proto__ = fn.prototype;
	// 3. 调用构造函数，并绑定this指向
	let res = fn.apply(obj, args)
	// 4. 判断构造函数是否有返回值，没有的话直接返回新对象。
	return res instanceof Object ? res : obj;
}
```

#### JS继承

##### 原型继承--父实例作为子类原型
```javascript
let person = {
  name: 'yk',
  age: 18
}

let p1 = Object.create(person) 

// 缺点：子类的实例共享了父类的引用属性，可以通过盗用构造函数解决
```
> Object.create() => 以现有对象为原型，创建一个新对象


 ![[1- JS基础方法与属性 2024-02-07 21.17.00.excalidraw]]

##### 组合继承--原型继承+盗用构造函数
```jsx
// 父类
function Father (name) {
  this.name = name
  this.hobby = ['足球', '篮球']
}
Father.prototype.getName() = function () {
  console.log(this.name)
}

// 子类继承
function Son(name, age) {
  Father.call(this, name); // 盗用构造函数
  this.age = age;
}
Son.prototype = new Father();
Son.prototype.constructor = Son;

// 缺点：存在效率问题，即调用了两次构造函数
```
![[1- JS基础方法与属性 2024-02-07 21.19.32.excalidraw]]
##### 寄生组合继承
```jsx
// 父类
function Father (name) {
  this.name = name
  this.hobby = ['足球', '篮球']
}
Father.prototype.getName() = function () {
  console.log(this.name)
}

// 子类继承
function Son (name, age) {
  Father.call(this.name) // 将Father本身属性挂载到Son上
  this.age = age
}
// 将Father原型上的属性挂载到Son上
Son.prototype = Object.create(Father.prototype)
Son.prototype.constructor = Son
```
![[1- JS基础方法与属性 2024-02-07 21.24.11.excalidraw]]
##### extends
>ES6特性，即为寄生组合继承的语法糖
```jsx
class Son extends Father {
  constructor (age) {
    super('name')
    this.age = age
  }
}
```


#### JS循环
除for循环外的循环特点如下
##### forEach循环
该方法不能用break和continue关键字和return返回数据，效率与for循环相同。优点是语法简洁，不用担心下标问题
```jsx
Array.prototype.myForEach = function(fn){
  var len = this.length;
  for(var i = 0; i < len; i ++){
	  //将元素传给回调函数
	  fn(this[i],i);
  }
}
```
由源码可知，forEach并没有创建新的数组，只是单纯的遍历操作。并且没有对原数组的值进行更改(除引用值外)。若想对原数组进行操作，可以使用第二、第三个参数index和arr
##### for in 循环
性能较差，因为会遍历自身和原型上所有的可枚举属性(Enumerable为true)；而`Object.keys()`只返回对象自身的可枚举属性。

遍历时可能发生乱序（数字属性按照索引值大小，字符串属性根据创建的顺序排列，详见[[3- 浏览器原理#排序属性和常规属性|排序属性与常规属性]]）。与之相对的，Object.getOwnPropertyNames(获取自身所有实例属性，无论是否可枚举) 与getOwnPropertySymbols的枚举顺序确定: 升序枚举数值键，然后字符串键和Symbol。

无法遍历Symbol属性(不可枚举)，Symbol需要用Object.getOwnPropertySymbols('obj')来获取
```js
let arr = Object.keys(obj).concat(Object.getOwnPropertySymbols(obj))
```
##### for of 循环
按照是否有迭代器规范来循环。带有Symbol.iterator的均实现了迭代器规范(Object没有)，如：Array、Set、Map和类数组对象(Arguments)等。它可以正常响应break、continue和return。
```js
let arr = [1,2,3];

arr[Symbol.iterator] = function () {
  let self = this, index = 0
  return {
	next () {
	  // 如果索引值大于长度-1则遍历结束
	  if (index > self.length - 1) {
		return {
		  done: true,
		  value: undefined
		}
	  }
	  // 否则done为false，获取的值为self(index)然后index再自增
	  return {
		done: false,
		value: self(index++)
	  }
	}
  }
}
```
即使不能for of的类数组对象，只要满足两个条件: 具备0,1,2...的顺序索引，具备length属性，再添加Symbol.iterator就可以被遍历。
```js
let obj = { 0: 10, 1: 11, 2: 12, length: 3}
obj[Symbol.iterator] = Array.prototype[Symbol.iterator]
for (const value of obj) {
  console.log(value) // 10, 11, 12
}
```
#### 闭包
闭包是在一个作用域中可以访问另一个函数内部的局部变量的函数。创建闭包的最常见的方式就是在一个函数内创建另一个函数，创建的函数可以访问到当前函数的局部变量

**闭包的用途主要有两种:** 保存和保护。**保存**是指将函数内部私有变量的生命周期通过闭包访问的方式延长；**保护**是指其运用函数特有的作用域，使函数内部私有变量不受全局影响，也不会污染全局。
#### 变量提升
概念：JS代码在执行过程中，JS引擎把变量声明和函数声明提升到代码开头的行为。变量提升后，会被赋值为undefined，表示声明但未定义。『注意』仅var存在变量提升，let和const不支持。

变量提升的**好处**：
- 提高性能：预编译过程中的变量提升，可以预先为变量分配栈空间。
- 容错率变高：变量提升使得某些不规范的代码变得可以执行。

变量提升的**问题**：
- 变量覆盖：
```js
var name = "JavaScript"
function showName(){
  console.log(name);
  if(0){
   var name = "CSS"
  }
}
showName()
```
此时打印的name位undefined。因为，showName作用域中的赋值操作虽然永不会执行，但变量提升会把name的声明提升到showName的顶层，使其访问变为undefined。
- 变量没有被销毁：
```js
function foo(){
  for (var i = 0; i < 5; i++) {
  }
  console.log(i); 
}
foo()
```
i的最终值并未被销毁。最后打印出5。因为在创建上下文阶段，i就已经被提升了。
#### var let const
- let和const支持块级作用域，var不支持
- let和const不存在**变量提升**，var存在。导致了let和const在声明之前不可用 **(暂时性死区)**
- var可以**重复声明**，后声明的同名变量会覆盖之前的。let和const不可以重复声明
- var声明的变量为全局变量，是为全局对象(浏览器为window，Node为global)增加属性
- var和let不必须设置初始值，且后续值可以改变。const必须初始化，并且不可更改
#### this指向
- 函数调用模式: this在函数中调用，非严格模式下指向全局对象，严格模式下为undefined。
- 方法调用模式: 函数作为一个对象的方法来使用，this指向这个对象。
- 事件函数: this指向事件源。
- 构造器调用模式: 一个函数用 new 调用时，函数执行前会新创建一个对象，this 指向这个新创建的对象。
- call、apply、bind的第一个参数可以改变this指向。
#### 箭头函数与普通函数的区别
- 箭头函数不可以使用arguments参数。没有原型(故new的时候会报错)、也没有自己的this，它的this只与定义时的上下文环境有关，故箭头函数内部的this一旦确定，不可更改。
- 箭头函数不能用作Generator函数，不可以使用yield关键字

#### 原型和原型链
**原型:** JS使用构造函数创建对象，每一个构造函数内部存在一个prototype属性，即显式原型。这个对象包含了构造函数所创建实例的共有属性和方法。针对于创建的实例来讲，可以利用proto属性(隐式原型)来访问其原型，但proto不在规范中。ES5后应使用Object.getPrototypeOf()方法来获取原型。
**原型链:** 当访问一个对象的属性时，如果对象内部不存在该属性，那么就会沿着隐式原型proto指针所构成的原型链一直向上查找至null(Object.prototype.proto)。尽头一般为Object.prototype，这也说明了为什么大多数对象可以调用toString()这个方法
#### Reflect
##### why Reflect ?
- 将Object中属于语言内部的方法（**Object.defineProperty**）放到Reflect对象上。
- 修改某些Object方法返回的不合理结果，比如defineProperty无法定义属性时会抛出错误，Reflect.defineProperty则会返回false
- 将Object的命令式行为，改写为函数式行为。key in Obj 改写为：Reflect.has(key)
##### 静态方法（13个）
1. Reflect.**get**(target, key, receiver) & Reflect.**set**(target, key, value, receiver)
	>**receiver:** target内部如果部署了getter，则getter中的this绑定receiver
```js
let obj = {
	foo: 1,
	bar: 2,
	get baz() {
		return this.foo + this.bar;
	}
}

let receiverObj = {
	foo: 4,
	bar: 4
}

Reflect.get(obj, baz, receiverObj) // 8
```
2. Reflect.**has**(obj, key)
> 对应key in obj 里的 in运算符
3. Reflect.**deleteProperty**(obj, key)
> 对应delete obj[key]
4. Reflect.**construct**(obj, args)
>对应 new target(...args)，不使用new调用构造函数的方法
5. Reflect.**getPrototypeOf**(obj) & Reflect.**setPrototypeOf**(obj, new)
> 读取obj的\__proto__属性
6. Reflect.**apply**(func, thisArg, args)
> Function.prototype.apply.call(fn, obj, args) => 简化为Reflect.apply(fn, obj, args)
7. Reflect.**defineProperty**(obj, propertyKey, attributes)
> 对应Object.defineProperty
8. Reflect.**ownKeys**(obj)
> 获取obj所有属性 === Object.getOwnPropertyNames + Object.getOwnPropertySymbols
9. Reflect.**isExtensible**(target) & Reflect.**preventExtensions**(obj)
>isExtensible判断target是否可扩展， preventExtensions将obj变为不可扩展
10. Reflect.**getOwnPropertyDescriptors**(obj, propertyKey)
> 拿到指定obj属性的描述
```js
let myObj = {};
Object.defineProperty(myObj, 'hidden', {
	value: true,
	enumerable: false
})

Reflect.getOwnPropertyDescriptors(myObj, 'hidden');
// {
//	value: true,
//	enumerable: false,
// 	writable: false,
//	configurable: false
// }
```
#### Map 和 WeakMap

Map可以使用任何类型作为key，来形成key-value的结构。其内部原理是维护了两个数组，分别存储key和value，故垃圾回收机制无法回收。可以调用keys()、values()、entries()方法。

WeakMap只可以使用引用数据类型作为key，作为弱引用数据结构，不可以调用keys等方法和size属性。因为它的内容取决于GC是否执行，执行前后的内容会发生变化。

WeakMap的常用场景：
- 关联DOM结构，这样在DOM结构被删除后，WeakMap的弱引用不会被引用计数或标记整理算法标记，可以直接回收DOM。
- 对私有属性的缓存，实例对象的私有属性可以用WeakMap缓存，这样当实例卸载时，私有属性也会消失。
#### 数组常用方法
##### 类数组对象的转换
拥有length属性和若干索引属性的对象即类数组对象，和数组相似但不可以调用数组方法。常见的如: arguments、DOM操作的返回值等。
- Array.from(arrLike)
- Array.prototype.slice.call(arrLike)
- Array.prototype.splice.call(arrLike, 0)
- Array.prototype.concat.apply([]. arrLike)
##### 改变原数组的方法
即原地修改，不生成新数组

1. splice(): 截取数组
```javascript
// 从start开始，截取num个元素。返回值是截取出来的数组
arr.splice(start, num)

// 在start位置，删除num个元素后插入item；如果num为0，则直接执行插入操作，不删除元素
arr.splice(start, num, item1, item2, ...) 
```

2. push()、pop()：数组尾新增/删除元素，改变原数组
> push返回新长度，pop返回被删除的元素

3. unshift()、shift()：数组头新增/删除元素，改变原数组
> unshift返回新长度，shift返回被删除元素

4. reverse()：翻转数组，返回翻转好的数组
5. sort()：排序数组
> sort((a, b) => a - b) **升序 **

##### 不改变原数组的方法
即操作的过程中会返回新数组

1. slice(start, end): 返回新数组，包含\[start, end)的所有元素 **浅拷贝** [[1- 基础方法与属性#浅拷贝方法]]
2. concat(data): 连接两个或多个数组，且不改变现有数组，返回被连接数组的副本 **浅拷贝**[[1- 基础方法与属性#浅拷贝方法]]
```js
let arr1 [1, 2], let arr2 = [3]
let arr3 = arr1.concat(arr2)  // [1, 2, 3]
```

3. join(symbol): 数组转string，并用symbol连接，返回转好的string
4. indexOf / lastIndexOf
>
indexOf(data）=> 检查有无该data，有返回索引，无返回-1
indexOf(data, index) => 检查从index开始
##### ES6新增方法

1. forEach
2. map
3. filter
4. every & some & find
5. reduce
#### 事件流与事件委托
事件流是JS规定的事件执行方向，由最上层元素至目标元素的事件捕获 -> 目标元素事件触发 -> 由目标元素向最上层元素的事件冒泡三个阶段组成。事件捕获和事件冒泡二者是相斥的，默认的模式是事件冒泡。

`addEventListener(eventName, fn, flase)由第三个参数控制事件捕获/事件冒泡。`

事件委托，又称事件代理。通过将事件处理程序绑定到共同父元素上，利用事件冒泡的特性，来减少大量子元素绑定相同的事件。
#### 鼠标事件属性

1. MouseEvent.clientX/Y：针对当前屏幕的位置信息，点击左上角为（0，0）
2. MouseEvent.pageX/Y：针对整个页面，是当前client数据+滚动条距离
3. MouseEvent.screenX/Y：提供鼠标对于屏幕的偏移量
4. MouseEvent.offsetX/Y：目标节点相对于事件对象padding edge的偏移量
#### 内容宽高属性
##### 事件对象
e.pageY -- 到文档顶部的距离；e.clientY到可视区域顶部的距离；e.offsetY到触发事件元素顶部的距离；e.screenY鼠标距离屏幕顶部的距离；「X轴同理」

[注] React的合成事件是缺少offsetY属性的，可以用e.nativeEvent.offsetY获取；或者自己算: offsetY = pageY - getBoundingClientRect().top - window.scrollY
![[Pasted image 20240813154749.png]]
##### getBoundingClientRect方法
该方法(element.getBoundingClientRect)返回一个DOMRect对象，是包含整个元素的最小矩形(padding、border-width也包含在内)。
![[Pasted image 20240813160714.png]]
##### client系列
clientWidth/Height: width / height  + 左右padding
clientTop/Left: 上/左边框的宽度
![[1- JS基础 2024-08-13 16.26.40.excalidraw]]
##### offset系列
offsetWidth/Height:：width / height  + 左右/上下padding + 左右/上下border
offsetTop：当前元素**上边框外缘 **到 最近的已定位父级（offsetParent）**上边框内缘 **的距离。无定位父级则为到body顶部距离
offsetLeft：当前元素**左边框外缘 **到 最近的已定位父级（offsetParent）**左边框内缘 **的距离。无定位父级则为到body左边距离
##### scroll系列
scrollWidth/Height: 标签内容层的实际宽高（可视区域宽高+被隐藏的部分）
scrollTop/Left: 内容层顶/左端 到可视区域顶/左端
window.scrollY/X: 页面在某个方向上的滚动值

![image.png](https://cdn.nlark.com/yuque/0/2023/png/25949356/1697175795783-7a96b2fa-3336-45f8-9f40-749bec9f3a18.png#averageHue=%23f0dbb7&clientId=u987ba2ff-08c5-4&from=paste&height=525&id=u6959d4ef&originHeight=867&originWidth=1131&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=151708&status=done&style=none&taskId=u87c83261-ae12-4ff9-8265-986319e08f7&title=&width=685.454505836342)
#### requestAnimationFrame
H5提供的请求动画的API，在主线程（JS引擎线程）上执行，如果主线程忙碌，动画效果可能会打折扣。其效果好的原因是：
- 其执行频率与当前设备的刷新频率相关，在每一次刷新间隔中只执行一次，避免丢帧和卡顿；
- 在页面处于不可见状态时，并不会执行，有效节省了CPU开销。
- 注：节流也可以利用requestAnimationFrame的特性，防止一次刷新间隔内多次执行。
##### 前置知识：
- 页面可见：当页面被最小化或者被切换成后台标签页时，页面为不可见，浏览器会触发一个 visibilitychange事件,并设置document.hidden属性为true；切换到显示状态时，页面为可见，也同样触发一个 visibilitychange事件，设置document.hidden属性为false。
- 动画帧请求回调列表：每个Document都有一个动画帧请求回调函数列表，该列表可以看成是由<handlerId, callback>元组组成的集合。其中handlerId是一个整数，唯一地标识了元组在列表中的位置；callback是回调函数
##### 具体用法
```js
// handlerId 为浏览器生成的大于0的整数
const handlerId = requestAnimationFrame(callback)

// 取消动画的方法
cancelAnimationFrame(handlerId)
```
#### Web Worker
Web Worker 是 HTML5 标准的一部分，这一规范定义了一套 API，允许我们在 js 主线程之外开辟新的 Worker 线程，并将一段 js 脚本运行其中，它赋予了开发者利用 js 操作多线程的能力。

因为是独立的线程，Worker 线程与 js 主线程能够同时运行，互不阻塞。所以，在我们有大量运算任务时，可以把运算任务交给 Worker 线程去处理，当 Worker 线程计算完成，再把结果返回给 js 主线程。这样，js 主线程只用专注处理业务逻辑，不用耗费过多时间去处理大量复杂计算，从而减少了阻塞时间，也提高了运行效率，页面流畅度和用户体验自然而然也提高了。

使用：`const worker = new Worker(path, options);`
path - 有效的js脚本的地址，必须遵守同源策略
options - type: 指定worker类型默认为classic；name：表示worker scope的一个DOMString值，用于调试。

##### worker线程与主线程间的数据传递
```js
// 主线程
const worker = new Worker('./test.js');

worker.postMessage(...args) // 向worker发送消息
sliceFileWorker.onmessage = (e) => {
	// 接收worker的返回值
}

// worker线程
self.onmessage = (e) => {
  // worker的逻辑
  let res = ...
  //将计算结果通过postMessage发送给主线程
  self.postMessage(res);
}
```
##### worker关闭
在主线程关闭
```js
// main.js（主线程）
const myWorker = new Worker('/worker.js'); // 创建worker
myWorker.terminate(); // 关闭worker
```
在worker线程关闭
```js
self.close(); // 直接执行close方法就ok了
```
#### Service Worker
#### 五类Observer综述
##### IntersectionObserver
IntersectionObserver 可以监听一个元素和可视区域相交部分的比例，然后在可视比例达到某个阈值的时候触发回调
```js
const intersectionObserver = new IntersectionObserver(
	function (entries) {
		entries.forEach(item => { console.log(item.target, item.intersectionRatio) })
	},
	{
		threshold: [0.5, 1] // 可视比例在0.5和1时触发回调
	}
)

// 调用observe方法
intersectionObserver.observe(document.querySelector('#box1'))
```
##### MutationObserver
监听一个JS对象的变化，我们会用 Object.defineProperty 或者 Proxy；而监听DOM元素的属性和子节点的变化，我们可以用 MutationObserver。
```js
const mutationObserver = new MutationObserver((mutationsList) => {
    console.log(mutationsList)
});

mutationObserver.observe(box, {
    attributes: true,
    childList: true
});
```
##### ResizeObserver
元素可以用 ResizeObserver 监听大小的改变，当 width、height 被修改时会触发回调
```js
const resizeObserver = new ResizeObserver(entries => {
    console.log('当前大小', entries)
});
resizeObserver.observe(box);
```
##### ReportingObserver
##### PerformanceObserver

### JS手写
#### 大数相加
```js
function addStrings(num1, num2) {
	const result = [];
	let carry = 0, i = num1.length - 1, j = num2.length - 1;

	while (i >= 0 || j >= 0) {
		const digit1 = i >= 0 ? parseInt(num1[i]) : 0;
		const digit2 = j >= 0 ? parseInt(num2[j]) : 0;
		const sum = digit1 + digit2 + carry;
		
		result.unshift(sum % 10);
		carry = Math.floor(sum / 10);

		i--, j--;
	}

	return result.join('');
}
```

#### 实现一个Queue
因为在操作JS原生的数组的过程中，如果是一个非常大的数组，shift带来的时间复杂度为O(n)，Queue可以实现O(1)级别的复杂度，并且不改变Array可遍历的特性
```js
class INode<T> {
	value: T;
	next: INode<T> | null;

	constructor(value: T, next: INode<T> | null = null) {
	this.value = value;
	this.next = next;
	}
}

export default class Queue<T> {
  private head: INode<T> | null;
  private tail: INode<T> | null;
  private size: number;

  constructor() {
    this.head = null;
    this.tail = null;
    this.size = 0;
  }

  enqueue(value: T) {
    const newNode = new INode<T>(value);
    if (this.tail) {
      this.tail.next = newNode;
      this.tail = newNode;
    } else {
      this.head = newNode;
      this.tail = newNode;
    }
    this.size++;
  }

  dequeue() {
    const currentHead = this.head;
    if (!currentHead) return;

    this.head = currentHead.next;
    this.size--;

    return currentHead.value;
  }

  get _size() {
    return this.size;
  }

  clear() {
    this.head = null;
    this.tail = null;
    this.size = 0;
  }

  // 生成器函数，将Queue变为可迭代的
  *[Symbol.iterator]() {
    let current = this.head;

    while (current) {
      yield current.value;
      current = current.next;
    }
  }
}
```
具体使用如下：
```js
const queue = new Queue();
queue.enqueue("1");
queue.enqueue("2");
queue.enqueue("3");
console.log(queue);

for (const value of queue) {
  console.log(value); // 输出 1, 2, 3
}

queue.dequeue();
queue.dequeue();
console.log(queue);
```
#### 数组常用算法
##### 数组扁平化
```js
// 1. 方法一：reduce
function flatten(arr) {
	return arr.reduce((res, next) => {
		return res.concat(Array.isArray(next) ? flatten(next) : next);
	}, [])
}

// 2. 方法二：递归
function flatten(arr) {
	let res = [];
	for (const ele of arr) {
		if (Array.isArray(ele)) {
			res.concat(flatten(arr));
		} else {
			res.push(ele);
		}
	}

	return res;
}
```
##### 数组去重
```js
// 1. forEach + indexOf
function unique(arr) {
	const res = [];
	arr.forEach(item => {
		if (res.indexOf(item) === -1) {
			res.push(item);
		}
	})

	return res;
}

// 2. forEach + obj
function unique(arr) {
	const res = [];
	const obj = {};
	arr.forEach(item => {
		if (!obj[item]) {
			obj[item] = true;
			res.push(item);
		}
	})

	return res;
}

// 3. reduce + Map
function unique(arr) {
	const res = [];
	arr.reduce((pre, next) => {
		if (!pre.get(next)) {
			pre.set(next, true);
			res.push(next);
		}
		return pre;
	}, new Map());

	return res;
}

// 4. 扩展运算符
const res = [...new Set(arr)]
```
##### reduce
```js
function myReduce(arr, cb, initValue) {
	let res = initValue === undefined ? arr[0] : initValue;
	let i = initValue === undefined ? 1 : 0

	for (i; i < arr.length; i++) {
      res = cb(res, arr[i], i);
    }

	return res;
}
```
##### reduce实现map
```js
funtion myMap(fn, callbackThis) {
	const res = [];
	// map第二个参数为this指向
	let cbThis = callbackThis || null;

	this.reduce((cur, index, arr) => {
		res.push(fn.call(cbThis, cur, index, arr));
	}, null)
}
```
##### 合并数组
```js
// 方法一：concat
// 缺点：a、b数组不变，返回的是一个新数组，浪费内存
let newArr = a.concat(b)

// 方法二：...运算符
// 性能最优
let newArr = [...a, ...b];

// 方法三：push
a.push.apply(a, b)

// 方法四：for of
for (let ele of b) { a.push(ele) }
```

##### 洗牌算法
```js
// 方法一：原地互换
function shuffle(arr) {
	for (let i = 0; i < arr.length; i++) {
		let randomIndex = i + Math.floor(Math.random() * (arr.length - i));
		[arr[randomIndex], arr[i]] = [arr[i], arr[randomIndex]];
	}
}

// 方法二：创建新数组
function shuffle(arr) {
	let res = [];
	while (arr.length) {
		let randomIndex = Math.floor(Math.random() * arr.length);
		res.push(arr.splice(randomIndex, 1)[0]);
	}

	return res;
}
```
#### 浅拷贝、深拷贝
>二者的区别: 
>- 浅拷贝创建一个原始对象的精确拷贝，原始对象的属性为基本类型，则拷贝值。为引用类型，拷贝该引用的地址。所以，拷贝对象和原始对象任意一方改变该引用地址的内部信息时，都会影响到另一方。
>- 深拷贝将一份对象从内存中完整拷贝，在堆内存开辟新区域存放。修改时，互相不影响。
##### 浅拷贝方法
- = 直接赋值属于浅拷贝
- 数组的concat和slice，根据原数组创建一个新数组，这一过程是浅拷贝
- `let newObj = {...obj}`
- `let newObj = Object.assign({}, obj);`
##### 深拷贝方法
方法一：暴力json
局限性：undefined、function和symbol在转换过程中会被忽略
```js
let obj = { name: 'yk' };
let newObj = JSON.parse(JSON.stringify(obj));
```
方法二：循环递归
大致思路是：首先将数据类型分为基本数据类型和引用数据类型（数组、对象），基本数据类型可直接返回，引用数据类型会涉及到递归处理。之后，再将引用数据类型分为可遍历（map、set、数组、对象）和不可遍历（Symbol、正则Reg、函数Function）两种，根据这些case继续细分处理。
```js
// 可遍历的类型
const mapTag = '[object Map]';
const setTag = '[object Set]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';

// 不可遍历类型
const symbolTag = '[object Symbol]';
const regexpTag = '[object RegExp]';
const funcTag = '[object Function]';

// 将可遍历类型存在一个数组里
const canForArr = ['[object Map]', '[object Set]',
                   '[object Array]', '[object Object]']

// 将不可遍历类型存在一个数组
const noForArr = ['[object Symbol]', '[object RegExp]', '[object Function]']

// 判断类型的函数
function checkType(target) {
    return Object.prototype.toString.call(target)
}

// 判断引用类型的temp
function checkTemp(target) {
    const c = target.constructor
    return new c()
}


function deepClone(target, map = new Map()) {

	// 首先获取target类型
	const type = checkType(target)

	// 基本数据类型直接返回 
	if (!canForArr.concat(noForArr).includes(type)) {
		return target
	}

	// 处理Symbol、Reg、Function
	if (type === symbolTag) return cloneSymbol(target)
	if (type === regexpTag) return cloneReg(target)
	if (type === funcTag) return cloneFunction(target)

	// 引用数据类型特殊处理
	const temp = checkTemp(target)
	
	// 解决循环引用的bug，已存在直接返回
	if (map.get(target)){
		return map.get(target)
	}
	// 不存在则第一次设置
	map.set(target, temp)

	// 处理Map类型
    if (type === mapTag) {
       target.forEach((value, key) => {
           temp.set(key, deepClone(value, map))
        })

        return temp
    }
	// 处理Set类型
    if (type === setTag) {
       target.forEach(value => {
           temp.add(deepClone(value))
        })

        return temp
    }
	// 处理数组和对象
	for (const key in target) {
		temp[key] = deepClone(target[key], map);
	}
	
	return temp;
}

// 拷贝Symbol的方法
function cloneSymbol(targe) {
    return Object(Symbol.prototype.valueOf.call(targe));
}
// 拷贝RegExp的方法
function cloneReg(targe) {
    const reFlags = /\w*$/;
    const result = new targe.constructor(targe.source, reFlags.exec(targe));
    result.lastIndex = targe.lastIndex;
    return result;
}
// 拷贝Function的方法
function cloneFunction(func) {
    const bodyReg = /(?<={)(.|\n)+(?=})/m;
    const paramReg = /(?<=\().+(?=\)\s+{)/;
    const funcString = func.toString();
    if (func.prototype) {
        const param = paramReg.exec(funcString);
        const body = bodyReg.exec(funcString);
        if (body) {
            if (param) {
                const paramArr = param[0].split(',');
                return new Function(...paramArr, body[0]);
            } else {
                return new Function(body[0]);
            }
        } else {
            return null;
        }
    } else {
        return eval(funcString);
    }
}
```

#### deepEqual
```js
function deepEqual(a, b) {
  if (a === b) {
    return true;
  } else if (
    typeof a === "object" &&
    a !== null &&
    typeof b === "object" &&
    b !== null
  ) {
    const keysA = Object.keys(a);
    const keysB = Object.keys(b);
    if (keysA.length !== keysB.length) {
      return false;
    }
    for (const key of keysA) {
      if (!deepEqual(a[key], b[key])) {
        return false;
      }
    }
    return true;
  } else {
    return false;
  }
}
```
#### call apply bind
call
```js
Function.prototype.myCall = function (context, ...args) {
	context = context || window;
	
	// 给context添加一个fn属性，this即调用call的函数
	const fn = Symbol();
	context[fn] = this;
	const res = context[fn](...args);
	// 删除fn
	delete context[fn];

	return res;
}
```

apply => 只在传参上与call有出入
```js
Function.prototype.myApply = function (context, ...args) {
	context = context || window;
	
	// 给context添加一个fn属性，为当前调用call的函数对象
	context.fn = this;
	const res = context.fn(args);
	// 删除fn
	delete context.fn;

	return res;
}
```
bind 返回一个绑定了this的函数
```js
Function.prototype.myBind = function (context, ...args) {
	const fn = this;
	// bind 返回的是一个函数
	return function bindFn(...newArgs) {
		if (this instanceof bindFn) {
			return fn.apply(this, [...args, ...newArgs]);
		}
		return fn.apply(context, [...args, ...newArgs]);
	}
}
```

#### lodash的get方法
达成如下效果
```js
var object = { 'a': [{ 'b': { 'c': 3 } }] };  
  
_.get(object, 'a[0].b.c');  
// => 3  
  
_.get(object, ['a', '0', 'b', 'c']);  
// => 3  
  
_.get(object, 'a.b.c', 'default');  
// => 'default'
```
代码实现
```js
function get(obj, path, defaultValue = undefined) {
	// 处理path
	if (typeof path === 'string') {
		// 将 a[0].b.c[1] => a.0.b.c.1
		path = path.replace(/([\[\]])/g, ($1) => {
			return $1 === '[' ? '.' : ''
		}).split('.');
	}
	
	// edge case
	if (!obj || typeof obj !== 'object' || !Array.isArray(path) || path.length === 0) {
		return path.length === 0 ? obj : defaultValue;
	}

	let target = obj;
	for (const item of path) {
		target = target[item]
		if (!target) break;
	}

	return target || defaultValue;
}
```
#### lodash的groupBy
```js
function groupBy(collection, by) {
	return collection.reduce((pre, cur) => {
		if (pre[by(x)]) {
			pre[by(x)].push(cur);
		} else {
			pre[by(x)] = [cur];
		}

		return pre;
	}, {})
}
```
#### 函数柯里化
核心理念是：柯里化后的函数，如果参数达到length，执行原函数。否则，返回新函数。
```js
function curry(fn) {
	return function curried(...args) {
		// args.length => 入参长度；
		// fn.length => 函数行参长度
		if (args.length >= fn.length) {
			// 执行原函数
			return fn.apply(this, args);
		} else {
			// 返回一个新函数
			return function (...args2) {
				curried.apply(this, [...args, ...args2]);
			}
		}
	}
}
```

#### Promise相关
都是将含有多个Promise实例组成的数组包装成一个Promise对象。Promise.all中的所有Promise都成功，则按顺序返回结果数组，出现失败，则整个过程直接结束，返回失败态的Promise。Promise.race为多个Promise对象竞争，返回率先完成执行的Promise的状态
##### 手写Promise All
```js
function myPromiseAll(array) {
  let res = [];
  let count = 0;
  return new Promise((resolve, reject) => {
    function addData(index, value) {
      res[index] = value;
      count += 1;
      // 若所有结果执行完毕，调用resolve
      if (count === array.length) {
        resolve(res);
      }
    }

    for (let i = 0; i < array.length; i++) {
      let currentPromise = array[i];
      // Promise.resolve同时处理值和promise对象，再调用then处理成功或失败
      Promise.resolve(currentPromise).then(
        (value) => addData(i, value),
        reject
      );
    }
  });
}
```
##### 带并发限制的Promise.All
```js
function myPromiseAll (array, limit) {
	const res = [];
	const len = array.length;
	let count = 0; // 当前执行过的个数
	let index = 0; // 当前执行到第几个Promise
	return new Promise((resolve, reject) => {
		for (let i = 0; i < limit; i++) {
			step(i);
		}
	})
	function step(i) {
		if (count === len) { // 执行完毕，返回结果
			resolve(res);
			return;
		}
		let currentPromise = array[index];
		if (currentPromise) {
			currentPromise().then(result => {
				res[i] = result;
				count++;
				step(index); // 递归调用step
			})
		}
		index++:
	}
}
```
##### 手写Promise Race
```js
function myPromiseRace(array) {
	return Promise((resolve, reject) => {
		for (let i = 0; i < array.length; i++) {
			// array数组中任一一个Promise函数状态改变，就可以返回
			Promise.resolve(array[i]).then(data => {
				resolve(data);
			}, reason => {
				reject(reason);
			})
		}
	})
}
```
##### 实现Promise
实现的功能:
- resolve, reject, then(异步API和链式调用)
- finally, catch 和 Promise.resolve()
- Promise.all & Promise.race
```js
const PENDING = 'pending' // 等待状态
const REJECTED = 'rejected' // 失败状态
const FULFILLED = 'fulfilled' // 成功状态

class MyPromise {
  // 1. 构造函数
  constructor (executor) {
    try {
      executor(this.resolve, this.reject);
    } catch (e) {
      this.reject(e);
    }
  }

  // 初始状态
  status = PENDING;
  // 成功的值 & 失败的原因
  value = undefined;
  reason = undefined;

  // 存储成功或失败异步API的回调
  successfulCallback = [];
  failCallback = [];
  
  resolve = value => {
    // 仅PENDING状态可执行
    if (this.status !== PENDING) return;
    this.status = FULFILLED;
    this.value = value;
    while (this.successfulCallback.length > 0) this.successfulCallback.shift()();
  }

  reject = reason => {
	// 仅PENDING状态可执行
    if (this.status !== PENDING) return;
    this.status = REJECTED;
    this.reason = reason;
    while (this.failCallback.length > 0) this.failCallback.shift()();
  }

  then = (successfulCallback, failCallback) => {
	// 当then中为常数时，也能将promise对象的状态继续向下传递
    successfulCallback = successfulCallback ? successfulCallback : value => value
    failCallback = failCallback ? failCallback : reason => { throw reason }

	// 创建返回的Promise对象
	const myPromise = new MyPromise((resolve, reject) => {
	  // 成功态
	  if (this.status === FULFILLED) {
	    setTimeout(() => {
          try {
            let x = successfulCallback(this.value)
            // 判断返回的x是数据还是Promise对象
            // x 为普通值，直接将x的值传递给下一个then。 x 为Promise对象，将x的状态和数据传递给下一个对象
            resolvePromise(myPromise, x, resolve, reject)
          } catch (e) {
            reject(e)
          }
        }, 0)		
	  } else if (status === REJECTED) {
	    setTimeout(() => {
          try {
            let x = failCallback(this.reason)
            resolvePromise(MyPromise, x, resolve, reject)
          } catch (e) {
            reject(e)
          }
        })		
	  } else {
	    // Promise对象包裹异步API时的等待态
	    // 状态为PEDNING时，将异步API的回调存储起来
	    this.successfulCallback.push(() => {
          setTimeout(() => {
            try {
              let x = successfulCallback(this.value)
              resolvePromise(promie2, x, resolve, reject)
            } catch (e) {
              reject(e)
            }
          }, 0)
        })
        this.failCallback.push(() => {
          setTimeout(() => {
            try {
              let x = failCallback(this.reason)
              resolvePromise(promie2, x, resolve, reject)
            } catch (e) {
              reject(e)
            }
          }, 0)
        })
      }
	})
  }
  
  static resolve = value => {
	if (value instanceof MyPromise) return value
    return new MyPromise(resolve => resolve(value))
  }

  // static all 同上
  // static race 同上
}
```

resolvePromise功能: 向下传递Promise状态
```js
function resolvePromise (myPromise, x, resolve, reject) {
  if (x === myPromise) {
    return reject(new TypeError('Chaining cycle detected for promise #<Promise>'))
  }
  if (x instanceof MyPromise) {
    x.then(resolve, reject)
  } else {
    resolve(x)
  }
}
```
#### Scheduler异步任务调度
```js
class Scheduler {
  constructor() {
    this.count = 2; // 最大并发任务数
    this.queue = []; // 任务队列
    this.activeCount = 0; // 当前活动任务数
  }

  add(task) {
    return new Promise((resolve) => {
      const runTask = () => {
        this.activeCount++;
        task().then(() => {
          resolve();
          this.activeCount--;
          if (this.queue.length > 0) {
            const nextTask = this.queue.shift();
            nextTask();
          }
        });
      };

      if (this.activeCount < this.count) {
        runTask();
      } else {
        this.queue.push(runTask);
      }
    });
  }
}

const timeout = (time) =>
  new Promise((resolve) => {
    setTimeout(resolve, time);
  });

const scheduler = new Scheduler();
const addTask = (time, order) => {
  scheduler.add(() => timeout(time)).then(() => console.log(order));
};

addTask(2000, "1");
addTask(500, "2");
addTask(300, "3");
addTask(400, "4");
// output: 2 3 1 4

```
#### EventBus
>基础版本实现：

1. eventMap集合来存储事件， key：事件名，value：事件数组列表
2. 订阅功能：subscribe
3. 发布功能：emit

**进阶一**：最大订阅数量限制 & 支持emit执行时的额外参数
**进阶二**：取消订阅 & 清空事件 & 订阅一次功能
**eventMap结构：**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25949356/1697523081694-f69a775e-a4a7-4222-9142-bdbdc3b5a640.png#averageHue=%23fafafa&clientId=uccedb30b-227c-4&from=paste&height=353&id=uc0c69ef2&originHeight=529&originWidth=875&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=38872&status=done&style=none&taskId=uc5751014-43c2-46ec-bfdf-78468a25298&title=&width=583.3333333333334)
代码实现：
```javascript
class BusService {
	constructor(maxListeners) {
		this.eventMap = {};
		// eventName最大监听数量
		this.maxListeners = maxListeners || Infinity;
		// 标识Id
		this.callbackId = 0;
	}

	// on 注册事件
	subScribe(eventName, cb) {
		if(!Reflect.has(this.eventMap, eventName)) {
			Reflect.set(this.eventMap, eventName, {});
		}
		// 订阅最大数量检测
		if (this.maxListeners !== Infinity && Object.keys(this.eventMap[eventName]).length >= this.maxListeners) {
			console.log(`事件${eventName}超过了可订阅数量`);
			return;
		}
		this.callback += 1;
		this.eventMap[eventName][this.callbackId] = cb;

		// 取消订阅的函数
		const unSubscribe = () => {
			delete this.eventMap[eventName][this.callbackId];
			if (Object.keys(this.eventMap[eventName]).length === 0){
				delete this.eventMap[eventName];
			}
		}

		return unSubscribe;
	}

	// emit 触发事件
	emit(eventName, ...params) {
		if(!Reflect.has(this.eventMap,eventName)){
	      console.warn(`从未订阅过此事件${eventName}`);
	      return 
	    }
	    this.callbackList = this.eventMap[eventName];
	    for (const [id, fn] of Object.entries(callbackList)) {
		    fn.call(this, ...params);
		    if (id.startsWith('once')) {
			    delete callbackList[id];
		    }
	    }
	}

	// 仅订阅一次
	subscribeOnce(eventName, cb) {
		if (!this.eventMap[eventName]) {
			this.eventMap[eventName] = {};
		}
		const onceCallbackId = 'once' + this.callbackId++;
		this.eventMap[eventName][onceCallbackId] = cb;

		// 取消订阅的函数
		const unSubscribe = () => {
			delete this.eventMap[eventName][this.callbackId];
			if (Object.keys(this.eventMap[eventName]).length === 0){
				delete this.eventMap[eventName];
			}
		}

		return unSubscribe;
	}
	
	// 清空某事件
	// 清空所有
}

// 导出实例
export default new BusService();
```
#### 遍历DOM节点
即DOM版本的层序遍历
```js
const traverse = (node) => {
  const res = []
  const helperStack = []

  //对node的处理
  if (node && node.nodeType === 1) {
    helperStack.push(node)
  }

  let curNode

  while (helperStack.length > 0) {
	// 存储每一层的结果
    let curArray = []
    let len = helperStack.length
    for (let i = 0; i < len; i++) {
      curNode = helperStack.shift()
      curArray.push(curNode.tagName)
      if (curNode.children) {
        for (let i = 0; i < curNode.children.length; i++) {
          helperStack.push(curNode.children[i])
        }
      }
    }
    // push到res数组中
    res.push(curArray)
  }

  return res
}
```
#### 大文件分片渲染
大文件分片的背景：传输视频时，上传过程出现网络波动需要重新开始。上传时间过长影响用户体验。

整体流程：文件分片计算hash值 -> 向后端发送请求，询问文件状态 => 根据文件状态进行后续上传 => 文件分片全部上传后，通知服务端整合文件

文件状态有三种
- 文件已经上传过了 => 直接显示上传完成，即秒传
- 文件存在但分片不完整 => 断点续传
- 文件不存在 => 将文件分片完整上传

点击上传后，触发上传的回调函数
```js
async function uploadFile(file, baseChunkSize, uploadUrl, verifyUrl, mergeUrl, process_cb) {
	// 1. 将文件进行分片并计算Hash值
	const { chunkList, fileHash } = await sliceFile(file, baseChunkSize);
	let allChunkList = chunkList;
	// 上传进度
	let progress = 0;

	// 发送请求,获取文件上传状态
	if (vertifyUrl) {
		const { data } = await requestInstance.post(vertifyUrl, {
		  fileHash,
		  totalCount: allChunkList.length,
		  extname: '.' + file.name.split('.').pop(),
		});
		const { neededFileList, message } = data;
		if (message) console.info(message);
		//无待上传文件，秒传
		if (!neededFileList.length) {
		  progress_cb(100);
		  return;
		}
	
		//部分上传成功，更新unUploadChunkList
		neededChunkList = neededFileList;
	}
	// 3. 同步上传进度(断点续传的情况下)，针对不同文件上传状态调用 progress_cb
	progress = ((allChunkList.length - neededChunkList.length) / allChunkList.length) * 100;
	
	//上传
	if (allChunkList.length) {
		const requestList = allChunkList.map(async (chunk, index) => {
		  if (neededChunkList.includes(index + 1)) {
			const response = await uploadChunk(chunk, index + 1, fileHash, uploadUrl);
			//更新进度
			progress += Math.ceil(100 / allChunkList.length);
			if (progress >= 100) progress = 100;
			progress_cb(progress);
			return response;
		  }
		});
		Promise.all(requestList).then(() => {
		  //发送请求，通知后端进行合并 //后缀名可通过其他方式获取，这里以.mp4为例
		  requestInstance.post(mergeUrl, { fileHash, extname: '.mp4' });
		});
	}
}

async function SliceFile(targetFile, baseChunkSize = 1) {
	return new Promise((resolve, reject) => {
		let blobSlice = File.prototype.slice;
		let chunkSize = baseChunkSize * 1024 * 1024;
	    //分片数
	    let targetChunkCount = targetFile && Math.ceil(targetFile.size / chunkSize);
	    // 当前已执行分片数
	    let currentChunkCount = 0;
	    // 当前收集的分片
	    let chunkList = [];
	    // 创建sparkMD5对象
	    let spark = new SparkMD5.ArrayBuffer();
	    //创建文件读取对象
	    let fileReader = new FileReader();
	    let fileHash = null;

		// 读取下一个文件
		const loadNext = () => {
			const start = chunkSize * currentChunkCount;
			const end = start + chunkSize;
			if (end > targetFile.size) {
				end = targetFile.size;
			}
			// 读取文件，触发onLoad
			fileReader.readAsArrayBuffer(blobSlice.call(targetFile, start, end));
		}

		//FilerReader onload事件
	    fileReader.onload = (e) => {
	      //当前读取的分块结果 ArrayBuffer
	      const curChunk = e.target.result;
	      //将当前分块追加到spark对象中
	      spark.append(curChunk);
	      currentChunkCount++;
	      chunkList.push(curChunk);
	      //判断分块是否全部读取成功
	      if (currentChunkCount >= targetChunkCount) {
	        //全部读取，获取文件hash
	        fileHash = spark.end();
	        resolve({ chunkList, fileHash });
	      } else {
	        loadNext();
	      }
	    };

		//FilerReader onerror事件
	    fileReader.onerror = () => {
	      reject(null);
	    };

		loadNext();
	})
}

// 由于我们的请求不仅包含文件，还包含分片索引以及hash值，因此我们的请求体应该是formData，还有一点需要就是此时我们传入的chunk的类型是ArrayBuffer,而formData中文件的类型应该是Blob
/*
	Blob: 表示二进制类型的对象，通常是影像、声音或多媒体文件
	ArrayBuffer: 表储存二进制数据的一段内存，它不能直接读写
*/
async function uploadChunk(chunk, index, fileHash, uploadUrl) {
  let formData = new FormData();
  //注意这里chunk在之前切片之后未ArrayBuffer，而formData接收的数据类型为 blob|string
  formData.append('chunk', new Blob([chunk]));
  formData.append('index', index);
  formData.append('fileHash', fileHash);
  return requestInstance.post(uploadUrl, formData);
}
```
hash计算在前端比较耗时，基于现有大文件分片上传的优化方案：
- hash算法的优化：只针对第一个和最后一个块做整体hash计算，中间块的文件可以只取前后n个字节，避免了hash整个文件所带来的浪费。
- 使用[[#Web Worker|web worker]]实现整体流程，不占用浏览器主线程。
- 断点续传也可以通过将已上传切片保存在localStorage实现。
```js
async function uploadFile(file, baseChunkSize, uploadUrl, verifyUrl, mergeUrl, process_cb) {
	// web worker完成hash和chunkList的获取
	const sliceFileWorker = new Worker(new URL('./sliceFileWorker.js', import.meta.url), { type: 'module' });
	// 给sliceFileWorker发送消息
	sliceFileWorker.postMessage({ targetFile: file, baseChunkSize });
	//分片处理完之后触发onmessage事件
	sliceFileWorker.onmessage = async (e) => {
		//获取处理结果
		const { chunkList, fileHash } = e.data;
		//后续代码与之前一样
		...
	}
}

// sliceFilerWorker.js中代码如下：
self.onmessage = async (e) => {
  //获取文件以及分块大小传输
  const { targetFile, baseChunkSize } = e.data;
  //分片并计算Hash
  const { chunkList, fileHash } = await sliceFile(targetFile, baseChunkSize);
  //将计算结果通过postMessage发送给主线程
  self.postMessage({ chunkList, fileHash });
}

async function sliceFile(targetFile, baseChunkSize = 1) {
    //与之前一样
    ...
}
```
### JS配套技术
#### JS模块化
##### **script 标签 => 原始的模块化**
>将每一个script引入的文件看作是一个**模块**，这会造成：不同模块的接口调用都是在一个文件中，故在原始时期，复杂的项目会使用**命名空间**。
缺点：污染全局作用域、文件必须按照script的书写顺序进行加载。

>**默认情况下：**JS遇到script标签会停下，等待执行完脚本再继续向下渲染。如果是外部脚本，还要加上下载时间
当脚本内容过多时，也可以实现异步加载，即设置defer和async让脚本**异步加载**：遇到这行命令开始下载外部脚本，但不会等待下载and执行结束，而是继续走后面的渲染。

>**defer：**等到整个页面渲染结束才执行，多个脚本按顺序执行**（渲染完执行）**
 **async：**一旦下载完，渲染引擎中断，执行该脚本完毕后再继续渲染，多个脚本不能保证执行顺序**（下载完执行）**
##### CommonJS规范（同步加载模块）
> Node采用CommonJS规范，在服务端模块加载是**运行时同步加载**，在浏览器模块需要**提前编译打包**

- 特点
模块可以被多次加载，但只会在第一次加载时运行，后续会访问第一次缓存的结果。想要模块再运行，必须清缓存

- 基本语法
	**暴露模块**： module.exports = value / exports.xxx = value
	**引入: **require(url / moduleName)


	**CommonJS暴露的是什么？？**
	CJS规定，module代表每个模块对象，加载模块 = 加载该模块的module.exports 属性。需要注意的是，CJS输出的是值的拷贝，也就是说，**一旦输出值，模块内部的变化就影响不到这个值**。
```jsx
// test.js
let counter = 3;
const addCounter = () => { counter += 1 }

module.exports = {
  counter,
  addCounter
}

// index.js
const test = require(./test.js)
console.log(test.counter); // 3
test.addCounter();
console.log(test.counter); // 3 not 4
```

CJS的原理初探
```js
// 缓存变量
var cache = {}
var modules = {
	'./name.js': (module) => {
		module.exports = 'yk' // 给module的exports赋值
	}
}

function require (modulePath) {
	const cacheModule = cache[modulePath];
	// 如果有缓存，返回缓存对象的exports属性
	if (cacheModule !== undefined) return cacheModule.exports; 
	// 令 cache[modulePath] 和 module 指向同一个地址
	module = (cache[modulePath]) = { export : {} };
	// 运行模块内代码
	modules[modulePath](module, module.exports, require);

	return module.exports;
}

// IIFE包裹，避免变量名字冲突
(() => {
  let author = require("./name.js"); // require会返回指定文件的module.exports属性
  console.log(author, "author");
})();
```
##### AMD（异步加载模块）

>由于Node的编程环境是服务器，模块文件一般存于本地硬盘可直接加载。故CommonJS的加载方式是同步的**（加载完成才能执行后面的操作）。**
 但浏览器端必须采用AMD模式异步加载模块，因为同步加载很容易造成阻塞DOM渲染。

- 定义
```jsx
// require([module], fn) => 
// 1. module为数组，成员为要加载的模块
// 2. fn为模块加载完成的callback

// 定义模块
define(function () {
  return { moduleName: outputName}
})

// 定义有依赖模块
define(['m1', 'm2'], function (m1, m2) {
  return { moduleName: outputName}
})
```

- 引入模块
```jsx
require(['m1', 'm2'], function(m1, m2) {
  // 使用m1, m2
})
```
##### CMD（通用模块加载）
##### ES6模块化（编译时加载）
设计思想：**静态化**，从而在编译时就确定模块的依赖关系，以及输入输出变量。**（有利于Tree-Shaking等代码优化手段**）
> 默认情况下，Node不支持ES6模块语法。但Babel可以将import/export => require/exports以兼容

#### MVC、MVP、MVVM
##### MVC框架

- **Model（数据+业务逻辑）：**保存程序的数据以及处理数据的逻辑
- **View（视图层）：**接受用户的交互请求，并向用户展示信息（可能是Model的全部数据，也可以是部分数据或多个Model的组合数据）
- **Controller：**担任Model和View层之前的桥梁，控制程序的流程。作为中间服务，解析View的请求并确保View层可以正确拿到Model层的数据。理论上：Controller是很轻的，内部不应该有大量代码

**好处：**

1. 不同层各司其职，前端只负责页面显示，后端负责controller逻辑处理（**模型与视图完全分离——即解耦**）
##### MVP框架
**演变历史：**最开始MVC的View依赖Model，实际降低了View的可复用性。所以后来View与Model解绑，不再进行通讯，统一由Controller处理(MVP)。
![Snipaste_2023-09-11_13-31-07.png](https://cdn.nlark.com/yuque/0/2023/png/25949356/1694410280588-d064e90d-e593-4f17-bf79-8c2d58f59617.png#averageHue=%23fafaee&clientId=u2b5324c2-3eba-4&from=drop&height=163&id=u4b7e5d3f&originHeight=413&originWidth=726&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=57736&status=done&style=none&taskId=ucbdcf7d6-d976-41c5-a778-907a529bdbf&title=&width=287)
##### MVVM框架
MVP的改进版(即将**Presenter改名为ViewModel**)，二者最大的区别是：实现了**View和Model的自动同步，**即当Model属性改变时，无需手动操作DOM，View会自动更新到页面

#### 包管理器相关
执行npm/yarn install 包是如何到达项目中的呢？
首先，解析包的版本号，下载对应版本的tar到本地离线镜像。
其次，将依赖从离线镜像解压到本地缓存，然后从缓存中拷贝到当前目录的node_modules。
##### npm

>遵循语义版本控制思想：主版本号.次版本号.补丁版本号
使用package.json管理项目依赖


- npm-v2的问题：嵌套依赖，a -> b -> c -> d .... ....
- npm-v3使用扁平化结构解决嵌套的问题，将所有依赖包都提升到顶层。但这个操作需要**遍历一遍所有项目的依赖关系**，十分耗时。本地缓存机制聊胜于无。
- npm-v5: 为了解决扁平化依赖算法耗时长的问题，以及为了保证A包、B包依赖同一个库的不同版本，产生的依赖结构不确定的问题，引入package-lock.json锁定项目的依赖结构，保证依赖稳定性。

> 依然存在的问题：a、b同时依赖不同版本的c，安装哪个？
##### yarn
FaceBook开发的包管理。通过并行下载和缓存加快安装速度；通过yarn-lock锁定版本号确保项目稳定；支持离线下载。总的来说，yarn是为了解决npm不能并行下载和版本锁定的问题。但在npm_v3 & npm_v5之后，也解决了Yarn解决的上述问题。

缺点是，大量包缓存会占用本地磁盘的资源。
##### pnpm
完全不同的依赖处理方式: 将依赖项安装在全局的store种，且每个依赖包的不同版本只安装一次。使用硬链接和软连接将他们关系到每个项目中，不同的项目可以共享全局store中的依赖。为了实现此过程，node_modules下会多出.pnpm目录，该目录为非扁平化结构。
![[Pasted image 20240313213026.png]]
所有的依赖都是从全局 store 硬连接到了 node_modules/.pnpm 下，然后之间通过软链接来相互依赖；即硬链接指向全局store中安装的依赖；软连接找到对应依赖的目录地址。
![[硬链接&软连接.png]]

pnpm也解决了一个比较容易忽视的问题：**依赖包的安全性**。在npm和yarn中， A 依赖 B， B 依赖 C，那么 A 就算没有声明 C 的依赖，由于有依赖提升的存在，C 被装到了 A 的node_modules里面，那我在 A 里面用 C也是可以的。

这里面的潜在问题就是，A非法访问C，当B不再依赖C时，A中引入C的代码会直接报错。npm的解决办法是指定`--global-style`禁止依赖提升，但这样做又回到了嵌套依赖的年代。

```ad-note
Linux链接分两种，一种被称为硬链接（Hard Link -- 对同一个文件的不用引用），另一种被称为符号链接（Symbolic Link -- 新建一个文件，内容指向另一个路径）。默认情况下，ln命令产生硬链接

如下场景： f1硬连到f2, f1 软连到f3
- 删除符号连接f3,对f1,f2无影响
- 删除硬连接f2，对f1,f3也无影响
- 删除原文件f1，对硬连接f2没有影响，导致符号连接f3失效
- 同时删除原文件f1,硬连接f2，整个文件会真正的被删除
```

#### 多包管理方案
##### Lerna

#### SPA和MPA的区别
SPA是单页面应用，在请求资源时，不做分包处理的话会将项目的全部资源下载，故首次加载会慢。但在页面功能切换时，只会做局部刷新。对于路由模式，有hash和history两种，且页面之前的传参方式多样，可以通过路由传值、Vuex、Redux等。SEO可能需要利用服务端渲染来提升。资源文件只需要加载一次。

MPA是多页面应用，在刷新时会整页刷新，网速慢的环境下用户体验差。它的路由模式也比较单一，就是链接跳转的模式。数据传递主要依赖于url带参数、cookie和本地存储的方式。SEO方案较容易实现。每个页面都有自己单独的资源文件。
#### Performance API
web-vitals库获取的用户体验数据可以满足我们量化用户体验的部分需求，但是如果我们有更特殊、更个性化的数据收集需求，就需要使用Performance API。

Performance API 是一组在浏览器平台测量和收集性能数据的接口，它允许开发者访问页面加载、资源获取、用户交互、JavaScript执行等性能相关的数据。
##### 基础性能记录 -- PerformanceEntry
在浏览器中基于 Performance API 获取的各类性能记录都是PerformanceEntry类的子类实例，常用的有三类：
- PerformanceResourceTiming：JS、CSS、Image等各类资源加载相关数据
- PerformanceEventTiming：**首次输入**`"first-input"`相关数据和**慢响应**事件相关数据（lick、input、mousedown 等各类**响应耗时超过104ms的**输入事件，104ms为默认值，可通过`durationThreshold`调整）。
- VisibilityStateEntry：页面可视化状态变化数据指标
```ad-cmt
在 PerformanceResourceTiming类型的记录中，
encodedBodySize：表示服务器响应的，未被压缩（例如Gzip压缩）的响应体大小。
decodedBodySize：表示服务器响应的，已被压缩的响应体大小。
以上2个属性只包括响应体的大小，不包括响应头的大小。 单位均为字节 byte。

transferSize：表示从服务器响应的响应头和响应体的总体积。单位也为字节 byte。
因此，encodedBodySize 和 decodedBodySize 之间的区别在于是否解压缩，而 transferSize 则包括了响应头和响应体的所有内容。
```
下图具体说明了PerformanceResourceTiming各属性的时序关系：
![[Pasted image 20240424114625.png]]
如上图所示，想要获取资源的加载总耗时，就可以找到其性能记录，用responseEnd 减去 fetchStart 得到；计算TCP握手耗时，可以通过connectEnd 减去 connectStart 得到。
##### 自定义性能记录创建
除了浏览器的原生方法，我们还可以通过performance.mark() && performance.measure()方法创建自定义的性能记录。这2个方法，都会创建一个`PerformanceEntry`子类的实例，并保存在当前运行环境的性能记录缓冲区中
```js
// 1. 首先使用.mark方法在性能记录缓冲区中，添加2个性能记录
performance.mark("login-started", {
  detail: { href: location?.href },
});

performance.mark("login-finished", {
  detail: { loginType: 'email' },
});

// 数据结构如下：
{
    detail: { href: '...' }
    name: "login-started",
    entryType: "mark",
    startTime: 4545338.199999809,
    duration: 0
}

// 2. 基于已经添加的2个性能记录，计算这2个记录的间隔时间
performance.measure("login-duration", 
	{ 
		detail: { userRegion: 'cn' },
		start: 'login-started',
		end: 'login-finished', 
	}
);

// 数据结构如下：
{
    detail: { userRegion: '...' }
    name: "login-duration",
    entryType: "measure",
    startTime: 4545338.199999809,
    duration: 275118.69999980927
}
```
##### 获取性能记录的方法
方法一：通过new PerformanceObserver的方式监听式持续获取。该方法的缺点是：已经触发的性能记录，不会被监听到
```js
// 1. 创建一个PerformanceObserver实例
const observer = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  entries.forEach((entry) => {
    console.log(`资源名称: ${entry.name}`);
    console.log(`资源类型: ${entry.initiatorType}`);
    console.log(`资源加载时间: ${entry.duration}ms`);
  });
});

// 2. 指定要观察的性能条目类型（下文为全部类型）
const entryTypes = [       // 对应上文的性能记录子类：
    "resource",            // PerformanceResourceTiming
    "visibility-state",    // VisibilityStateEntry
    "mark",                // PerformanceMark
    "measure",             // PerformanceMeasure
    "event",
    "element",
    "first-input",
    "largest-contentful-paint",
    "layout-shift",
    "longtask",
    "navigation",
    "paint",
    "taskattribution",
];

// 3. 启动 PerformanceObserver 来观察指定类型的性能条目
observer.observe({ entryTypes });

// 4. 停止观察
observer.disconnect();
```
方法二：通过performance.getEntries() / getEntriesByName(nameStr, typeStr) / getEntriesByType(typeStr)来获取性能缓存区中的性能记录，其中ByName和ByType还会通过参数获取指定的性能指标。
```js
performance.getEntries()    // PerformanceEntry: []

performance.getEntriesByName("login-started", "mark");     // PerformanceMark: []

performance.getEntriesByType("resource");    // PerformanceResourceTiming: []
```
##### 前端性能监控实例
```js
// getCDNMetric需要用setTimeout延迟执行，来等待网页性能数据获取完成。
function getCDNMetric() {
	const entries = performance.getEntriesByType('resource');
	entries.forEach((entry) => {
	  const entryName = entry.name;
	  // 计算性能事件
	  const connectionData = getConnectionData(entry);
	  console.log(
		`connectionData=${JSON.stringify(connectionData, null, 2)}`
	  );
	  Object.keys(connectionData).forEach((name) => {
		// 上报性能指标，reportGauge函数中也可通过Math.random的方法，来设置上报的采样率，避免每条数据都上报，从源头减少数据量的产生。
		reportGauge(
		  name,
		  `frontend data of ${name}`,
		  {
			entryName,
			id: Date.now(),
		  },
		  connectionData[name]
		);
	  });
	});
}

// 计算性能的函数
const ResourceTiming = {
	DNSLookupTime: ['domainLookupEnd', 'domainLookupStart'], // DNS寻址耗时
	TCPHandshakeTime: ['connectEnd', 'connectStart'], // TCP握手+TLS
	TLSNegotiationTime: ['requestStart', 'secureConnectionStart'], // 仅TLS握手
	requestToResponseTime: ['responseEnd', 'requestStart'], // 请求到响应的耗时
};
function getConnectionData(entry) {
	let data = {};
	Object.keys(ResourceTiming).forEach((name) => {
	  const [end, start] = ResourceTiming[name];
	  data[name] = entry[end] - entry[start];
	});
	
	return data;
}
```