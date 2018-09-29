##### 解构赋值—对象解构

`var { a , b } = ..`是ES6解构赋值的一种形式，比如：

```javascript
function getData() {
	// ..
	return {
		a: 42,
		b: "foo"
	};
}

var { a, b } = getData();

console.log( a, b ); // 42 "foo"
```

对象解构也可用于被命名的函数参数，这时它是同种类的隐含对象属性赋值的语法糖：

```javascript
function foo({ a, b, c }) {
	// 不再需要：
	// var a = obj.a, b = obj.b, c = obj.c
	console.log( a, b, c );
}

foo( {
	c: [1,2,3],
	a: 42,
	b: "foo"
} );	// 42 "foo" [1, 2, 3]
```

注意平时使用！

##### 短接

对于`&&`和`||`两个操作符来说，如果左手边的操作数足够确定操作的结果，那么右手边的操作数将 **不会被求值**。故而，有了“短接”这个名字。

例如，说`a && b`，如果`a`是falsy`b`就不会被求值，因为`&&`操作数的结果已经确定了，所以再去麻烦地检查`b`是没有意义的。同样的，说`a || b`，如果`a`是truthy，那么操作的结果就已经确定了，所以没有理由再去检查`b`。

比如常见的在某个对象存在情况下，去使用对象的某个属性：

```javascript
function doSomething(opts) {
	if (opts && opts.cool) {
		// ..
	}
}
```

相似地可以用`||`短接，类似于首选方案和其次方案：

```javascript
function doSomething(opts) {
	if (opts.cache || primeCache()) {
		// ..
	}
}
```

##### 对于操作符优先级的态度

在操作符优先级/结合性可以使代码更短更干净的地方使用操作符优先级/结合性，在( )手动分组可以帮你创建更清晰的代码并减少困惑的地方使用( )手动分组

##### ES6 函数默认参数值

如果你省略一个参数，或者你在它的位置上传递一个`undefined`值的话，就会应用这个默认值。

```javascript
function foo( a = 42, b = a + 1 ) {
	console.log( a, b );
}

foo();					// 42 43
foo( undefined );		// 42 43
foo( 5 );				// 5 6
foo( void 0, 7 );		// 42 7
foo( null );			// null 1
```

##### switch 

默认匹配为严格等价，但是可以模拟宽松等价的行为

```javascript
var a = "42";

switch (true) {
	case a == 10:
		console.log( "10 or '10'" );
		break;
	case a == 42:
		console.log( "42 or '42'" );
		break;
	default:
		// 永远不会运行到这里
}
// 42 or '42'
```

