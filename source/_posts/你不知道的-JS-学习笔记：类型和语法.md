---
title: 你不知道的 JS 学习笔记：类型和语法
date: 2022-02-21
categories: 前端
tags: JavaScript
---
## 第一章：类型
- **JS 中的类型：值的内部特征，定义了值的行为，使其区别于其他值**
- JS 的七种内置类型
  - number 数字
  - string 字符串
  - boolean 布尔值
  - null 空值
  - undefined 未定义
  - symbol 符号
  - object 对象（除了 object 其他都是基本类型）
- 可以使用 typeof 检查基本类型
    ``` js
        console.log(typeof "42" === "string"); // true
        console.log(typeof 42 === "number"); // true
        console.log(typeof true === "boolean"); // true
        console.log(typeof null === "object"); // true 注意 null 是 object，这是 JS 的 BUG
        console.log(typeof undefined === "undefined"); // true
        // ES6中新加入的类型
        console.log(typeof Symbol() === "symbol"); // true
        console.log(typeof { life: 42 } === "object"); // true
    ```
- Function 是对象的字类型，拥有自己的属性（比如 length，返回参数的个数）
- JS 中的变量没有类型，只有值才有，变量先被赋值字符串类型的值，后被赋值数字类型的值，所以 JS 是弱类型语言
- undefined
	- 已经在作用域声明，但还没被赋值
	- 在作用域中没有被声明是 undeclared
	- **注意**：对于上面两种情况，typeof 返回的都是 undefined

## 第二章：值
- 2.1 数组
	- 数组可以存放任何类型的值
	- JS 中数组不用预先声明大小
	- 数组也是对象，可以包含自己的键值属性，但不会计算在 length 中
	- 类数组
		- 一组通过数字索引的值，比如：DOM 查询操作返回结果
		- 可以使用 `Array.slice` 或 `Array.from` 将类数组转化为数组
- 2.2 字符串
	- 字符串可以借用数组方法
		- 借用 join 方法示例：`let b = Array.prototype.join( a, "-");`
		- 无法直接借用 reverse 方法，需要先转化为字符数组才能使用
            ``` js
                let c = a
                .split( "" )
                .reverse()
                .join( "" );
            ```
- 2.3 数字
	- 数字以十进制的方式显示，小数后的 0 会被省略
	- `tofiexed()` 方法可以指定小数部分的显示位数，结果是字符串形式
	- `toprecision()` 方法可以指定有效位数的显示位数
        ``` js
            var a = 42.59; a.toPrecision(1); // "4e+1"
            a.toPrecision(2); // "43"
            a.toPrecision(3); // "42.6"
            a.toPrecision(4); // "42.59"
            a.toPrecision(5); // "42.590"
            a.toPrecision(6); // "42.5900"
        ```
	- 存在二进制浮点数计算精度问题（所有遵循IEEE 754规范的语言都是如此）
        ``` js
            console.log(0.1 + 0.2 === 0.3); // false
        ```
		- 原因：二进制浮点数 0.1 和 0.2 相加后约等于 0.30000000000000004，所用判断为 false
		- 解决方案：使用误差范围值
			- ES6 中定义为 `Number.EPSILON`,通常为：2^-52 (2.220446049250313e-16)
			- 使用误差范围判断两个数字是否相等
                ``` js
                    function numbersCloseEnoughToEqual(n1, n2) {
                    return Math.abs(n1 - n2) < Number.EPSILON;
                    }

                    console.log(numbersCloseEnoughToEqual(0.1 + 0.2, 0.3)); // true
                    console.log(numbersCloseEnoughToEqual(0.0000001, 0.0000002)); // false
                ```
	- 整数的安全范围
		- ES6 定义：[Number.MIN_SAFE_INTEGER, MAX_SAFE_INTEGER]
		- 即：[-(2^53  -  1), 2^53  -  1]
	- 判断是否是整数
		- 使用 `Number.isInteger()` 判断是否是整数
		- 使用 `Number.isSafeInteger()` 判断是否是安全范围内的整数
- 2.4 特殊数值
	- undefined 和 null
		- undefined 类型只有一个值，即：undefined
		- null 类型只有一个值，即：null
		- undefined 指未被赋值，null 指曾被赋值，但目前没有值
		- undefined 可以作为变量声明和赋值（**不要这么做**）
	- void
		- 指没有返回值，多用于函数没有返回结果
		- **可以使用 `void 0` 获取真正的 undefined**（非严格模式 undefined 可以被赋值，而 `void 0` 必定返回 undefined）
	- 特殊的数字
		- NAN：
			- 无效数值（仍然是一个数字类型，但指数字类型中的错误情况）
			- NAN 和谁比较都是 false，包括自己
			- 使用 `Number.isNaN()` 判断一个数字是否为 NAN
                ``` js
                    if (!Number.isNaN) {
                    Number.isNaN = function (n) {
                        return typeof n === "number" && window.isNaN(n);
                    };
                    }
                ```
		- 无穷数字：Infinity
			- ES6 中定义为：`Number.POSITIVE_INFINITY`，`Number.NEGATIVE_INFINITY`
			- 计算结果一旦溢出为无穷数，就无法再转换为有穷数
		- 零值：0、-0
			- 乘法和除法运算会得到 -0
			- 存在 -0 的原因：某些程序需要使用级数来表示（比如动画帧的移动速度），数字的符号位（sign）用来代表其他信息（比如移动的方向）
			- 0 === -0
	- 特殊等式
		- `Object.is(a, b)` 可以判断两个值是否绝对相等
		- 优先使用 == 和 ===，因为效率更高
- 2.5 值和引用
	- JS 中没有指针，JS 的变量不可能指向另一个变量的引用，**JS 引用指向的是值**
	- 基本类型通过**复制**方式复制或传递，引用类型通过**引用**方式复制或传递
	- 函数中的引用问题
		- 函数参数 a 通过**复制**的方式复制给函数内的 x
		- **引用 x 不能改变引用 a 的指向，只能改变 a 和 x 共同指向的值**
            ``` js
                function foo(x) {
                x.push(4);
                console.log(x);

                // 注意引用 x 指向了其他的值
                x = [4, 5, 6];
                x.push(7);
                console.log(x);
                }

                const a = [1, 2, 3];
                foo(a); // 分别输出：[ 1, 2, 3, 4 ] [ 4, 5, 6, 7 ]
                console.log(a); // [ 1, 2, 3, 4 ]
            ```
	- 我们无法自行决定使用复制赋值还是引用赋值，**一切由值的类型决定**

## 第三章：原生函数
- JS 常用的原生函数
	- String()
	- Number()
	- Boolean()
	- Array()
	- Object()
	- Function()
	- RegExp()
	- Date()
	- Error()
	- Symbol()
- 3.1 内部属性 `[[Class]]`
	- 所有 typeof 返回为 “object” 的对象都包含内部属性 `[[Class]]`，可以通过 toString 方法查看
        ``` js
            console.log(Object.prototype.toString.call([1, 2, 3])); // [object Array]
            console.log(Object.prototype.toString.call(() => {})); // [object Function]
        ```
- 3.2 封装对象包装
	- 基本类型没有如：`length` 属性和 `toString()` 方法，**JS 会自动为基本类型包装一个封装对象**
	- 浏览器已经为封装对象做优化，写代码时不用考虑提前包装，不然可能降低执行效率
- 3.3 拆封
	- 可以使用 `valuOf()` 获取封装对象中基本类型的值
        ``` js
            let a = new String("abc");
            let b = new Number(12);
            let c = new Boolean(true);

            console.log(a.valueOf()); // abc
            console.log(b.valueOf()); // 12
            console.log(c.valueOf()); // true
        ```
- 3.4 原生函数作为构造函数
	- 尽量不要使用构造函数的方式创建：数组、对象、函数、正则表达式，容易造成意想不到的问题
	- 尽量不要创建和使用空单元数组
	- 对于 Date 和 Error 必须使用原生函数创建（因为没有对应的常量形式）
	- Symbol
		- 具有唯一性的特殊值，用于声名对象属性不容易导致重名
		- 使用 Symbol() 原声构造函数自定义符号**不能**带 `new` 关键字
		- 注意：Symbol 不是对象，而是**一个基本类型**
	- 原生原型
		- 可以将 `Array.prototype.join()` 写作 `Array#join()`
		- 三个特殊的默认类型
			- Function.prototype 默认是一个函数
			- RegExp.prototype 默认是一个正则表达式
			- Array. prototype 默认是一个数组
			- 默认值在使用的时候只创建一次，可以节约资源

## 第四章：强制类型转换
- 4.1 值类型转换
	- 类型转换：值的类型从一种类型转换为另一种类型，为显式转换
	- 隐式的类型转换即强制类型转换
	- 类型转换（显示转换）发生在静态类型语言编译阶段，强制类型转换发生在动态类型语言运行时
    ``` js
        const a = 31;
        console.log(a + ""); // 隐式类型转换
        console.log(String(a)); // 显式类型转换
    ```
- 4.2 抽象值操作
	- 抽象操作 ToString
		- null -> "null"
		- undefined -> "undefined"
		- true -> "true"
		- 6 -> "6"
		- 数组特殊：[1,2,3] -> "1,2,3"
		- `JSON.stringfy()` 转化为字符串也用了 ToString
			- 结果总是字符串：`JSON.stringify("42"); // ""42"" 包含双引号`
			- 字符串、数字、布尔值、null 的规则和 TOString 相同
			- 遇到 undefined、function、symbol 会自动忽略，在数组中出现前面三个则返回 null
			- 包含循环引用会报错
			- 如果对象存在 `toJSON()` 方法，调用 `JSON.stringfy()` 方法会使用该函数的返回值，返回：一个能够被字符串化的安全的 JSON 值
	- 抽象操作 ToNumber
		- 数字 -> 自身
		- 布尔值：false -> 0，true -> 1
		- null -> 0
		- undefined -> NaN
		- 对象 -> 抽象操作 ToPrimitive 规则
	- 抽象操作 ToBoolean
		- undefined、null、false、""、0 / -0、NaN -> false
		- 上述以外 -> true
	- 抽象操作 ToPrimitive
		- 如果有 `Symbol.toPrimitive` 方法，优先调用
		- 调用 `valueOf()` 方法，如果转换为基础数据类型则返回
		- 调用 `toString()` 方法，如果转换为基础数据类型则返回
		- 以上三种没有转换成功则报错

- 4.3 显式强制类型转换
	- 字符串、数字间的相互转换
		- 使用 `String()`、`Number()`、`.toString()` 方法
		- 使用 `+` 可以将字符串转化为数字
		- 字符串 -> 数字
			- 如果只有数字 -> 十进制数字
			- 如果包含有效浮点数数字 -> 浮点数数字
			- "" -> 0
			- 以上三种以外为 NaN
		- parseInt(string, radix) 方法
			- 如果 string 开头是 x / X -> 16 进制数字，开头是 0 -> 8 进制数字
			- 最好将 radix 显式设置为 10，不然遇到 08、09 的情况会被转化为 0，（08、09 不是有效的 10 进制数字）
			- ES5 之后默认转化为 10 进制
	- `+` 可以将日期显示转化为数字，比如获取当前时间戳：`+new Date()`，但做好还是使用 `new Date().getTime()` 和 `Date.now()` 的方式
	- `~` 非运算符
		- `~x` 大致等同于 `-(x+1)`，`console.log(~42); // 43`
		- `~-1` -> `0`，可以用于如：`indexOf()` 方法返回为 -1 情况
	- 显示转化为布尔值
		- 使用 `Boolean()` 方法
		- 使用 `!!` ，第一个 `!` 将值显式转化为布尔值，第二个 `!` 将结果反转回原值

- 4.4 隐式强制类型转换
	- 字符串与数字之间隐式强制类型转换
		- 使用 `+` 时，如果一个操作数是字符串（对象通过 ToPrimitive 转化为字符串），则进行字符串拼接，否则执行数字加法
            ``` js
                console.log(1 + 2); // 3
                console.log(1 + "2"); // 12
                console.log([1, 2] + [3, 4]); // 1,23,4
            ```
		- `数字 + ""` 将数字转化为字符串，使用是 `valueOf()` 方法
		- 使用 `String(数字)` 的方式将数字转化为字符串使用的是 `toString()` 方法
		- 所以在定制 `valueOf()` 和 `toString()` 方法要注意，因为会影响强制类型转换的结果
		- `字符串 - 0` 可以将字符串转化为数字
	- 注意：`[] + {}` 和 `{} + []`，它们返回不同的结果，分别是 `[object Object]` 和 0
		- `{}` 出现在 + 运算符表达式中，因此它被当作一个值（空对象）来处理。 `[]` 会被强制类型转换为 `""`，而 {} 会被强制类型转换为 `[object Object]`
		- `{}` 被当作一个独立的空代码块（不执行任何操作），代码块结尾不需要分号，最后+ [] 将 `[]` 显式强制类型转换为 0
	- 布尔值 -> 数字的隐式强制类型转换
		- undefined、null、false、""、0 / -0、NaN 在加法运算时会转换为 0，其他转化为 1
	- 转换为布尔值的隐式强制类型转换
		- 以下五种情况非布尔值会被强制转换为布尔值
			- `if()` 判断表达式
			- `for ( .. ; .. ; .. )` 语句中的第二个条件判断表达式
			- `while()` 和 `do...while()` 判断表达式
			- `?:` 判断表达式
			- `||` 和 `&& ` 判断表达式
	- `&&` 和 `||` 运算符的返回值并不一定是布尔类型，而是两个操作数其中一个的值
		- `let a = b || "123";` 控制合并运算符，如果 b 还没有赋值，那么 a 默认为 123
		- `a && foo()` 等价于 `if (a) { foo() };`
	- Symbol 类型允许显式强制类型转换，但是隐式强制类型转会产生错误
        ``` js
            let s1 = Symbol("cool");
            String(s1); // "Symbol(cool)"
            let s2 = Symbol("not cool");
            s2 + ""; // TypeError
        ```
- 4.5 宽松相等和严格相等
	- **`==` 允许在相等比较中进行强制类型转换，而 `===` 不允许**
	- 抽象相等（`==` 的行为）
		- 如果两个值的类型相同，就仅比较它们是否相等
			- 注意：`NaN` 不等于 `NaN`，`+0` 不等于 `-0`
		- 两个对象指向同一个值时即视为相等，不发生强制类型转换
		- `==` 在比较两个不同类型的值时会发生隐式强制类型转换，将其中之一或两者都转换为相同的类型后再进行比较
		- 在 `==` 中 null 和 undefined 相等

- 4.6 抽象关系比较
	- 对于 `a < b` 的比较规则
		- 双方先调用 ToPrimitive 转化为字符串，
			- 如果存在数字就转化为数字比较
			- 如果双方都是字符串就按字母顺序比较

## 第五章：语法
- 5.1 语句和表达式
	- 语句和表达式示例
        ``` js
            let a = 3 * 5; // 声明语句
            let b;
            b = a; // 赋值表达式
            b; // 表达式语句
        ```
	- 语句都有一个结果值，结果值也包括 undefined
		- 在浏览器 console 输入语句，默认会显示最后一条语句的结果值
	- 表达式的副作用：造成其他的改变
		- 函数调用产生的副作用
            ``` js
                function foo() {
                a = a + 1;
                }

                var a = 1;
                foo(); // 结果值：undefined，副作用：a 的值被改变
            ```
		- `delete` 操作对象的副作用是属性从对象中被删除
	- 上下文规则
		- 同样的语法在不同的情况会有不同的解释
		- 大括号 `{}` 规则
			- 定义对象常量
				- a 是赋值的对象（左值），{...} 好似所赋予的值（右值）
                    ``` js
                        let a = {
                        foo: bar() // 假设 bar 已经声明
                        }
                    ```
			- 标签
				- `{}` 在此是为一个普通的代码块
				- 标签语句：`foo` 是 `bar()` 的标签，即通过 `foo` 能够跳转到 `bar()` 函数
					- 比如 `break` 语句可以从内层循环条装到外层循环或者结束循环，所以 `break` 也是一个标签
                        ``` js
                            {
                            foo: bar() // 假设 bar 已经声明
                            }
                        ```
			- 对象解构
			- `if...else` 的代码块
- 5.2 运算符优先级
	- `,` 连接一系列语句时，它的优先级最低
	- `&&` 运算符的优先级 > `=`
	- `&&` 运算符优先级 > `||`
	- 短路特性：进行 `&&` 或 `||` 判断时，如果左边的值为 false 或 true，则不需要对右边的值判断
	- 三元运算符的执行方式
		- `a ? b : c ? d : e;` 等价于 `a ? b : (c ? d : e)`
- 5.3 自动分号
	- 分号自动插入（Automatic Semicolon Insertion，ASI）：JS 会自动为代码补上缺失的分号
- 5.4 错误
	- 暂时性死区（Temporal Dead Zone，暂时性死区）：代码中的变量还没有初始化不能被引用的情况
- 5.5 函数参数
	- 不要同时访问命名参数和其对应的arguments数组单元
- 5.6 try...finally
	- 如果finally中抛出异常（无论是有意还是无意），函数就会在此处终止。如果此前try中已经有return设置了返回值，则该值会被丢弃
- 5.7 switch
	- switch 使用的是 `===` 严格比较 是否和 true 相等，所以如果结果返回不是 true 可能造成其他问题
        ``` js
            var a = "hello world";
            var b = 10;
            switch (true) {
            case a || b == 10: // 返回的是 "hello world"
                // 永远执行不到这里
                break;
            default:
                console.log("Oops");
            }
        ```

## 附录A：混合环境 JS
- JavaScript 语言的官方名称是 ECMAScript，JavaScript 是该规范在浏览器上的实现
- 由于浏览器兼容性问题存在可能导致与官方规范的差异
	- 在非严格模式中允许八进制数值常量存在，如0123（即十进制的83）
	- `window.escape(..)` 和 `window.unescape(..)` 能够转义（escape）和回转（unescape）带有%分隔符的十六进制字符串。例如，`window.escape( "? foo=97%&bar=3%" )` 结果为 `"%3Ffoo%3D97%25%26bar%3D3%25"`
	- `String.prototype.substr` 第二个参数是结束位置索引（非自包含）， `String.prototype.substring`  第二个参数是长度（需要包含的字符数）
- 宿主对象
	- 内建对象和函数，比如：DOM 元素，内部的 `[[class]]` 来自预定义属性
	- 和普通对象的行为差异
		- 无法正常访问 object 的内建方法，如 `toString()`
		- 无法写覆盖
		- 包含一些预定义的只读属性
		- 包含无法将 this 重载为其他对象的方法
- 声明一个全局变量的结果不仅仅是创建一个全局变量，而且还会在 global 对象（在浏览器中为window）中创建一个同名属性
	- 由于浏览器历史问题，**在创建带有 id 属性的 DOM 元素时也会创建同名的全局变量**
- 不要扩展原生原型，可能产生冲突
- 使用 `<script> .. </script>` 引入的脚本，共享 global 对象（浏览器中的 window），但是全局变量作用域的提升机制在此时不适用
	- 下面的代码都无法运行（foo() 还未被声明）
        ``` html
            <script>
                foo();
            </script>
            <script>
                function foo() { .. }
            </script>
        ```