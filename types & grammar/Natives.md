### 原生类型

是最常用的原生类型的一览：

- `String()`
- `Number()`
- `Boolean()`
- `Array()`
- `Object()`
- `Function()`
- `RegExp()`
- `Date()`
- `Error()`
- `Symbol()` 

#### 内部 `[[Class]]`

内部的标签属性`[[Class]]` 不能直接地被访问，但通常可以间接地通过在这个值上借用默认的`Object.prototype.toString(..)` 方法调用来展示。举例来说：

```javascript
Object.prototype.toString.call( [1,2,3] );			// "[object Array]"
Object.prototype.toString.call( /regex-literal/i );	// "[object RegExp]"

Object.prototype.toString.call( "abc" );	// "[object String]"
Object.prototype.toString.call( 42 );		// "[object Number]"
Object.prototype.toString.call( true );		// "[object Boolean]"

Object.prototype.toString.call( null );			// "[object Null]"
Object.prototype.toString.call( undefined );	// "[object Undefined]"
```

对于`Array`和正则表达式，`[[Class]]`值对应于原生类型构造器；

`string`、`number`、`boolean`在放入`Object.prototype.toString.call(..)`时自动地被它们分别对应的对象包装器**封箱**，所以`[[class]]`值也对应于封箱后对象的原生类型构造器；

`null`和`undefined`是个特例，因为没有对应的原生类型构造器；

#### 封箱包装器

封箱包装其的概念是针对`string`、`number`和`boolean`三种基本类型值的。

它分为自动封箱和手动封箱。一般不使用手动封箱，让自动封箱隐含发生，因为浏览器对自动封箱去调用某些属性和方法进行了性能优化，而且代码也更加简洁。

##### 自动封箱

基本类型值没有属性和方法，所以为了访问例如`.length`、`toString()`，JS会<u>自动地封箱基本类型值</u>来满足这样的访问

```javascript
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"
```

浏览器们长久以来就对 `.length` 这样的常见情况进行性能优化，这意味着如果你试着直接使用对象形式再去访问，那么实际上你的程序将会 *变慢*。

##### 手动封箱

可以使用基本类型值对应的原生类型构造器，也可以使用`Object(..)`函数（没有`new`关键字）

```javascript
var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"
```

###### 开箱

如果有一个包装器对象，想要取出底层的基本类型值，可以使用`valueOf(..)`

```javascript
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

当以一种查询基本类型值的方式使用对象包装器时，开箱也会<u>隐含地发生</u>：

```javascript
var a = new String( "abc" );
var b = a + ""; // `b` 拥有开箱后的基本类型值"abc"

typeof a; // "object"
typeof b; // "string"
```

#### 原生类型作为构造器

除去`String`、`Number`和`Boolean`，了解一下剩余7种原生类型，字面形式创建和原生类型作为构造器创建的选择。

##### `array`、`object`、`function`和正则表达式

使用字面形式创建它们的值几乎总是更好的选择，而且字面形式与构造器形式创建的值是同一种对象。

- `array`使用构造器形式创建，如下所示会创建具有空值槽的稀散数组，<u>稀散数组是个怪胎别去碰</u>：

  ```javascript
  var a = new Array( 3 );
  
  a.length; // 3
  a;	//[empty × 3]
  ```

- `object`如果使用`new Object()`构造器形式，<u>得一个一个地添加属性，倍儿麻烦</u>
- `Function` 构造器仅在<u>最最罕见</u>的情况下有用，也就是你需要动态地定义一个函数的参数和/或它的函数体
- 正则表达式用字面量形式是被大力采用的，不仅因为语法简单，而且还有性能的原因 —— JS 引擎会在代码执行前预编译并缓存它们。不过构造形式`new RegExp(..)`有一个用途：在需要使用`new RegExp("pattern","flags")`中`flags`时候

##### `Date(..)`和`Error(..)`

`Date`和`Error`没有字面量形式，只能使用原生类型构造器构建。

- `Date`获取时间戳

  - 获取当前时间时间戳：`Date.now()`（ES5添加），等价于 `new Date().getTime()`
  - 获取指定时间时间戳：`new Date(...).getTime()`

- 典型地，你将与 `throw` 操作符一起使用这样的 error 对象：

  ```javascript
  function foo(x) {
  	if (!x) {
  		throw new Error( "x wasn't provided" );
  	}
  	// ..
  }
  ```

  **提示：** 技术上讲，除了一般的 `Error(..)` 原生类型以外，还有几种特定错误的原生类型：`EvalError(..)`、`RangeError(..)`、`ReferenceError(..)`、`SyntaxError(..)`、`TypeError(..)` 和 `URIError(..)`。但是手动使用这些特定错误原生类型十分少见。如果你的程序确实遭受了一个真实的异常，它们是会自动地被使用的（比如引用一个未声明的变量而得到一个 `ReferenceError` 错误）。

##### `Symbol(..)`

`Symbol` *不是* `object`，它们是<u>简单的基本标量</u>；

ES6有几种<u>预定义的 Symbol</u>，比如 `Symbol.iterator` ：

```javascript
obj[Symbol.iterator] = function(){ /*..*/ };
```

要<u>定义自己的 Symbol</u>，使用 `Symbol(..)` 原生类型。`Symbol(..)` 原生类型“构造器”很独特，因为它不允许你将 `new` 与它一起使用，这么做会抛出一个错误；

`Symbol`主要用途是<u>作为对象的私有属性</u>，对于大多数开发者，会在私有属性名上加入 `_` 下划线前缀：

```javascript
var _mysym = Symbol( "my own symbol" );
_mysym;				// Symbol(my own symbol)
_mysym.toString();	// "Symbol(my own symbol)"
typeof _mysym; 		// "symbol"

var a = { };
a[_mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

#### 原生类型原型

每一个内建的原生构造器都拥有它自己的 `.prototype` 对象 —— `Array.prototype`，`String.prototype` 等等。

对于它们特定的对象子类型，这些对象含有独特的行为。

例如，所有的字符串对象，和 `string` 基本值的扩展（通过封箱），都可以访问在 `String.prototype` 对象上做为方法定义的默认行为。