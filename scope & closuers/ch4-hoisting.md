### 提升

在一个作用域中声明的任何变量都附着在这个作用域上。

出现在一个作用域内各种位置的声明如何附着在作用域上？

#### 编译器再次来袭

编译器的工作：在代码执行之前，变量和函数声明被从代码流提升至当前作用域顶部。

例如：

```javascript
//变量提升
console.log( a );	//undefined

var a = 2;

//相当于
var a;
console.log(a);
a = 2;
```

```javascript
//函数提升
foo();	//可以直接调用，函数值提升了

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```

但是函数表达式不会被提升：

```javascript
foo(); // 不是 ReferenceError， 而是 TypeError!

var foo = function bar() {
	// ...
};
```

#### 函数优先

函数声明和变量声明都会被提升，如果遇到重复，函数首先被提升，其次才是变量。

一般避免这样的重复情况

#### 