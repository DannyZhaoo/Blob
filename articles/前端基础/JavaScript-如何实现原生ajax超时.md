# 如何实现原生ajax超时

涉及内容：
1. 原生XMLHttpRequest是否支持超时
2. 原生Fetch是否支持超时
3. 如果不支持，该如何修改

### 原生XMLHttpRequest
原生XMLHttpRequest是支持timeout的，默认是0，也就是不限制超时，
那么这个时候超时就是根据http连接超时来判断的。

>
> http超时分为两种 
> 1. http请求超时（也就是连接超时）
	就是在一定的时间内还没有连接到服务器，就会认为是超时。
> 2. http响应超时
	就是服务器接到请求，在一定的时间内，还没有对请求端做出回应，那么就会认为是超时。
>

下面写一个例子吧，毕竟自己对原生的XML不太熟悉
```javascript
	// xhr可以重复使用，不用一直创建，可以重新open，然后进行send
	const xhr = new XMLHttpRequest()
	// 一定要在open之前加上progress，否则这个不会触发
	xhr.addEventListener("progress", (e) => {
		if(e.lengthComputable) {
			let percentComplete = e.loaded / e.total * 100
			// ...
		} else {
			// ....如果total是undefined
		}
	}) 
	// 所有请求的返回的数据都已经全部放在response中时触发
	xhr.addEventListener("load", (e) => {
		// ...
	})
	// 有错误时触发, timeout的错误不在此处处理
	xhr.addEventListener("error", (e) => {

	})
	// 被取消触发
	xhr.addEventListener("abort", (e) => {

	})
	// 对跨站的请求是否携带cookie，对同站的没有影响
	xhr.withCredentials = true

	// 设置返回类型
	xhr.responseType =  "json"
	// 
	xhr.onreadystatechange = () => {
		// 0 在创建后，open还没有被调用
		if (xhr.readyState === XMLHttpRequest.UNSENT) {}
		// 1 open已经被调用
		if (xhr.readyState === XMLHttpRequest.OPENED) {}
		// 2 send已经调用了，并且已经收到response的header了
		if (xhr.readyState === XMLHttpRequest.HEADERS_RECEIVED) {}
		// 3 loading  部分response已经收到了
		if (xhr.readyState === XMLHttpRequest.LOADING) {}
		// 4 整个请求阶段已经完成
		if (xhr.readyState === XMLHttpRequest.DONE) {}
	}

	// 通过这个设置请求方法，请求url，请求是否是异步（竟然还可以不是异步的！！！）， 用户（默认是null），密码（默认是null）
	xhr.open('GET', '/public/favicon.ico', true, user, password)
	// 如果是get请求，就不用参数，如果是post请求，也可携带
	xhr.send()
	// xhr请求被中断
	xhr.abort()

```

### 原生fetch

fetch是不支持timeout的，而且相对比XMLHttpRequest有如下不同：
1. fetch不会返回reject对http的错误status，例如404，500
2. fetch 不接收跨站的set-cookie
3. fetch默认不发送cookie，除非设置credential
4. fetch不能设置timeout

如下展示一些常用的用法
```javascript
	var controller = new AbortController()
	var signal = controller.signal
	// 可以直接调用abort, 可以中断请求
	controller.abort()

	fetch('http://example.com/movies.json', {
		method: 'POST',
		mode: 'cors',  // no-cors, *-cors, same-origin
		cache: 'no-cache', // default
		headers: {
			'Content-Type': 'applicatoin/json'
		},
		redirect: 'follow', // manual error 这个比较特殊呢
		referrerPolicy: 'no-referrer',
		credentials: 'include', // same-origin, omit 发送cookies
		body: JSON.stringify(data), // 这里的处理必须和headers中的content-type一致，此处也就是参数的意思
		signal
	})
	.then((response) => {
		if (!respones.ok) {
			throw new Error('Network response was not ok.')
		}
		if (response.status !== 200) {
			throw new Error('http status isnot 200')
		}

		return response.json();
	})
	.then((data) => {
		console.log(data);
  	}).catch(err => {
  		// abort会触发这个错误
  		throw new Error('abort')
  	})


```


### 如何修改

因为XMLHttpRequest已经实现超时接口了，所以这里就记录一下使用fetch的timeout
```javascript
	function timeout (ms, promise) {
		return new Promise(function(resolve, reject) {
			setTimeout(() => {
				reject(new Error("timeout"))
			}, ms)
			promise.then(resolve, reject)
		})
	}

	timeout(1000, fetch(url, ).then((response) => {
		// 
	}).catch(err => {
		// may cache timeout error
	}))

```

