##### `typeof` 检测值的类型

`typeof` 操作符可以检测给定值的类型，而且总是返回七种字符串值中的一种。但是并不是与js定义的7种内建类型一一对应：

```javascript
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true

// 在 ES6 中被加入的！
typeof Symbol()      === "symbol";    // true

//特殊的
typeof function foo(){}	=== "function"	//true
```

`null`是特例：

```javascript
typeof null === "object"; // true
```

测试`null`值需要复合一个条件：

```javascript
var a = null;

(!a && typeof a === "object"); // true
```