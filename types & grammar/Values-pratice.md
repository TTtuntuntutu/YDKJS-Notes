##### 类 Array 转 Array

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

2. `Array.from(..)`

   ```javascript
   ...
   var arr = Array.from( arguments );
   ...
   ```

##### `string` 借用 `array` 非变化方法

https://jsbin.com/yixazuhanu/edit?html,js,console

