## 文法

文法，用来描述语法如何组合在一起形成结构良好，合法的程序。

### 语句与表达式

> 英语：一个“句子（sentence）”是一个表达想法的词汇的完整构造。它由一个或多个“短语（phrase）”组成，它们每一个都可以用标点符号或连词（“和”，“或”等等）连接。一个短语本身可以由更小的短语组成。一些短语是不完整的，而且本身没有太多含义，而另一些短语可以自成一句。这些规则总体地称为英语的 *文法*。

JavaScript文法也类似。语句就是句子，表达式就是短语，而操作符就是连词/标点。

#### 语句完成值

控制台仅仅报告语句的完成值，比如出现很多的`undefined`。

```javascript
var b;

if (true) {
	b = 4 + 38;
}
```

如果将这段代码敲入控制台，你会看到它报告`42`，因为`42`是`if`块儿的完成值，它取自`if`的最后一个复制表达式语句`b = 4 + 38`。换句话说，一个块儿的完成值就像 *隐含地返回* 块儿中最后一个语句的值。

这个值是无法直接捕获的：

```javascript
var a, b;

a = if (true) {			//Uncaught SyntaxError: Unexpected token if
	b = 4 + 38;
};
```

捕获的方法：`eval(...)`(丑拒)，ES7提案`do{...}`（未来可期）

#### 表达式副作用

比如`++`递增操作符和`--`递减操作符：

```javascript
var a = 42;
var b = a++;

a;	// 43
b;	// 42
```

在返回`42`给`b`的同时，改变`a`的值为`43`；

`++a++`会得到一个`ReferenceError`，首先`a++`被求值，返回`43`，然后`++43`导致报错，因为副作用操作符要求一个变量引用作为副作用的目标。

**顺便了解一下`,`操作符**：

```javascript
var a = 42, b;
b = ( a++, a );

a;	// 43
b;	// 43
```

比如`delete`，在这里表达式返回`true`，副作用删除属性：

```javascript
var obj = {
	a: 42
};

obj.a;			// 42
delete obj.a;	// true
obj.a;			// undefined
```

比如`=`赋值操作符，在这里`a=2`表达式返回`42`，副作用是做了对`a`的赋值：

```javascript
var a;

a = 42;		// 42
a;			// 42
```

利用这一点可以：

```javascript
var a, b, c;

a = b = c = 42;
```

还可以通过将两个`if`语句组合为一个来进行简化

```javascript
/*function vowels(str) {
	var matches;

	if (str) {
		// 找出所有的元音字母
		matches = str.match( /[aeiou]/g );

		if (matches) {
			return matches;
		}
	}
}

vowels( "Hello World" ); // ["e","o","o"]*/

function vowels(str) {
	var matches;

	// 找出所有的元音字母
	if (str && (matches = str.match( /[aeiou]/g ))) {
		return matches;
	}
}

vowels( "Hello World" ); // ["e","o","o"]
```

#### 上下文规则

在JavaScript文法规则中有好几个地方，同样的语法根据它们被使用的地方/方式不同意味着不同的东西。这样的东西可能，孤立的看，导致相当多的困惑。

##### `{..}`大括号

###### 对象字面量

对象字面量是长这样的：

```javascript
// 假定有一个函数`bar()`的定义

var a = {
	foo: bar()
};
```

如果去掉上面代码的`var a =`，变成：

```
// 假定有一个函数`bar()`的定义

{
	foo: bar()
}
```

`{..}`是一个独立的块儿，`foo:bar()`称为”打标签的语句“，`foo`是`bar()`语句的标签。可以使用`break`对循环、非循环标签跳转，使用`continue`对循环标签跳转。

###### 块儿

说到`{..}`块儿，有一个为人诟病的JS坑：

```javascript
[] + {}; // "[object Object]"
{} + []; // 0
```

第一行`[]+{}`是一个`+`操作符的表达式，`[]`被强制转换为`""`，`{}`被强制转换为`"[object Object]"`，所以字符串拼接结果为`"[object Object]"`；

第二行`{}`被翻译为一个独立的空代码块儿，什么也不做，`{}+[]`相当于`+[]`，`[]`先被强制转换为`""`，再被强制转换为`0`，所以结果为`0`；

###### 对象解构(解构赋值)

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

##### `else if`和可选块儿

`if`和`else`语句后面的代码块儿仅包含一个语句时，`if`和`else`语句允许省略这些代码块儿周围的`{ }`：

```javascript
if (a) doSomething( a );
//等价于
if (a) { doSomething( a ); }
```

对于我们常见的`else if`：

```javascript
if (a) {
	// ..
}
else if (b) {
	// ..
}
else {
	// ..
}
```

JS文法其实没有`else if`，这就是上面提到的`else`语句后面仅包含一个语句的情况，相当于：

```javascript
if (a) {
	// ..
}
else {
	if (b) {
		// ..
	}
	else {
		// ..
	}
}
```

了解是怎么回事就好拉！

### 操作符优先级

操作符优先级：当在一个表达式中有多于一个操作符时，什么样的规则统治着操作符被处理的方式

比如：

```javascript
var a = 42, b;
b = ( a++, a );

a;	// 43
b;	// 43
```

现在如果去除`()`：

```javascript
var a = 42, b;
b = a++, a;

a;	// 43
b;	// 42
```

`b`的赋值发生了变化。因为`,`操作符要比`=`操作符的优先级低。所以，`b = a++, a`被翻译为`(b = a++), a`，所以结果`b`结果变成了`42`。

分析一个**极为复杂**的例子：

```javascript
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

d;		// ??
```

#### 短接

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

#### 更紧密的绑定

```javascript
var d = a && b || c ? c || b ? a : c && b : a
// 等价于var d =(a && b || c) ? (c || b) ? a : (c && b) : a 
```

因为`&&`优先级比`||`高，而`||`优先级比`? :`高。另一种常见的解释方式是，`&&`和`||`要比`? :`“结合的更紧密”。

#### 结合性

更紧密的绑定说明了优先级的高低决定了执行的先后顺序。但如果对于**多个同等优先级的操作符**呢？它们总是从左到右或是从右到左地处理吗？这就是结合性的问题。

结合性分左结合（从左到右组队）和右结合（从右到左组队）两类。

- 考虑`&&`和`||`操作符：

  ```javascript
  a && b && c;	//等价 (a&&b)&&c
  a || b || c;	//等价 (a||b)||c
  ```

  `&&`和`||`是左结合的。

- 考虑`?:`三元操作符：
  ```javascript
  a ? b : c ? d : e;	//等价 a ? b : (c ? d : e)
  ```

  `? : `三元操作符是右结合的。

- 考虑`=`赋值操作：

  ```javascript
  var a, b, c;
  
  a = b = c = 42;
  
  //等价 a = (b = (c = 42))
  ```

  `=`赋值操作符是右结合的

回到那个复杂的例子，在更紧密的绑定的基础上，得出最后的分组结果：

```javascript
var a = 42;
var b = "foo";
var c = false;

var d = a && b || c ? c || b ? a : c && b : a;

//等价于	((a && b) || c) ? ((c || b) ? a : (c && b)) : a
//答案就是a
d;	//42
```

#### 消除歧义

对于操作符优先级的态度：

**在操作符优先级/结合性可以使代码更短更干净的地方使用操作符优先级/结合性，在( )手动分组可以帮你创建更清晰的代码并减少困惑的地方使用( )手动分组**

### 错误

JavaScript拥有不同的错误子类型（`TypeError`、`ReferenceError`、`SyntaxError`等等）。

错误分在运行时期发生的错误、在编译时被强制执行的特定错误（也被称为"早期错误"）。

只有运行时发生的错误才可以`try...catch`捕获。

“早期错误”：

1. 任何直接的语法错误都是一个早期错误（例如，`a = ,`）

2. 文法还定义了一些语法上合法但是无论怎样都不允许的东西

   ```javascript
   var a = /+foo/;	
   
   var b;
   42 = b;
   
   function foo(a,b,a) { }					// 还好
   function bar(a,b,a) { "use strict"; }	// 错误！
   
   (function(){
   	"use strict";
   
   	var a = {
   		b: 42,
   		b: 43
   	};			// 错误！
   })();
   ```

#### TDZ(时间死区)

TDZ指的是代码中还不能使用变量引用的地方，因为它还没有到完成它所必须的初始化。

最明白的例子是ES6的`let`块儿作用域：

```javascript
{
	a = 2;		// ReferenceError!
	let a;
}
```

赋值`a = 2`在变量`a`（它确实是在`{ .. }`块儿作用域中）被声明`let a`初始化之前就访问它，所以`a`位于TDZ中并抛出一个错误。

`typeof`对于未声明的变量是安全的，对于TDZ的引用没有这样的安全例外：

```javascript
{
	typeof a;	// undefined
	typeof b;	// ReferenceError! (TDZ)
	let b;
}
```

#### 函数参数值

这里也有一个TDZ的例子：

```javascript
var b = 3;

function foo( a = 42, b = a + b + 5 ) {
	console.log(a,b);
}

foo();	//ReferenceError: b is not defined
```

因为`b = a + b + 5`这里的第二个`b`不会拉到外面的`b`引用，在参数`b`的TDZ内，所以`ReferenceError`啦！



对于ES6的参数默认值，如果你省略一个参数，或者你在它的位置上传递一个`undefined`值的话，就会应用这个默认值。

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

那如果想要区分是`undefined`还是省略导致使用参数默认值，就得使用`arguments`。`arguments`是个伪数组，对于明确传入`undefined`,`arguments`会为这个参数存在一个元素。

如果你传递一个参数，`arguments`的值槽和命名的参数在非严格模式下总是**链接**到同一个值上。所以改变命名的参数也会影响`arguments`的值槽，比如这里的`a`：（要小心！）

```javascript
function foo(a) {
	a = 42;
	console.log( arguments[0] );
}

foo( 2 );	// 42 (链接了)
foo();		// undefined (没链接)
```

### `try...finally`

其实完整的结构是这样的`try...(catch...)finally...`。`finally`子句在`try`（和`catch`，如果存在的话）完成后立即运行，在其他任何代码之前。

1. 如果`try`子句内有一个`return`语句，调用端代码是在`finally`执行之后才收到这个值：（顺序）

   ```javascript
   function foo() {
   	try {
   		return 42;
   	}
   	finally {
   		console.log( "Hello" );
   	}
   
   	console.log( "never runs" );
   }
   
   console.log( foo() );
   // Hello
   // 42
   ```

   如果一个异常从`finally`子句中被抛出（偶然地或有意地），它将会作为这个函数的主要完成值进行覆盖。如果`try`块儿中的前一个`return`已经设置好了这个函数的完成值，那么这个值就会被抛弃。

   相似`return`的，还有`throw`、`continue`和`break`。

2. 在`finally`内部的`return`有着覆盖前一个`try`或`catch`子句中的`return`的特殊能力，但是仅在`return`被明确调用的情况下：

   ```javascript
   function foo() {
   	try {
   		return 42;
   	}
   	finally {
   		// 这里没有 `return ..`，所以返回值不会被覆盖
   	}
   }
   
   function bar() {
   	try {
   		return 42;
   	}
   	finally {
   		// 覆盖前面的 `return 42`
   		return;
   	}
   }
   
   function baz() {
   	try {
   		return 42;
   	}
   	finally {
   		// 覆盖前面的 `return 42`
   		return "Hello";
   	}
   }
   
   foo();	// 42
   bar();	// undefined
   baz();	// "Hello"
   ```

3. 一个`finally` + 打了标签的`break`实质上取消了`return`

   ```javascript
   function foo() {
   	bar: {
   		try {
   			return 42;
   		}
   		finally {
   			// 跳出标记为`bar`的块儿
   			break bar;
   		}
   	}
   
   	console.log( "Crazy" );
   
   	return "Hello";
   }
   
   console.log( foo() );
   // Crazy
   // Hello
   ```

   Don't do it......

### `switch`

在`switch`中，判断表达式与`case`表达式进行匹配时是严格等价。

但是可以模拟宽松等价的行为：

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

额外解释一个demo：

```javascript
var a = 10;

switch (a) {
	case 1:
	case 2:
		// 永远不会运行到这里
	default:
		console.log( "default" );
	case 3:
		console.log( "3" );
		break;
	case 4:
		console.log( "4" );
}
// default
// 3
```

1. `default`未必是放最后的，当然放最后更容易理解
2. 这里分别输出`default`和`3`的理由，首先表达式`a`去匹配每一个`case`，因为匹配失败，所以`default`情况输出`default`；接着函数继续往下执行，输出`3`，`break`跳出`switch`





参考文献

https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/types%20%26%20grammar/ch5.md