## 强制转换

探索强制转换的优点和缺点

### 转换值

JavaScript强制转换总是得到基本标量值的一种，`string`、`number`或者`boolean`。

### 抽象值操作

学习一些基本规则，这些规则控制着值如何强制转换为一个`string`、`number`或者`boolean`。特别关注于`ToString`、`ToNumber`、`ToBoolean`，并且稍稍关注一下`ToPrimitive`。

#### `ToString`

`ToString` 抽象操作，处理一个非 `string` 值被强制转换为一个 `string` 表现形式。

1. 内建的基本类型值，拥有自然的字符串化形式：

   - `null` 变为 `"null"`、`undefined` 变为 `"undefined"`、`true` 变为 `"true"`

   - `number` 一般会以期望的自然方式表达，在非常小或非常大情况下会以指数形式表达：

     ```javascript
     // `1.07`乘以`1000`，7次
     var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;
     
     // 7次乘以3位 => 21位
     a.toString(); // "1.07e21"
     ```

2. 普通的对象

   - 如果一个对象拥有自己的`toString`，它自己的`toString`方法被调用。

     比如数组的`toString`，转换为所有的值（每个都字符串化）的（字符串）连接，并用 `","`分割每个值：

     ```javascript
     var a = [1,2,3];
     
     a.toString(); // "1,2,3"
     ```

   - 排除自己存在`toString`方法的情况，调用默认的`toString`（即`Object.prototype.toString`），返回内部`[[Class]]`

##### JSON字符串化

JSON字符串化的工作逻辑：

1. 简单值，JSON字符串行为基本和`toString()`相同，除了<u>字符串类型</u>：

   ```javascript
   JSON.stringify( 42 );	// "42"
   JSON.stringify( "42" );	// ""42"" （一个包含双引号的字符串）
   JSON.stringify( null );	// "null"
   JSON.stringify( true );	// "true"
   ```

2. 如果一个`object`定义了一个`toJSON`方法，这个方法会被首先调用，返回的值再用于JSON字符串化。作用之一是，消除JSON 不安全值：

   ```javascript
   var o = { };
   
   var a = {
   	b: 42,
   	c: o,
   	d: function(){}
   };
   
   // 在 `a` 内部制造一个循环引用
   o.e = a;
   
   // 这会因循环引用而抛出一个错误
   // JSON.stringify( a );
   
   // 自定义一个 JSON 值序列化
   a.toJSON = function() {
   	// 序列化仅包含属性 `b`
   	return { b: this.b };
   };
   
   JSON.stringify( a ); // "{"b":42}"
   ```

3. JSON 不安全值：`undefined`、`function`、（ES6+）`symbol`、和带有循环引用的 `object`（一个对象结构中的属性互相引用而造成了一个永不终结的循环）

   - `JSON.stringify(..)` 遇到 `undefined`、`function`、和 `symbol` 时将会自动地忽略它们。如果在一个 `array` 中遇到这样的值，它会被替换为 `null`（这样数组的位置信息就不会改变）。如果在一个 `object` 的属性中遇到这样的值，这个属性会被简单地剔除掉。

     ```javascript
     JSON.stringify( undefined );					// undefined
     JSON.stringify( function(){} );					// undefined
     
     JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
     JSON.stringify( { a:2, b:function(){} } );		// "{"a":2}"
     ```

   - 如果试着`JSON.stringify(..)` 一个带有循环引用的 `object`，就会抛出一个错误

`JSON.stringify(...)` 有用功能

1. 第二个可选参数 *替换器* ：它提供一种过滤机制，指出一个 `object` 的哪一个属性应该或不应该被包含在序列化形式中。参数值既可以是一个`array`也可以是一个`function`：

   ```javascript
   var a = {
   	b: 42,
   	c: "42",
   	d: [1,2,3]
   };
   
   JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"
   
   JSON.stringify( a, function(k,v){
   	if (k !== "c") return v;
   } );
   // "{"b":42,"d":[1,2,3]}"
   ```

   **注意：** 在 `function` *替换器* 的情况下，第一次调用时 key 参数 `k` 是 `undefined`（而对象 `a` 本身会被传入）。`if` 语句会 **过滤掉** 名称为 `c` 的属性。字符串化是递归的，所以数组 `[1,2,3]` 会将它的每一个值（`1`、`2`、和 `3`）都作为 `v` 传递给 *替换器*，并将索引值（`0`、`1`、和 `2`）作为 `k`。

2. 第三个可选参数 *填充符* ：在对人类友好的输出中它被用做缩进。*填充符* 可以是一个正整数，用来指示每一级缩进中应当<u>使用多少个空格字符</u>。或者，*填充符* 可以是一个 `string`，这时<u>每一级缩进将会使用它的前十个字符</u>。

   ```javascript
   var a = {
   	b: 42,
   	c: "42",
   	d: [1,2,3]
   };
   
   JSON.stringify( a, null, 3 );
   // "{
   //    "b": 42,
   //    "c": "42",
   //    "d": [
   //       1,
   //       2,
   //       3
   //    ]
   // }"
   
   JSON.stringify( a, null, "-----" );
   // "{
   // -----"b": 42,
   // -----"c": "42",
   // -----"d": [
   // ----------1,
   // ----------2,
   // ----------3
   // -----]
   // }"
   ```

#### `ToNumber`

如果任何非 `number` 值，以一种要求它是 `number` 的方式被使用，比如数学操作，就会发生`ToNumber`操作。

1. `true` 变为 `1` 而 `false` 变为 `0`。`undefined` 变为 `NaN`，而（奇怪的是）`null` 变为 `0`
2. 对于`string`值，如果`ToNumber`失败，结果是`NaN`，以及 `0` 前缀的八进制数不会被作为八进制数来处理
3. 对象（以及数组）将会首先被转换为它们的<u>基本类型值的等价物</u>，而后这个结果值（如果它还不是一个 `number` 基本类型）会根据前面提到的 `ToNumber` 规则被强制转换为一个 `number`
   - 为了转换为基本类型值的等价物，`ToPrimitive` 抽象操作将会查询这个值，看它有没有 `valueOf()` 方法。如果 `valueOf()` 可用并且它返回一个基本类型值，那么 *这个* 值就将用于强制转换。如果不是这样，但 `toString()` 可用，那么就由它来提供用于强制转换的值。如果这两种操作都没提供一个基本类型值，就会抛出一个 `TypeError`。

#### `ToBoolean`

认识一下falsy表：

- `undefined`
- `null`
- `false`
- `+0`, `-0`, and `NaN`
- `""`

falsy表中的值在进行`boolean`强制转换时会转换为`false`，其余的都转换为`true`

### 明确的强制转换

在代码中指名一些模式，在这些模式中可以清除明白地将一个值从一种类型转换至另一种类型，以确保不给未来读到这段代码的开发者留下任何坑。

#### 明确地：Strings <--> Numbers

1. 使用内建的`String(..)`和`Number(..)`函数。**非常重要的是**，使用时不在前面使用`new`关键字，这样，就不是在创建对象包装器

   ```javascript
   var a = 42;
   var b = String( a );
   
   var c = "3.14";
   var d = Number( c );
   
   b; // "42"
   d; // 3.14
   ```

2. 使用`toString`，将Number转为Strings

   ```javascript
   var a = 42;
   var b = a.toString();
   
   b; //"42"
   ```

   调用`a.toString()`在表面上是明确的，但是这里有一些<u>藏起来的隐含性</u>：`toString()`不能在像`42`这样的 *基本类型* 值上调用。所以JS会自动地将`42`“封箱”在一个对象包装器中，这样`toString()`就可以针对这个对象调用。

3. 使用一元操作符`+`、`-`，将Strings转为Number

   ```javascript
   var c = "3.14";
   var d = +c;
   var e = -c;
   
   d; // 3.14
   e; // -3.14
   ```

##### 一元操作符`+`另一个常见用途：获取时间戳

```javascript
//获取指定时间时间戳
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000

//获取当前时间戳
var timestamp = +new Date();	
```

这里获取时间戳的方法使用了强制转换，另一种不使用强制转换的方法更加推荐，因为更明确：

```javascript
//获取当前时间戳
var timestamp = Date.now();
//获取指定时间时间戳
var d = new Date("Mon, 18 Aug 2014 08:53:06 CDT").getTime()
```

##### 奇异的`~`

`~`是什么？做了什么？

`~`操作符也称为按位取反，首先将值“强制转换”为一个32位`number`值（`ToInt32`），然后实施按位取反（翻转每一个比特位）。

1. `ToInt32`转换，等价于`|`操作符的`0 | x`，demo中，这些特殊的数字是不可用32位表现的，所以`ToInt32`将这些值的结果指定为`0`：

   ```javascript
   0 | -0;			// 0
   0 | NaN;		// 0
   0 | Infinity;	// 0
   0 | -Infinity;	// 0
   ```

2. 实施按位取反，相当于将`ToInt32`得到的结果`x`，做`-(x+1)`操作。除了对数值取反也对符号取反：

   ```javascript
   ~42;	// -(42+1) ==> -43
   ```

`~`的应用：

1. 使用`~`的哨兵值`-1`做判断（只有`~-1`的结果为`0`，`-1`被称为哨兵值）

   例如：

   ```javascript
   var a = "Hello World";
   if (~a.indexOf( "lo" )) {	
   	// 找到了！
   }
   
   if (!~a.indexOf( "ol" )) {	
   	// 没找到！
   }
   
   //代替原来与-1作比较
   if (a.indexOf( "lo" )!==-1) {	
   	// 找到了！
   }
   
   if (a.indexOf( "ol" )===-1) {	
   	// 没找到！
   }
   ```

2. 一些开发者使用双波浪线`~~`来截断一个`number`的小数部分

   `~ ~`的工作方式是，第一个`~`实施`ToInt32`“强制转换”并进行按位取反，然后第二个`~`进行另一次按位取反，将每一个比特位都翻转回原来的状态。于是最终的结果就是`ToInt32`“强制转换”（也叫截断）；

   所以它和`Math.floor(..)`是有区别的，因为按照`~~`的工作方式，小数部分无论正负都是被截断：

   ```javascript
   //正数的结果是一致的
   Math.floor(50.6);	//50
   ~~50.6;			//50
   
   //负数的结果有区别
   Math.floor( -49.6 );	// -50
   ~~-49.6;			// -49
   ```

#### 明确地：解析数字字符串

解析数字字符串是什么？

比如：

```javascript
var a = "42";
var b = "42px";

Number( a );	// 42
parseInt( a );	// 42

Number( b );	// NaN
parseInt( b );	// 42
```

从一个字符串中解析出一个数字是 *容忍* 非数字字符的 —— 从左到右，如果遇到非数字字符就停止解析 —— 而强制转换是 *不容忍* 并且会失败而得出值`NaN`。

`parseInt(..)`工作在`string`值上，如果传入一个非`string`，所传入的值首先将自动地被强制转换为一个`string`，这样的行为是一个 bad idea。

解析数字字符串的第二个参数有什么用？

一个臭名昭著的例子：

```javascript
parseInt( 1/0, 19 ); // 18
```

1. 传入`1/0`非`string`值，被转化为`Infinity`
2. `19`指定按19进制解析，在19进制中`I`是最大的`18`，`N`无意义，所以结果是`18`。第二个参数的作用是指定按什么进制解析，默认为十进制

#### 明确地：* --> Boolean

1. `Boolean(..)`是强制进行`ToBoolean`转换的明确方法
2. 使用`!!`双否定操作符进行`boolean`强制转换，更常用

### 隐含的强制转换

隐含强制转换是什么、可以是什么？学习隐含强制转换做某些有用的事儿，避免被滥用或误用来做某些可怕的事儿。

#### 隐含地：Strings <--> Numbers

为了服务于`number`的相加和`string`的连接两个目的，`+`操作符被重载了。

`+`的算法：

1. 如果两个操作数之一已经是一个`string`，或者两个操作数之一收到一个`object`时，通过`ToPrimitive`（`valueOf`和`toString`方法）转化为`string`，即和`ToNumber`处理`object`过程中间步骤一致。这样的情况下，做字符串拼接
2. 其余情况做数字加法

`-`的算法：`-`操作符是仅为数字减法定义的，所以两个操作数都会被强制转换为`number`

##### 实践

1. 通过`number`+`""`，把一个`number`强制转换为一个字符串

   ```javascript
   var a = 42;
   var b = a + "";
   
   b; // "42"
   ```

2. 通过`string`-`0`，把一个`string`强制转换为一个数字

   ```javascript
   var a = "3.14";
   var b = a - 0;
   
   b; // 3.14
   ```

#### 隐含地：Boolean --> Number

隐含强制转换一个真正闪光的情况：将特定类型的复杂`boolean`逻辑简化为简单的数字加法。

考虑以下代码，在有且仅有一个参数为`true`时返回`true`：

```javascript
function onlyOne(a,b,c) {
	return !!((a && !b && !c) ||
		(!a && b && !c) || (!a && !b && c));
}

var a = true;
var b = false;

onlyOne( a, b, b );	// true
onlyOne( b, a, b );	// true

onlyOne( a, b, a );	// false
```

使用`+`操作符将Boolean隐含强制转换为Number，简化代码：

```javascript
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		// 跳过falsy值。与将它们视为0相同，但是避开NaN
		if (arguments[i]) {
			sum += arguments[i];
		}
	}
	return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a );				// true
onlyOne( b, a, b, b, b );		// true

onlyOne( b, b );				// false
onlyOne( b, a, b, b, b, a );	// false
```

#### 隐含地：* --> Boolean

在以下上下文环境中，任何不是 `boolean` 的值，被隐含地强制转化为一个`boolean`：

1. 在一个`if (..)`语句中的测试表达式
2. 在一个`for ( .. ; .. ; .. )`头部的测试表达式（第二个子句）
3. 在`while (..)`和`do..while(..)`循环中的测试表达式
4. 在`? :`三元表达式中的测试表达式（第一个子句）
5. `||`（“逻辑或”）和`&&`（“逻辑与”）操作符左手边的操作数

##### `||`和`&&`操作符

`||`和`&&`操作符的工作原理？

demo：

```javascript
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

`||`和`&&`操作符都在 **第一个操作数**（`a`或`c`） 上进行`boolean`测试。如果这个操作数还不是`boolean`（就像在这里一样），就会发生一次普通的`ToBoolean`强制转换，这样测试就可以进行了。

对于`||`操作符，如果测试结果为`true`，`||`表达式就将 *第一个操作数* 的值（`a`或`c`）作为结果。如果测试结果为`false`，`||`表达式就将 *第二个操作数* 的值（`b`）作为结果。

相反地，对于`&&`操作符，如果测试结果为`true`，`&&`表达式将 *第二个操作数* 的值（`b`）作为结果。如果测试结果为`false`，那么`&&`表达式就将 *第一个操作数* 的值（`a`或`c`）作为结果。

所以`||`和`&&`操作符其实就是在**在两个操作数的值中选择一个**。

几乎等价于三元操作符的模拟：

```javascript
a || b;
// 大体上等价于：
a ? a : b;

a && b;
// 大体上等价于：
a ? b : a;
```

#### Symbol强制转换

需要强制转换一个`symbol`值的情况可能极其少见，典型的被使用的方式（作为对象私有属性）可能不会用到强制转换。

1. 既可以 *明确地* 也可以 *隐含地* 强制转换为`boolean`（总是`true`）
2. 从一个`symbol`到一个`string`的 *明确* 强制转换是允许的，但是相同的 *隐含* 强制转换是不被允许的，而且会抛出一个错误
3. `symbol`值根本不能强制转换为`number`

### 宽松等价与严格等价

`==`和`===`都会检查它们的操作数的类型。不同之处在于它们在类型不同时如何反应。`==`允许在等价性比较中进行强制转换，而`===`不允许强制转换。`==`和`===`使用的是相同的算法。

所以关键是，当比较这两个值时，<u>我想要进行强制转换吗？</u>

#### 抽象等价性

`==`操作符的行为被定义为“抽象等价性比较算法”，在两个被比较值是同一类型情况下，`===`操作符和`==`行为一致。具体如下：

1. 如果两个被比较的值是同一类型，结果如期望一样比较（宽松等价和严格等价的行为一致）

   有一些例外要小心：

   - `NaN`永远不等于它自己、`+0`和`-0`是相等的
   - 关于`object`，两个`objeect`仅在它们引用 *完全相同的值* 时 *相等*（这里没有强制转换发生）

2. 如果使用`==`宽松等价来比较两个不同类型的值，两者或其中之一将需要被 *隐含地* 强制转换。由于这个强制转换，两个值最终归于同一类型，可以使用简单的值的等价性来直接比较它们相等与否。

##### 比较：`string`和`number`

考虑如下代码：

```javascript
var a = 42;
var b = "42";

a === b;	// false
a == b;		// true
```

使用`==`宽松等价时，永远是`string`值被强制转换为一个`number`

##### 比较：任何东西与`boolean`

考虑如下代码：

```javascript
var a = "42";
var b = true;

a == b;	// false
```

使用`==`宽松等价时，`boolean`值首先被转换为一个`number`。

所以`true`转换为`1`，就回到了`"42"==1` `string`和`number`比较的问题上，显然这是不相等的，所以结果为`false`.

**建议：不要使用`== true`或`== false`**

##### 比较：`null`和`undefined`

使用`==`宽松等价比较`null`和`undefined`，它们是互相等价的，而且在整个语言中不会等价于其他值了。

比如：

```javascript
var a = doSomething();

if (a == null) {
	// ..
}
```

`a == null`检查仅在`doSomething()`返回`null`或者`undefined`时才会通过，而在任何其他值的情况下将会失败，即便是`0`，`false`，和`""`这样的falsy值。

##### 比较：`object`与非`object`

如果`object`和一个`string`、`number`宽松等价比较，返回`ToPrimitive`处理`object`得到的值，和`string`、`number`再比较的结果。

```javascript
var a = 42;
var b = [ 42 ];

a == b;	// true
```

"拆箱" 是一个基本类型值的`object`包装器被展开，返回其底层的基本类型值。例如`new String("abc")`这样的形式，返回`abc`。其中有些值放到宽松等价这里是例外：

```javascript
var a = null;
var b = Object( a );	// 与`Object()`相同
a == b;					// false

var c = undefined;
var d = Object( c );	// 与`Object()`相同
c == d;					// false

var e = NaN;
var f = Object( e );	// 与`new Number( e )`相同
e == f;					// false
```

`null`和`undefined`不能被装箱，所以`Object(null)`、`Object(undefined)`就像`Object()`一样，它们都仅仅产生一个普通对象。所以`a == b`、`c == d`返回`false`；

`NaN`可以被装箱，但是`NaN`不等于自身。所以`e == f`返回`false`

#### 边界情况

7个坑：

```javascript
"0" == false;			// true -- 噢！
false == 0;				// true -- 噢！
false == "";			// true -- 噢！
false == [];			// true -- 噢！
"" == 0;				// true -- 噢！
"" == [];				// true -- 噢！
0 == [];				// true -- 噢！
```

建议：

1. 如果比较的任意一边可能出现`true`或者`false`值，那么就永远，永远不要使用`==`。
2. 如果比较的任意一边可能出现`[]`，`""`，或`0`这些值，那么认真地考虑不使用`==`。

https://dorey.github.io/JavaScript-Equality-Table/

#### 抽象关系比较

工作原理：

1. 如果是`object`，先调用`ToPrimitive`强制转换
2. 如果比较涉及两个`string`值，按字典顺序比较
3. 如果其中之一不是`string`，使用`ToNumber`操作规则将这两个值强制转换为`number`值，并进行数字的比较。

`a<=b`等价于`!(a>b)`，`a>=b`等价于`!(a<b)`。demo：

```javascript
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false
a > b;	// false

a <= b;	// true
a >= b;	// true
```

如果强制转换由帮助并且合理，做明确强制转换处理：

```javascript
var a = [ 42 ];
var b = "043";

a < b;						// false -- 字符串比较！
Number( a ) < Number( b );	// true -- 数字比较！
```

