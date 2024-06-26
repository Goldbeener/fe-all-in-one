```js
await expression

/***
expression 
可以是一个promise
或者是一个thenable对象
*/

thenable 对象本质是一个对象，只是有一个then方法，接受两个回调函数
分别处理成功和失败

const aThenAble = {
	then(onFullfilled, onRejected) {
		// 
		onFullfilled()
	}
}
```

await 后面可以跟一个thenable对象
利用这个特性可以实现一个函数，直接调用或者await调用

```js
function useRequest(requestFn) {
	const data = ref();
	const loading = ref(false);

	const fetch = async () => {
		loading.value = true
		const res = await requestFn()
		loading.value = false
		data.value = res
	}

	const shell = {
		data,
		loading,
	}

	function waitUntilLoaded() {
		return new Promise((resolve, reject) => {
			until(shell.loading)
				.toBe(false)
				.then(() => resolve(shell))
				.catch(reject)
		})
	}

	return {
		...shell,
		then(onFullfilled, onRejected) {
			return waitUntilLoaded().then(onFullfilled, onRejected)
		}
	}
}
```