# promise

````
>setTimeout(()=>{console.log("我才是第一");},0);
>console.log("我是第一");


因为setTimeout是异步的事件，所以主线程把它调入Event Loop线程进行注册。
主线程继续执行console.log("我是第一");
主线程执行完毕，从Event Loop 线程读取回调函数。再执行console.log("我才是第一");

````

```
javascript是一门单线程语言，而浏览器是多线程的，所以为了执行不同的同步异步的代码，JavaScript运行的环境采用里事件循环和消息队列来达到目的。
javascript的执行机制Event Loop,

任务类型的区分:
macro-task大概包括：script(整体代码), setTimeout, setInterval, setImmediate, I/O, UI rendering。
micro-task大概包括: process.nextTick, Promise, Object.observe(已废弃), MutationObserver(html5新特性)
setTimeout/Promise等我们称之为任务源。而进入任务队列的是他们指定的具体执行任务（回调函数）。


同步任务和异步任务会被分发到不同的线程去执行。

执行一个宏任务（栈中没有就从事件队列中获取）
执行过程中如果遇到微任务，就将它添加到微任务的任务队列中
宏任务执行完毕后，立即执行当前微任务队列中的所有微任务（依次执行）
当前宏任务执行完毕，开始检查渲染，然后GUI线程接管渲染
渲染完毕后，JS线程继续接管，开始下一个宏任务（从事件队列中获取）

以上这部分，涉及太多专有名词，我也不懂，很花时间的，
主要就一个词，事件循环机制(Event Loop)
```

```
js的执行是异步单线程的，它的问题就是没办法控制先后执行顺序

要严格顺序执行，可以：
	回调函数；
	promise / yield ；
	async / await ( 需要 es5 兼容 )；（async是Generator的语法糖）
	引入 bluebird，then,async.js 库；

回调用的最多，原理就是把a函数的名称做为变量传给b函数，并在b函数内调用，这个过程就叫回调。
function a(){}
function b(fn){
    ...其它代码
    fn && fn();
}
b(a)
有一个传参的问题，如图
```



```

为什么是promise?
一，promise主要是串行回调，也是回调地狱 的问题 
第二个是settimeout不可控，不好维护，能帮助解决函数的先后执行问题（粗糙理解为异步代码同步化的操作）
第三，某种情形下他是一种很好的解决方案(高德）
第四，是过度到async和await
（await等的就是一个Promise。如果等的不是Promise，那加了await和不加没区别）
（ES7的async/await关键词，他们是更深入的将promises集成进语言中的）
五，Axios 是一个基于 promise 的 HTTP 库,

个人认为，学promise最快的方式，自己定义一个，实际调用一下
```



````
前公司封装的一个方法（学习写法）
  $rootScope.globalAction.promiseCall = function(params, api){
            var deferred = $q.defer();
            SignService.sign(params)
                .then(function(result){
                    $http.jsonp(ApiUrlBuilder(api), {"params": result})
                        .success(function(response){
                            if (response.status == "success") {
                                deferred.resolve(response);
                            } else {
                                deferred.reject(response.info);
                            }
                        })
                        .error(function(error){
                            deferred.reject(error);
                        });
                });
            return deferred.promise;
        };
        jq的defer,但是他们和promise不太一样，如果有写这个方法的，出现问题，自己去找原因
        
````

```
我这写的

	function getMobile(){
		$.ajax({
			type:'POST',
			url: '../customer/getMobile',
			contentType:'application/json;charset=UTF-8',
			beforeSend:function(){ 
			},
			success:function(result){

			},
			cache: false
		});
	}
	
function phoneStatus(){
	return new Promise(function(resolve,reject){
		$.ajax({
			type:'POST',
			url: '../customer/getMobile',
			contentType:'application/json;charset=UTF-8',
			success:function(result){
				resolve(result)
			},
			error:function(err){
				reject(err)
			}
		});
	})
}
调用
phoneStatus().then(res={
    console.log(res)
}).catch(err=>{
    console.log(err)
})

```



```

平推式的写法：快速创建promise对象
Promise.resolve('foo') 等价于 new Promise(resolve => resolve('foo'))



写法上的问题
somePromise().then(function () {
  // I'm inside a then() function!
});

1、返回另一个promise；
2、返回一个同步值（或者undefined）；
3、抛出一个同步错误。




somePromise().then(function () {
  return XXX;
}).then(res=>{
    console.log(res)
})

没返回值就会穿透，有副作用 
Promise.resolve('foo').then(Promise.resolve('bar')).then(function (result) {
  console.log(result);
});

不能加第二个参数
somePromise().then(function () {
  return someOtherPromise();
}, function (err) {
  // handle error
});
等同于
somePromise().then(res={
    console.log(res)
}).catch(err=>{
    console.log(err)
})

```

```
多个promise 并行
var p= Promise.all([p1,p2,p3]),p1,p2,p3都是resolved,p是resolved,有一个是reject，均为reject,近似于且的关系

Promise.all([checkLogin(),getUserInfo()]).then(([res1,res2])=>{
  console.log(`result1:${res1.result},result2:${res2.userId}`)
})
```

```
一个问题 
phoneStatus().then(function(res){
			
	}).then(function(res){			

	}).catch(err=>{
		console.log(err)
	})
	
	[ERROR] 5:10:missing name after . operator
	
	phoneStatus().then(function(res){
			
	}).then(function(res){			

	})['catch'](function(err){
		console.log(err)
	})
```

```
一定要加返回值，Return (值的穿透问题 )
一定不要给他加第二个参数
一定要加catch

两个基本点，会定义和会使用
```

> promise a+规范，或者自己实现一个promise

<https://www.jianshu.com/p/aab82c355ac4>