使用内部属性`[[Class]]`区分类型更加精确：

```javascript
function getExactType(value){
  var innerClass = Object.prototype.toString.call(value);

  var type = innerClass.slice(8,innerClass.length-1);

  return type;
}

console.log(getExactType(null));	//"Null"
console.log(getExactType([1,2,3]));	//"Array"
console.log(getExactType(/regex-literal/i));	//"RegExp"
```

