## 数组操作

```javascript
过滤数组元素
arr = arr.filter(item=>item.id!=id)
添加元素
list.push()//末尾添加
list.unshift()//开头添加
```

## JSON

```
json对象 = JSON.parse(字符串)
json字符串 = JSON.stringfy(json对象)
```

## 判断是不是某种数据类型

```javascript
转换为字符串:
var.to_String()
转换为数组:

```

## 导出模块

- 默认导出(export default)：导入的时候可以随意命名，每个文件只能有一个默认导出

```javascript
//module.js
export default function add(a,b){return a+b}
//main.js
import func from './module.js'
console.log(func(2+3))
```

