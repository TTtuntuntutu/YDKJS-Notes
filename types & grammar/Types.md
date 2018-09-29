## 类型

每个值都有一种关联的类型，每一种类型对应着固有的行为。

### 内建类型

JavaScript定义了7种内建类型：`null`、`undefined`、`boolean`、`string`、`number`、`object`、`symbol`。

除了`object`，其他所有类型都是基本类型。

#### `typeof`检测值的类型

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

### 值作为类型

在 JavaScript 中，变量没有类型 -- **值才有类型**。变量可以在任何时候，持有任何值。

#### `undefined` vs "undeclared"

*一个“undefined”变量* 是在可访问的作用域中已经被声明过的，但是在 *这个时刻* 它里面没有任何值。相比之下，*一个“undeclared”变量* 是在可访问的作用域中还没有被正式声明的。

```javascript
var a;

a; // undefined	"undefined"变量
b; // ReferenceError: b is not defined	"undeclared"变量
```

#### `typeof` Undeclared

```javascript
typeof b; // "undefined"
b;		//ReferenceError: b is not defined
```

当操作`typeof b`时，b是一个 undeclared 变量，也不会有错误被抛出。这是`typeof`的一种特殊的安全防卫行为。

安全防卫行为的具体使用参见：https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/types%20&%20grammar/ch1.md#typeof-undeclared

暂时可能用途不大