##### JSON 字符串化

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

##### 明确转换：String <---> Number

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

##### 隐含地：Strings <--> Numbers

为了服务于`number`的相加和`string`的连接两个目的，`+`操作符被重载了。

`+`的算法：

1. 如果两个操作数之一已经是一个`string`，或者两个操作数之一收到一个`object`时，通过`ToPrimitive`（`valueOf`）和`toString`方法转化为`string`，即和`ToNumber`处理`object`过程一致。这样的情况下，做字符串拼接
2. 其余情况做数字加法

`-`的算法：`-`操作符是仅为数字减法定义的，所以两个操作数都会被强制转换为`number`

**实践**

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

##### 隐含地：Boolean --> Number

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

##### 获取时间戳

1. 使用一元操作符`+`做强制转换：

   ```javascript
   //获取指定时间时间戳
   var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );
   
   +d; // 1408369986000
   
   //获取当前时间戳
   var timestamp = +new Date();
   ```

2. 不使用强制转换：

   ```javascript
   //获取当前时间戳
   var timestamp = Date.now();
   //获取指定时间时间戳
   var d = new Date("Mon, 18 Aug 2014 08:53:06 CDT").getTime()
   ```

##### 使用`~`的哨兵值`-1`做判断

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

##### 在`if`上下文环境下，使用表达式的正确姿势

```javascript
var a = "42";

// 不好（会失败的！）：
if (a == true) {
	// ..
}

// 也不该（会失败的！）：
if (a === true) {
	// ..
}

// 足够好（隐含地工作）：
if (a) {
	// ..
}

// 更好（明确地工作）：
if (!!a) {
	// ..
}

// 也很好（明确地工作）：
if (Boolean( a )) {
	// ..
}
```

