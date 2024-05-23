### 返回runner
> 如何实现下列效果？
```ts
let foo = 10;
const runner = effect(() => {
	foo++;
	return 'foo';
})
expect(foo).toBe(11); // 第一次执行effect内部fn的foo++

// 执行runner = 执行effect内部的fn
const res = runner();
console.log(res) // 'foo'
expect(foo).toBe(12);
```
参考最初版本的[[1- Proxy与响应式对象#副作用是如何被创建的|副作用创建]]
```ts
function effect(fn) {

	// 创建并执行RE对象
	const _effect = new ReactiveEffect(fn);
	_effect.run();_
	
	// 返回runner
	const runner: any = _effect.run.bind(_effect);
	
	return runner;
}
```
### scheduler功能
scheduler为函数，obj.foo改变时，要求执行scheduler而不是传入effect的fn
```jsx
const runner = effect(() => {
	dummy = obj.foo;
}, { scheduler })

function scheduler() {
	console.log('scheduler');
}

obj.foo++; // scheduler调用
```
修改effect如下：
```ts
function effect(fn, options) {

	// 创建并执行RE对象
	const _effect = new ReactiveEffect(fn, options);
	_effect.run();_
	
	// 返回runner
	const runner: any = _effect.run.bind(_effect);
	
	return runner;
}

class ReactiveEffect{
	pricate fn;
	constructor(fn, public scheduler?) {
		this._fn = fn;
		this.scheduler = scheduler; // 为activeEffect对象绑定scheduler属性
	}
}
```
接下来，只需在trigger时判断effect.scheduler是否存在即可。存在则执行，不存在执行this._fn
### stop 与 onStop 功能
执行effect时会返回runner，调用stop(runner)使得effect的副作用消失。但如果手动调用runner的话，仍旧会触发副作用。单测如下：
```js
it('stop', () => {
	const obj = reactive({ prop: 1 });
	let dummy;
	const runner = effect(() => {
		dummy = obj.prop;
	})

	obj.prop = 2;
	expect(dummy).toBe(2);
	stop(runner);
	
	obj.prop = 3;
	expect(dummy).toBe(2);

	runner();
	expect(dummy).toBe(3);
})
```
**实现思路**：将effect注册在deps中的的副作用删除。

首先，需要将effect对象挂载到runner上。
```js
function effect (fn, options) {
	...
	const runner: any = _effect.run.bind(_effect);
	runner.effect = _effect;

	return runner;
}
```
执行stop(runner)时，通过runner访问到对应的ReactiveEffect对象。执行其stop函数。
```js
function stop(runner) {
	runner.effect.stop();
}

class ReactiveEffect {
	...
	stop() {
		// 调用cleanupEffect清除副作用
		cleanupEffect(this);
	}
}

cleanupEffect(effect) {
	effect.deps.forEach((dep: any) => {
		dep.delete(effect);
	})
}
```
onStop功能：作为effect第二个参数options的属性传入。stop(runner)运行时，执行onStop函数。
```js
it('onStop', () => {
	const obj = reactive({ prop: 1 });
	let dummy;
	const onStop = jest.fn();
	
	const runner = effect(() => {
		dummy = obj.prop;
	}, {
		onStop
	})

	stop(runner);
	expect(onStop).toHaveBeenCalledTimes(1);
})
```
实现思路：在RE对象内部的stop函数执行时，内部同时调用onStop
```js
class ReactiveEffect {
	onStop?: () => void 
	...
	stop() {
		cleanupEffect(this);
		this.onStop && this.onStop();
	}
}

function effect (fn, options) {
	const _effect = new ReactiveEffect(fn, options.scheduler);
	// 将options的属性挂载到RE对象上，包括onStop函数
	Object.assign(_effect, options);
	...
}
```
### stop的优化