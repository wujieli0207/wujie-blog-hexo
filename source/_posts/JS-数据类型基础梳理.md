---
title: JS 数据类型基础梳理
date: 2021-12-28
categories: 前端
tags: JavaScript
---

## JS 的数据类型基础

- JS 中数据类型分为两大类：基础数据类型，引用数据类型

- 基础数据类型有 7 种，分别为：

  - Number 类型
  - String 类型
  - Boolean 类型
  - Null：“无” “空” “不存在”
  - Undefined：已被声明但未被赋值
  - Symbol：用于对象唯一标识符
  - BigInt 类型：表示大于或小于 2^53-1 的数字

- 引用数据类型： Object 为引用数据类型，还有其他子类型，如：`Function`、`Array`、`RegExp`、`Date`

- 基础数据类型存储在**栈内存**，被引用或者拷贝的时候，会创建一个完全相等的变量

- 引用类型存储在**堆内存**，存储的是地址，多个引用指向同一个地址

## JS 数据类型判断方式

- typeof 判断数据类型

  - typeof 能够准确判断基础数据类型，对于引用数据类型不能准确判断（引用类型只能判断出 Function）
  - `typeof null` 结果是 object 是 JS 早期错误，为兼容而保留
  - `typeof alert` alert 在 JS 中是一个函数

  ```javascript
  console.log(typeof undefined) // "undefined"
  console.log(typeof 0) // "number"
  console.log(typeof 10n) // "bigint"
  console.log(typeof true) // "boolean"
  console.log(typeof 'wujie') // "string"
  console.log(typeof Symbol('id')) // "symbol"
  console.log(typeof Math) // "object"  (1)
  console.log(typeof null) // "object"  (2)
  console.log(typeof alert) // "function"  (3)
  ```

- instanceof 判断数据类型

  - 通过判断对象原型链上的对象类型，来推断新对象的数据类型
  - 适用于判断引用数据类型，但不适合基础数据类型判断
  - 实现 instanceof 封装代码示例

    ```js
    function myInstanceof(left, right) {
      // 先使用 typef 判断，如果不是引用数据类型则直接返回
      if (typeof left !== 'object' || left === null) {
        return false
      }
      // 获取参数原型对象，循环对比原型链条上的对象是否和 right 对象的类型一致,
      let proto = Object.getPrototypeOf(left)
      while (true) {
        // 没有找到相同原型对象则返回 false
        if (proto === null) {
          return false
        }

        // 找到相同的原型对象才返回 true
        if (proto === right.prototype) {
          return true
        }
        proto = Object.getPrototypeOf(left)
      }
    }

    console.log(myInstanceof(new Number(10), Number)) // true
    console.log(myInstanceof(10, Number)) // false
    ```

- Object.prototype.toString 方法

  - 对象的原型方法，可以统一返回格式为 `[object Xxx]` 的字符串（注意 Xxx 第一个字母大写）
  - 对于 Object 对象，可以直接调用，其他对象需要通过 call 调用
    ```js
    console.log(Object.prototype.toString({})) // [object Object]
    console.log(Object.prototype.toString.call({})) // [object Object]
    console.log(Object.prototype.toString.call(10)) // [object Number]
    console.log(Object.prototype.toString.call('10')) // [object String]
    console.log(Object.prototype.toString.call(true)) // [object Boolean]
    console.log(Object.prototype.toString.call(null)) // [object Null]
    console.log(Object.prototype.toString.call(undefined)) // [object Undefined]
    console.log(Object.prototype.toString.call(function () {})) // [object Function]
    console.log(Object.prototype.toString.call(/wujiel/g)) // [object RegExp]
    console.log(Object.prototype.toString.call(new Date())) // [object Date]
    console.log(Object.prototype.toString.call([])) // [object Array]
    ```

- 全局通用的类型判断方法：可以通过 `typeof` +` Object.prototype.string.call()` 来实现

  ```js
  function getType(obj) {
    let type = typeof obj

    // 基础类型直接返回
    if (type !== 'object') {
      return type
    }

    // 引用类型 toString 判断,通过正则表达式过滤结果
    return Object.prototype.toString
      .call(obj)
      .replace(/^\[object (\S+)]$/, '$1')
      .toLowerCase()
  }

  console.log(getType(10)) // number
  console.log(getType([])) // array
  console.log(getType({})) // object
  console.log(getType(function () {})) // function
  console.log(getType(new Date())) // date
  ```

## JS 数据类型转换

- JS 的数据类型转换分为：强制类型转换，隐式类型转换

### 强制类型转换

- 通过 `Number() parseInt() parseFloat() toString() String Boolean()` 方法实现的数据转换

- `Number()` 类型转换规则

  - 数字，返回自身
  - Boolean，false 转换为 0，true 转换为 1
  - null 转换为 0
  - undefined 转换为 NaN
  - Symbol 抛出异常
  - 对象，使用 `Object` 转换规则
  - 字符串
    - 如果只包含数字，转换为十进制数字
    - 如果包含有效浮点格式转化为浮点数
    - 空字符串转换为 0
    - 以上三种之外的转换为 NaN

- `Boolean()` 类型装转换规则
  - 除了 undefined、null、false、"" 、0（包括 +0 和 -0）、NaN 转换为 false ，其他都转换为 true

### 隐式类型转换

- 逻辑运算操作符（&& || !），运算符（+ - \* /），关系操作符（> < >= <=），相等运算符（`==`），if / while 条件，在遇到两边类型不一致的情况，都会出现隐式类型转换

- `==` 隐式转换规则

  - 如果类型相同，无须转换
  - 如果其中一个是 null 或者 undefined，那么另一个必须为 null 或者 undefined，才会返回 true，否则都返回 false
  - 如果其中一个是 Symbol 类型，那么返回 false
  - 两个如果为 string 和 number 类型，那么就会将 string 转换为 number
  - 如果一个操作值是 boolean，那么转换 boolean 为 number
  - 如果一个操作值为 object 且另一方为 string、number 或者 symbol，就会把 object 转为原始类型再进行判断（调用 object 的 valueOf/toString 方法进行转换）

- `+` 隐式转换规则
  - 其中一个是字符串，另一个是 undefined null，则 undefined null 转换为字符串拼接；另一个普通对象、数组、正则等，先调用 Object 转换规则，再进行拼接
  - 其中一个是数字，另一个是 undefined null，则 undefined null 转换为数字进行计算
  - 其中一个是字符串、一个是数字，则按照字符串规则进行拼接
  - 示例代码
  ```js
  console.log(1 + true) // 2
  console.log(1 + null) // 1
  console.log(1 + undefined) // 1NaN
  console.log(1 + []) // 1{}
  console.log(1 + {}) // 1[object, object]
  console.log(1 + '1') // 11
  ```

### Object 隐式转换规则

- 如果有 `Symbol.toPrimitive` 方法，优先调用
- 调用 `valueOf()` 方法，如果转换为基础数据类型则返回
- 调用 `toString()` 方法，如果转换为基础数据类型则返回
- 以上三种没有转换成功则报错
- 示例代码

```js
let object = {
  value: 1,
  valueOf() {
    return 2
  },
  toString() {
    return 3
  },
  [Symbol.toPrimitive]() {
    return 4
  },
}

console.log(1 + object) // 5 优先调用 Symbol.toPrimitive 方法
console.log(1 + {}) // 1[object Object] 调用 toString 方法
console.log(1 + [1, 2, undefined, 3]) // 11,2,,3
```
