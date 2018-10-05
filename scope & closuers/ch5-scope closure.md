### 作用域闭包

闭包是依赖于词法作用域编写代码而产生的结果

识别，接纳，并以自己的意志利用闭包

#### 事实真相

> 闭包就是函数能够记住并访问它的词法作用域，即使当这个函数在它的词法作用域之外执行时。

> 闭包 使函数可以继续访问它在编写时被定义的词法作用域

想要清晰地观察到闭包，可以做函数的传递，传递函数到被定义的词法作用域之外，再执行，比如：

```javascript
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 
```

#### 现在我能看到了

在实际开发中，只要将函数作为头等值传来传去，比如：计时器、事件处理器、Ajax请求等，当你传入一个 **callback**，就会在它周围悬挂一些闭包。

demo：

```javascript
function wait(message) {
	setTimeout( function timer(){
		console.log( message );
	}, 1000 );
}

wait( "Hello, closure!" );
```

在 `wait` 执行一千毫秒之后，内部函数 `timer` 依然拥有覆盖着 `wait()` 内部作用域的闭包。在 *引擎* 的内脏深处，内建的工具 `setTimeout(..)` 拥有一些参数的引用，可能称为 `fn` 或者 `func` 或者其他诸如此类的东西。*引擎* 去调用这个函数，它调用我们的内部 `timer` 函数，而词法作用域依然完好无损。所以会输出 `message`

#### 循环+闭包

用来展示闭包最常见的例子是老实巴交的`for`循环：

```javascript
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

结果输出6个6，因为`timer`函数作为值传递给`setTimeout`时，都**闭包在同一个共享的作用域上**，等执行的时候 `i` 已经变成了 `6`。

但是本来的意图是输出 0,1,2,3,4,5，该怎么办呢？

解决思路是**为每一次迭代都准备一个新的闭包的作用域**，解决方法有两种：

1. 使用IIFE创建函数作用域：

   ```javascript
   //低级版
   for (var i=1; i<=5; i++) {
   	(function(){
   		var j = i;
   		setTimeout( function timer(){
   			console.log( j );
   		}, j*1000 );
   	})();
   }
   
   //进阶版
   for (var i=1; i<=5; i++) {
   	(function(j){
   		setTimeout( function timer(){
   			console.log( j );
   		}, j*1000 );
   	})( i );
   }
   ```

2. 块儿作用域：

   ```javascript
   //低级版
   for (var i=1; i<=5; i++) {
   	let j = i; // 呀，给闭包的块儿作用域！
   	setTimeout( function timer(){
   		console.log( j );
   	}, j*1000 );
   }
   
   //进阶版
   for (let i=1; i<=5; i++) {
   	setTimeout( function timer(){
   		console.log( i );
   	}, i*1000 );
   }
   ```

#### 模块

模块利用了闭包的力量，而且不像 callback 那样浮于表面。

```javascript
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

行使模块模式有两个条件：

1. 有一个外围函数，且至少被调用一次
2. 外围函数执行结果，返回一个新的模块实例，模块实例引用内部函数，因为闭包的存在，可以去获取隐藏和私有的内部变量。这个模块实例也被称为 模块的公有API

##### 现代的模块

**模块管理器**实质上是将这种模块定义包装进一个友好的API。

```javascript
var MyModules = (function Manager() {
	var modules = {};	//收集在modules对象

	function define(name, deps, impl) {	
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
        //添加依赖
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {	//获取对应模板快
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

```javascript
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

##### 未来模块

ES6将一个文件视为一个独立的模块。

相比于基于函数的模块，可以在运行时修改模块的API：

```javascript
var foo = (function CoolModule(id) {
	function change() {
		// 修改公有 API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

ES6模块API是静态的，可以被编译器识别，不可以在运行时期修改。