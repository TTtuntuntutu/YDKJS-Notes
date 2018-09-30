### 函数与块儿作用域

将作用域比作 “气泡”，什么才能制造一个新 “气泡”？也就是，作用域是怎么产生的问题。

#### 函数中的作用域

每一个声明的函数都为自己创建了一个气泡。

考虑一个函数的传统方式是，你声明一个函数，并在它内部添加代码。但是相反的想法也挺有意思：拿编写的代码的任意一部分，在它周围包装一个函数声明，制造一个 “气泡” 去 “隐藏” 这段代码。

##### “最低权限原则”：优化作用域集结构

有一种称为 “最低权限原则” 的软件设计原则 —— 应当只暴露所需要的最低限度的东西，而隐藏其他的一切。这个原则扩展到 **用哪个作用域来包含变量和函数的选择**。

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

