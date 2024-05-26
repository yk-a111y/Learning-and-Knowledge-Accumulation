##  Proxy对象是什么
>  **proxy**( raw, handler )： 为raw创建一个代理对象，在handler中实现对raw基本操作的拦截及自定义(访问属性、函数调用等。)
## 响应式对象是什么
> 通过reactive函数创建Proxy对象，并通过定制handler实现特定的响应式效果
```jsx
function reactive(raw) {
	return new Proxy(raw, customHandler);
}

// 一般响应式对象的handler
mutableHandler = {
	get: function getter(target, key) {
		const res = Reflect.get(target, key)
		// 收集依赖
		track(target, key);
		return res;
	},
	set: function setter(target, key, val) {
		const res = Reflect.set(target, key, val);
		// 触发依赖
		trigger(target, key);
		return res;
	}
}

// readonly对象的handler
readonlyHandler = {
	get: function getter() {
		const res = Reflect.get(target, key)
		// readonly不收集依赖
		// track(target, key);
		return res;
	},
	set: function setter() {
		// 由于是readonly，不能set，故在setter逻辑中写错误提示信息
		console.erroe('error msg')
	}
}
```
## 副作用是如何被创建的
> effect函数用来创建副作用，即一个响应式对象内容改变时，其他与这个响应式对象连接的变量都会更新
```js
// 响应式对象user
const user = reactive({ age: 10 });

let nextAge;
// 副作用：将user.age的变化与nextAge连接起来
effect(() => {
	nextAge = user.age + 1;
})

// 响应式对象变化时，会执行副作用函数，引起与其连接的变量变化
user.age = 11;
expect(nextAge).toBe(12);
```
- **effect函数是如何实现的？**
```js
function effect(fn) {
	// 创建一个ReactiveEffect类，并调用其run方法
	const _effect = new ReactiveEffect(fn);
	_effect.run();
}
```
其中，ReactiveEffect的结构如下：
```js
class ReactiveEffect {
	private _fn;
	constructor(fn) {
		this._fn = fn;
	}
	// run方法，执行传入的副作用函数
	run() {
		// 当前执行的effect对象
		activeEffect = this;
		// return runner
		return this._fn();
	}
}
```
如此一来，在effect函数调用时，传入的副作用会被执行一次。那么，后续依赖项更新时，副作用怎样才能随后执行呢？[[2- 收集依赖与触发依赖|收集依赖与触发依赖]]
