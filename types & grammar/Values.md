### 值

`array`、`string`、和 `number` 是任何程序的最基础构建块，JavaScript 在这些类型上有一些或使你惊喜或使你惊讶的独特性质。

让我们来看几种 JS 内建的值类型，并探讨一下我们如何才能更加全面地理解并正确地利用它们的行为。

#### `Array`

1. js中`array` 只是值的容器，值可以是任何类型，也不需要预先指定`array`的大小。

2. 小心创建稀散数组，空槽值表现的`undefined`和指定值为`undefined`是不一样的：

   ```javascript
   var a = [ ];
   
   a[0] = 1;
   // 这里没有设置值槽 `a[1]`
   a[2] = [ 3 ];
   
   a[1];		// undefined
   
   a.length;	// 3
   ```

3. `array`也是对象，所以可以添加`string`键/属性，然而，，如果一个可以被强制转换为10进制 `number` 的 `string` 值被用作键的话，它会认为你想使用 `number` 索引而不是一个 `string` 键：

   ```javascript
   var a = [ ];
   
   a["13"] = 42;
   
   a.length; // 14
   ```

   一般来说，向 `array` 添加 `string` 键/属性不是一个好主意

##### 类 Array

类 Array，比如DOM查询操作返回的DOM元素列表、函数为了像列表一样访问参数值而暴露的`arguments`对象。类 Array 有 `length`属性、`forEach`方法。

有时候想把类 `array` 值转化为 一个真正的 `array`，有两个办法：

1. `Array.prototype.slice.call`

   ```javascript
   function foo() {
   	var arr = Array.prototype.slice.call( arguments );
   	arr.push( "bam" );
   	console.log( arr );
   }
   
   foo( "bar", "baz" ); // ["bar","baz","bam"]
   ```

2.  `Array.from(..)`

   ```javascript
   ...
   var arr = Array.from( arguments );
   ...
   ```

#### `String`

有一种观点说 `string` 实质上是字符的 `array`，比如`"foo"` 就是`["f","o","o"]`。因为`string`确实与`array`有很肤浅的相似性：

- 都有`length`属性
- 都有`indexOf`方法
- 都有`concat`方法

举个例子：

https://jsbin.com/wemolunime/edit?html,js,console



这种观点是不确切的。关键点在于`string`是不可变的，而`array`是相当可变的。换句话说，`string#fn`总是返回新的`string`，`array#fn`有返回新的`array`，也有相当一部分是数组原地修改。比如：

```javascript
c = a.toUpperCase();
a === c;	// false
a;			// "foo"
c;			// "FOO"

b.push( "!" );
b;			// ["f","O","o","!"]
```



另外，操作`string`时，可以借用**非变化**的`array`方法：

https://jsbin.com/yixazuhanu/edit?html,js,console

翻转一个字符串需要`str.split("").reverse().join("")`先转换为`array`，翻转，再转换回`string`。这里是因为`Array#reverse`是一个原地修改数组的方法，`string`无法直接接用。

#### `Number`

JavaScript 只有一种数字类型：`number`。这种类型包含“整数”值和小数值。我说“整数”时加了引号，因为 JS 的一个长久以来为人诟病的原因是，和其他语言不同，JS 没有真正的整数。一个“整数”只是一个没有小数部分的小数值。也就是说，`42.0` 和 `42` 一样是“整数”。

JavaScript 的 `number` 的实现基于“IEEE 754”标准，通常被称为“浮点”。JavaScript 明确地使用了这个标准的“双精度”（也就是“64位二进制”）格式。

##### 数字的语法

`number`值可以用`Number`对象包装器封装，所以`number`值可以访问`Number#fn`。比如`toFixed`指定一个值带有多少位小数，`toPrecision`指定一个值用多少位有效数字表示，返回的偶是字符串；



非常大或非常小的 `number` 将默认以指数形式输出，与 `toExponential()` 方法的输出一样，比如：

```javascript
var a = 5E10;
a;					// 50000000000
a.toExponential();	// "5e+10"

var b = a * a;
b;					// 2.5e+21

var c = 1 / a;
c;					// 2e-11
```



`number` 字面量还可以使用其他进制表达，比如二进制，八进制，和十六进制。推荐保持使用小写的谓词，`0x`（十六）、`0o`（八）、`0b`（二）

##### 小数值

二进制浮点数在涉及小数计算时，有一个臭名昭著的副作用：

```javascript
0.1 + 0.2 === 0.3; // false
```

简单地说，`0.1` 和 `0.2` 的二进制表示形式是不精确的，所以它们相加时，结果不是精确地 `0.3`。而是 **非常** 接近的值：`0.30000000000000004`。



（在应用程序处理整数时，数字操作的结果是非常安全的。）

要解决`0.1 + 0.2` 与 `0.3` 相等测试失败，常见的作法是设置一个 *容差* ，这个值也被称为 *机械极小值* ，这个值通常为`2^-52`。在ES6中这个容差值预定义位 `Number.EPSILON`。

```javascript
function numbersCloseEnoughToEqual(n1,n2) {
	return Math.abs( n1 - n2 ) < Number.EPSILON;
}

var a = 0.1 + 0.2;
var b = 0.3;

numbersCloseEnoughToEqual( a, b );					// true
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 );	// false
```

##### 安全整数范围

所谓"安全"，也就是说，可以保证被表示的值是无误的。ES6预定义了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`分别表示，最大安全整数，最小安全整数。

##### 测试整数

使用`ES6`定义的`Number.isInteger(..)`，测试一个值是否是整数。

为前ES6填补`Number.isInteger(..)`：

```javascript
if (!Number.isInteger) {
	Number.isInteger = function(num) {
		return typeof num == "number" && num % 1 == 0;
	};
}
```

使用`ES6`定义的`Number.isSafeInteger(..)`，测试一个值是否是安全整数。

为前 ES6 浏览器填补 `Number.isSafeInteger(..)`：

```javascript
if (!Number.isSafeInteger) {
	Number.isSafeInteger = function(num) {
		return Number.isInteger( num ) &&
			Math.abs( num ) <= Number.MAX_SAFE_INTEGER;
	};
}
```

##### 32位有符号整数

但有一些数字操作（比如位操作符）是仅仅为32位 `number` 定义的。

这个范围是从 `Math.pow(-2,31)`（`-2147483648`，大约-21亿）到 `Math.pow(2,31)-1`（`2147483647`，大约+21亿）。

可以使用`a | 0`位操作符做`ToInt32`转换。

#### 特殊值

##### `undefined`

`undefined`可以做标识符，但是为什么要这么做。。。也不要去覆盖`undefined`。

```javascript
function foo() {
	"use strict";
	var undefined = 2;
	console.log( undefined ); // 2
}

foo();
```

使用`void __`表达式结果总是`undefined`，用的多的是`void 0`

##### `null`

`null`是个关键字，不能作为标识符。其中`null`和`undefined`宽松等价的结果为`true`

##### `NaN`

`NaN` 是一种“哨兵值”，它代表 `number` 集合内的一种特殊的错误情况。这种错误情况实质上是：“我试着进行数学操作但是失败了，而这就是失败的 `number` 结果。”

测试一个值是否是`NaN`，不能直接将它与 `NaN` 本身比较，因为`NaN`是唯一不等于自身的：

```javascript
NaN == NaN; //	false
NaN === NaN; // fasle
```

测试的方法是使用 ES6 提供的 `Number.isNaN(..)`。

填补1，利用`NaN`是唯一不等于自身的值：

```javascript
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return n !== n;
	};
}
```

填补2：

```javascript
if (!Number.isNaN) {
	Number.isNaN = function(n) {
		return (
			typeof n === "number" &&
			window.isNaN( n )
		);
	};
}
```

##### 无穷

`Infinity`就是`Number.POSITIVE_INFINITY`，`-Infinity`就是`Number.NEGATIVE_INFINITY`

其中`Infinity/Infinity`结果为`NaN`

##### 零

JavaScript拥有普通的`0`（也称为正零`+0`）和一个负零`-0`。

为什么需要一个负零呢？在一些应用程序中，开发者使用值的大小来表示一部分信息（比如动画中每一帧的速度），而这个 `number` 的符号来表示另一部分信息（比如移动的方向）。在这些应用程序中，举例来说，如果一个变量的值变成了 0，而它丢失了符号，那么你就丢失了它是从哪个方向移动到 0 的信息。保留零的符号避免了潜在的意外信息丢失。

有两个特例：

- `-0 + ""`，将`-0`转换为字符串结果是`"0"`，与正零结果无区别
- `-0`与`0`无论是严格等价还是宽松等价，结果都是`true`

##### 特殊等价

当使用等价性比较时，值 `NaN` 和值 `-0` 拥有特殊的行为。

可以使用ES6 `Object.is(..)` 统一处理：

```javascript
var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN );	// true
Object.is( b, -0 );		// true

Object.is( b, 0 );		// false
```

填补：

```javascript
if (!Object.is) {
	Object.is = function(v1, v2) {
		// 测试 `-0`
		if (v1 === 0 && v2 === 0) {
			return 1 / v1 === 1 / v2;
		}
		// 测试 `NaN`
		if (v1 !== v1) {
			return v2 !== v2;
		}
		// 其他情况
		return v1 === v2;
	};
}
```

#### 值与引用

JS 中的引用指向一个**（共享的）** **值**，所以如果你有十个不同的引用，它们都总是同一个共享值的不同引用；

JS 中，值的 *类型*  用来唯一控制值是通过值拷贝，还是引用拷贝来赋予：

- 简单值总是通过值拷贝来赋予/传递：`null`、`undefined`、`number`、`string`、`boolean`、`symbol`
- 复合值—— `object` 以及`object`子类型，总是在赋值或传递时创建一个值的引用

##### 引用实践

1. 传递一个复合值给一个函数，为了防止函数操作影响原先的复合值，我们可以传递一份复合值的拷贝：

   ```javascript
   function foo(){...}
   var a = [1,2,3];
   
   foo( a.slice() );
   ```

2. 简单值传递给函数，函数操作是不会修改原先变量持有的简单值，我们可以用对象包装简单值，做到数据同步：

   ```javascript
   function foo(wrapper) {
   	wrapper.a = 42;
   }
   
   var obj = {
   	a: 2
   };
   
   foo( obj );
   
   obj.a; // 42
   ```

3. 特殊的demo：

   ```javascript
   function foo(x) {
   	x = x + 1;
   	x; // 3
   }
   
   var a = 2;
   var b = new Number( a ); // 或等价的 `Object(a)`
   
   foo( b );
   console.log( b ); // 基本标量值：2, 不是 3
   ```

   `Number`对象传递给`x`，`x+1`时开箱取底层基本标量值`2`。

   但是一个 `Number` 对象持有一个基本标量值 `2`，那么这个 `Number` 对象就永远不能再持有另一个值；你只能用一个不同的值创建一个全新的 `Number` 对象。

   所以结果还是`2`；