### 函数与块儿作用域

将作用域比作 “气泡”，什么才能制造一个新 “气泡”？也就是，作用域是怎么产生的问题。

#### 函数中的作用域

每一个声明的函数都为自己创建了一个气泡。

#### 隐藏于普通作用域

考虑一个函数的传统方式是，你声明一个函数，并在它内部添加代码。但是相反的想法也挺有意思：拿编写的代码的任意一部分，在它周围包装一个函数声明，制造一个 “气泡” 去 “隐藏” 这段代码。

##### “最低暴露原则”：优化作用域集结构

有一种称为 “最低暴露原则” 的软件设计原则 —— 应当只暴露所需要的最低限度的东西，而隐藏其他的一切。这个原则扩展到 **用哪个作用域来包含变量和函数的选择**。

比如：

```javascript
function doSomething(a) {
	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

function doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething( 2 ); // 15
```

`doSomething` 调用 `doSomethingElse`和`b`，`doSomethingElse`和`b`在全局作用域中，如果被其他代码修改，则会影响 `doSomething`。其实更好的做法是将 `doSomethingElse`和`b`作为`doSomething`的细节：

```javascript
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

符合 “最低权限原则”

##### 发生冲突、避免冲突

例如：

```javascript
function foo() {
	function bar(a) {
		i = 3; // 在外围的for循环的作用域中改变`i`
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // 噢，无限循环！
	}
}

foo();
```

`bar` “气泡”内的 `i=3`，覆盖了 `foo` "气泡"内的`i`值，导致`for`循环无限循环。这在作用域之间产生了一个冲突。声明一个本地变量或者选择另外一个标识符名称，比如 `j=3`可以解决问题：

```javascript
...
function bar(a) {
	var i = 3; 
	console.log( a + i );
    //或者
    // var j = 3;
    // console.log( a + j)
}
...
```
另外冲突有可能发生在加载多个库到程序中，**全局“名称空间”**是创建一个对象，将所有暴露出来的功能作为属性挂在这个对象上，例如：

```javascript
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

**模块管理** 是另一种方法，要求库的标识符被明确地导入到另一个指定的作用域中。（深入研究待定）

#### 函数作为作用域

（<u>在逆向思维，用函数隐藏代码的路上越走越远！！</u>）

在一段代码周围包装一个函数，去隐藏这段代码包含的任何变量或函数声明。比如：

```javascript
var a = 2;

function foo() { // <-- 插入这个

	var a = 3;
	console.log( a ); // 3

} // <-- 和这个
foo(); // <-- 还有这个

console.log( a ); // 2
```

这引入了两个问题：

1. 声明一个命名函数`foo()`，会 “污染” 外围作用域（这里是全局）
2. 为运行这段包装的代码，必须通过名称（`foo()`）明确调用

##### 立即调用函数表达式

解决 “污染” 问题，用函数表达式；解决明确调用问题，用立即调用。合在一起，就是立即调用函数表达式。

先看解决代码：

```javascript
var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

第一个外围的 `( )` 使这个函数变成表达式，而第二个 `()` 执行这个函数。

如果`function`不在第一个位置那么就是一个函数表达式，函数表达式的函数名称作为一个标识符绑定在 `foo` ”气泡“里面，这样就解决 ”污染“ 问题。

**Further：**

- 立即调用函数表达式传参

  ```javascript
  var a = 2;
  
  (function IIFE( global ){
  
  	var a = 3;
  	console.log( a ); // 3
  	console.log( global.a ); // 2
  
  })( window );
  
  console.log( a ); // 2
  ```

- 立即调用函数表达式 变体

  ```javascript
  var a = 2;
  
  (function IIFE( def ){
  	def( window );
  })(function def( global ){
  
  	var a = 3;
  	console.log( a ); // 3
  	console.log( global.a ); // 2
  
  });
  ```

##### 匿名函数表达式

比如：

```javascript
setTimeout( function(){
	console.log("I waited 1 second!");
}, 1000 );
```

匿名函数表达式，就是没有函数名字，优点是方便，缺点：

1. **调试困难**：在栈轨迹上匿名函数没有有用的名称可以表示，这可能会使得调试更加困难。
2. **引用自己困难：**没有名称的情况下，如果这个函数需要为了递归等目的引用它自己，那么就需要很不幸地使用 被废弃的`arguments.callee` 引用。另一个需要自引用的例子是，当一个事件处理器函数在被触发后想要把自己解除绑定。
3. **不易懂：**匿名函数省略的名称经常对提供更易读/易懂的代码很有帮助。一个描述性的名称可以帮助代码自解释。

So...不推荐使用

#### 块儿作为作用域

介绍两种创建块儿作用域的方法，以及解释块儿作用域，相比函数作用域，的优点。

##### `let`

`let` 关键字将变量声明附着在它所在的任何块儿（通常是一个 `{ .. }`）的作用域中。例如：

```javascript
var foo = true;

if (foo) {
	let bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}

console.log( bar ); // ReferenceError
```

为了便于以后重构中将整个块儿四处移动，可以简单引入一个`{..}`：

```javascript
var foo = true;

if (foo) {
	{ // <-- 明确的块儿
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

对比于`var`，`let`不会提升：

```javascript
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

##### `const`

类似于 `let`，`const` 也创建一个 块儿作用域。但是值是固定的。

```javascript
var foo = true;

if (foo) {
	var a = 2;
	const b = 3; // 存在于包含它的`if`作用域中

	a = 3; // 没问题！
	b = 4; // 错误！
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

##### 优点

**闪闪发光的 `for` 循环例子**

对于这样一个`for`循环例子：

```javascript
for (var i=0; i<10; i++) {
	console.log( i );
}
```

在使用`var`时其实是会在外围作用域新建了一个`i`变量，会污染外围作用域。使用 `let` 创建块儿作用域，且每一次循环的迭代都重新绑定`i`：

```javascript
for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError
```

相当于：

```javascript
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // 每次迭代都重新绑定
		console.log( i );
	}
}
```

**垃圾回收**

```javascript
function process(data) {
	// 做些有趣的事
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

`process`之后程序与`someReallyBigData`无关，但是 JS 引擎很可能仍会保留这段巨大数据结构。而使用`let`，块儿作用域之后会做垃圾回收：

```javascript
function process(data) {
	// 做些有趣的事
}

// 运行过后，任何定义在这个块中的东西都可以消失了
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```