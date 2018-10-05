## 异步：现在与稍后

引入异步：现在执行的程序、稍后执行的程序。稍后执行的程序，比如 定时器、鼠标点击、Ajax 的 callback。这就引入异步的概念。

### 事件轮询

#### 机制

JS 引擎 运行在一个宿主环境中，一个接一个执行 event。宿主环境 为 JS引擎 安排做什么，JS引擎向宿主环境做事件轮询，获知此时此刻该做什么。

从事件轮询队列中取出事件的每次迭代称为 “tick”。

每一次取出的事件，在执行时具有原子性，即这个事件做完才会去取下一个事件。

考虑一下这段代码：

```javascript
var a = 20;

function foo() {
	a = a + 1;
}

function bar() {
	a = a * 2;
}

// ajax(..) 是一个给定的库中的随意Ajax函数
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`foo` 还是 `bar` 先执行是不确定的，这被称为 “竞合状态”

#### `setTimeout` 看事件轮询

`setTimeout` 不会将回调放在事件轮询队列上，它设置一个定时器，当这个定时器时间到了，才会将回调放进事件轮询，这样在未来某个 tick 取出执行。

所以 `setTimeout` 计时器可能不会完美地按预计时间触发。只是得到一个保证，多少时间之后加入到事件轮询队列。这样事件轮询是不可控的。

ES6 明确指出事件轮询应当如何工作，使得有能力对事件轮询队列的排队做直接、细颗粒度的控制。、

### 并行线程

并行和异步的概念是不一样的，JavaScript没有并行。并行最常见的工具是进程与线程，多个线程可以共享一个进程的内存资源。在并行计算时，线程间可能会发生穿插/干扰，需要特殊的步骤来防止。

### 并发

并发是当两个或多个“进程”在同一时间段内同时执行。（这里的“进程”可以理解为“任务”）

比如：一个网站随着用户向下滚动，显示逐步加载的状态更新列表。这里有两个“进程”：

1. 用户向下滚动页面时触发的`onscroll`事件（发起取得新内容的Ajax请求）
2. 将接收返回的Ajax应答（将内容绘制在页面上）

“进程1”：

```javascript
onscroll, request 1
onscroll, request 2
onscroll, request 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
onscroll, request 7
```

"进程2"：

```javascript
response 1
response 2
response 3
response 4
response 5
response 6
response 7
```

一个`onscroll`事件与一个Ajax应答事件很有可能在同一个 *时刻* 都准备好被处理了，在时间线上可能是这样的：

```javascript
onscroll, request 1
onscroll, request 2          response 1
onscroll, request 3          response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6          response 4
onscroll, request 7
response 6
response 5
response 7
```

但是JavaScript一次只能处理一个事件，所以在事件轮询队列可能是这样的：

```javascript
0onscroll, request 1   <--- 进程1开始
onscroll, request 2
response 1            <--- 进程2开始
onscroll, request 3
response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
response 4
onscroll, request 7   <--- 进程1结束
response 6
response 5
response 7            <--- 进程2结束
```

最后的结论是：**单线程事件轮询是并发的一种表达**

#### 非互动

如果多个“进程”穿插时，任务间没有联系，就没必要互动。如果不互动 ，最后时间轮询队列的不确定性，是完全可以接受的。

#### 互动

如果 "进程" 有必要间接地互动，需要协调互动行为来保证 顺序。

比如：

```javascript
var res = [];

function response(data) {
	res.push( data );
}

// ajax(..) 是某个包中任意的Ajax函数
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

如果明确需要`res[0]`拥有`"http://some.url.1"`调用的结果，而`res[1]`拥有`"http://some.url.2"`调用的结果。可以这样：

```javascript
var res = [];

function response(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}

// ajax(..) 是某个包中任意的Ajax函数
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

再比如：

```javascript
var a, b;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(y) {
	b = y * 2;
	baz();
}

function baz() {
	console.log(a + b);
}

// ajax(..) 是某个包中任意的Ajax函数
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

不管 `foo`和`bar` 谁先触发，`baz`运行时结果都为`NaN`。有一种简单的方法是“大门”：

```javascript
var a, b;

function foo(x) {
	a = x * 2;
	if (a && b) {
		baz();
	}
}

function bar(y) {
	b = y * 2;
	if (a && b) {
		baz();
	}
}

function baz() {
	console.log( a + b );
}

// ajax(..) 是某个包中任意的Ajax函数
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

#### 协作

“协作” 的目标是，将一个长时间运行的“进程”打断为许多步骤或批处理，以至于其他的并发“进程”有机会将它们的操作穿插进事件轮询队列。

比如：

```javascript
var res = [];

// `response(..)`从Ajax调用收到一个结果数组
function response(data) {
	// 我们一次只处理1000件
	var chunk = data.splice( 0, 1000 );

	// 连接到既存的`res`数组上
	res = res.concat(
		// 制造一个新的变形过的数组，所有的`data`值都翻倍
		chunk.map( function(val){
			return val * 2;
		} )
	);

	// 还有东西要处理吗？
	if (data.length > 0) {
		// 异步规划下一个批处理
		setTimeout( function(){
			response( data );
		}, 0 );
	}
}

// ajax(..) 是某个包中任意的Ajax函数
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

协作不能保证顺序，若需要保证顺序，还得考虑互动。

### Jobs

ES6，在事件轮询队列之上，引入了 “工作队列” 的概念。

“工作队列”是一个挂靠在事件轮询队列的每个tick末尾的队列。在事件轮询的一个tick期间内，某些可能发生的隐含异步动作的行为将不会导致一个全新的事件加入事件轮询队列，而是在当前tick的工作队列的末尾加入一个新的记录（也就是一个Job）。

