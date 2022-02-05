---
title: 你不知道的 JS 学习笔记：this和对象原型
date: 2022-02-05 20:18:01
categories: 前端
tags: JavaScript
img: https://notesimgs.oss-cn-shanghai.aliyuncs.com/img/20220204150854.jpg
---
## 第一章：关于 this
- 误解
	- this 不是指向函数自身
	- this 在任何情况下都不指向函数的词法作用域
- this 是函数被调用时发生的绑定，**指向什么完全取决于在哪里被调用**

## 第二章 this 全面解析
### 2.1 调用位置
- 确认函数的调用位置的方式是：分析调用栈

### 2.2 绑定规则
- 默认绑定
	- 在没有其他规则时，非严格模式，this 默认指向全局变量，严格模式，this 指向 undefined
	- 非严格模式示例
	  ```js
	  function foo() {
		console.log( this.a );
	  }

	  var a = 2;

	  foo(); // 2
	  ```
- 隐式绑定
	- 当函数引用有上下文时，隐式绑定规则会把函数调用中的 this 绑定到上下文对象
	- 示例代码
	  ```js
	  function foo() {
		console.log( this.a );
	  }

	  var obj = {
		a: 2,
		foo: foo
	  }

	  obj.foo(); // 2
	  ```
	- 隐式丢失：丢失隐式绑定对象，从而使用默认绑定
	- 非严格模式的隐式丢失示例
		- 此时 bar 引用的 foo 函数本身
	  ```js
	  function foo() {
		console.log(this.a);
	  }

	  var obj = {
		a: 2,
		foo: foo,
	  };

	  var bar = obj.foo;

	  var a = "global";

	  bar(); // 2

	  ```
	- 传入回调参数是同样会存在隐式丢失问题
	  ```js
	  function foo() {
		console.log(this.a);
	  }

	  function doFoo(fn) {
		// fn 引用的就是 foo
		fn();
	  }

	  var obj = {
		a: 2,
		foo: foo,
	  };

	  var bar = obj.foo;

	  var a = "global";

	  doFoo(obj.foo);
	  ```
- 显式绑定
	- 使用 call 函数或者 apply 函数实现
	- 第一个参数是一个对象，把对象绑定到 this，调用函数时再指定这个 this
	- call 函数示例
	  ```js
	  function foo() {
		console.log(this.a);
	  }

	  var obj = {
		a: 2,
	  };

	  foo.call(obj); // 2
	  ```
	- 硬绑定：创建函数并在内部手工调用 call 或 apply，强制把函数的 this 绑定到对象
		- 应用：创建包裹函数
	  ```js
	  function foo(something) {
		console.log(this.a, something);
		return this.a + something;
	  }

	  var obj = {
		a: 2,
	  };

	  var bar = function () {
		return foo.apply(obj, arguments);
	  };

	  var b = bar(3); // 2 3
	  console.log(b);
	  ```
		- 应用：bind 函数：把参数设置为 this 到上下文并调用原始函数，返回一个新函数
	  ```js
	  function foo(something) {
		console.log(this.a, something);
		return this.a + something;
	  }

	  var obj = {
		a: 2,
	  };

	  var bar = foo.bind(obj); // bar 是一个新的函数，this 指向 obj

	  var b = bar(3); // 2 3
	  console.log(b);
	  ```
- new 绑定
	- new 调用函数（发生构造函数调用）过程
		- 创建一个全新的对象
		- 新对象被执行`[[原型]]`连接
		- 新对象被绑定到函数调用的 this
		- 如果函数没有返回其他对象，new 表达式的函数调用自动返回这个新对象
	- 示例代码：new 操作符会构造一个新对象并绑定到 foo 调用的 this 上
	  ```js
	  function foo(a) {
		this.a = a;
	  }

	  var bar = new foo(2);
	  console.log(bar.a); // 2
	  ```

### 2.3 优先级
- 显式绑定优先级 > 隐式绑定
- new 绑定优先级 > 隐式绑定
- **判断 this 规则的方式**
	- 是否在 new 中调用，是的话 this 绑定的是新创建的对象
	- 是否通过 call、apply、bind 的显示绑定，是的话 this 绑定的是指定的对象
	- 是否在某个上下文对象中调用绑定，是的话 this 绑定的是上下文对象
	- 以上三种都不是的话，使用默认绑定，严格模式帮定至 undefined，非严格模式绑定到全局对象
	- ![](https://notesimgs.oss-cn-shanghai.aliyuncs.com/img/20220205200347.png)

### 2.4 绑定例外
- 如果把 null / undefined 作为 this 绑定对象传入 call、apply、bind，这些 null / undefined 在调用时会被忽略，使用的是隐式绑定
	- 应用：apply 展开数组 or 函数柯里化，传入一个 null 作为占位符
	```js
	  function foo(a, b) {
		console.log(`a: ${a}, b: ${b}`);
	  }

	  // 数组展开为参数
	  foo.apply(null, [2, 3]); // a: 2, b: 3

	  // 使用 bind 进行柯里化
	  var bar = foo.bind(null, 2); // a: 2, b: 5
	  bar(5);
	  ```
	- 可以使用 `Object.create(null)` 创建空对象（不会创建 `Object.prototype`，比 `{}` 更空），称作 DMZ 对象
- 间接引用情况会导致绑定例外
  ```js
  function foo(a) {
	console.log(a);
  }

  var a = 2;
  var o = { a: 3, foo: foo };
  var p = { a: 4 };
  o.foo(); // 3
  // 注意：返回值是目标函数的引用，相当于直接调用 foo 函数
  (p.foo = o.foo)(); // 2
  ```
- 软绑定：可以手动指定 this，否则应用默认隐式绑定或默认绑定
  ```js
  if (!Function.prototype.softBind) {
	Function.prototype.softBind = function (obj) {
	  var fn = this;
	  // 获取所用 curried 参数
	  var curried = [].slice.call(arguments, 1);
	  var bound = function () {
		return fn.apply(
		  !this || this === (window || global) ? obj : this,
		  curried.concat.apply(curried, arguments)
		);
	  };
	  bound.prototype = Object.create(fn.prototype);
	  return bound;
	};
  }
  ```

### 2.5 this 词法
- **箭头函数不使用 this 的四种规则，而是根据外层（函数或者全局）作用域决定**
  - 箭头函数会继承外层函数调用的 this 绑定
  - 内部的箭头函数会捕获调用时 foo 的this，而 foo 的 this 被绑定到 obj1
  ```js
  function foo() {
	return (a) => {
	  // 注意：this 继承自 foo
	  console.log(this.a);
	};
  }

  var obj1 = { a: 2 };
  var obj2 = { a: 3 };

  var bar = foo.call(obj1);
  bar.call(obj2); // 2
  ```

## 第三章：对象
### 3.1 对象
- 可以通过 `{}` 或者 new 关键字声明对象

### 3.2 类型
- 内置对象
	- String
	- Number
	- Boolean
	- Object
	- Function
	- Array
	- Date
	- RegExp
	- Error
- JS 会自动把字面量转换为一个对象
	- 比如：自动将字符串字面量转会为 String 对象，从而可以访问 String 对象的方法
- null 、undefined 只有文字形式
- Date 只有构造形式（对象）
- Object、Array、Function、RegExp，只有构造形式，都是对象

### 3.3 内容
- 可以通过 `.` 或者 `[]` 来访问对象中的属性，
- 属性名都是字符串，所以传入的值会被自动转化为字符串
- 可计算属性名称
	- 使用 [] 包裹的表达式作为属性名
	- 常用于 Symbol
  ```js
  var MyObject = {
	[Symbol.Something]: "wujieli"
  }
  ```
- 复制对象
	- 浅拷贝
		- 引用类型还是指向原来的对象
		- `JSON.parse(JSON.stringify(someObj))` 和 `Object.assign()` 可以实现浅拷贝
	- 深拷贝
		- 引用类型复制一套独立的
- 属性操作符（数据描述符）
	- 除 value 外，还包括：writable（可写）、enumerable（可枚举）、configurable（可配置）
		- writable 为 false 则不可修改
		- configurable 为 false 则不可以通过 `Object.defineProperty()` 修改属性描述符，不能删除属性
		- enumerable 为 false，属性不会出现在循环枚举中
	- Object.defineProperty()： 添加新属性或者修改已有属性
	  ```js
	  var obj = {};

	  Object.defineProperty(obj, "a", {
		value: 2,
		writable: true,
		configurable: true,
		enumerable: true,
	  });

	  console.log(obj.a); // 2
	  ```
- 对象不变性
	- 通过 `writable: false` 和 `configurable: false` 可以创建一个常量属性，不可修改、重定义、删除
	- Object.preventExtensions( obj ) ：禁止添加新属性
	- Object.seal( obj ) ：创建一个密封对象，在现有对象调用 `Object.preventExtensions` 且 `configurable: false`
	- Object.freeze( obj ) ：现有对象调用 `Object.seal` 且 `writable: false`
- `[[get]]` 属性
	- 在对象中查找同名属性，找到了就返回
	- 如果没找到就根据原型链找，找不到则返回 undefined
- `[[put]]` 属性
	- 属性是否是访问描述符，如果是并存在 setter 就调用 setter
	- writable 是否为 false，是 false 的话非严格模式静默失败，严格模式抛出 TypeError 异常
	- 以上都不是，将值设置为该属性的值
- 访问描述符
	- 通过 getter 获取属性，通过 setter 设置属性，通常成对出现
	- 访问描述符只有：set、get、configurable、enumerable 属性
- 属性存在性
	- `[属性名称]` in obj：in 关键可以检查属性是否存在与对象，找不到会查找对象的原型链
	- Object.hasOwnProperty()：查找对象是否包含属性，不会查找原型链

### 3.4 遍历
- for...in 循环：遍历对象可枚举属性，包括原型链
- for...of 循环：循环遍历对象的所有 value
	- 向被访问对象请求一个迭代器，通过迭代器对象的 next() 方法实现遍历所有值
	- 数组内置 `@@iterator` 返回迭代器对象的函数
	- 普通对象没有 `@@terator` 目的是为了避免影响未来对象类型


## 第四章：混合对象“类”
### 4.1 类理论
- 数据及对数据的操作应该封装打包作为数据结构
- 使用**类（class）**对数据结构进行分类
- 类的核心概念
	- 实例化：类虽然有相同的属性或方法，但是实例中的数据可能不同
	- 继承：类的属性或方法不用在子类重复定义，而是直接继承父类的属性或方法
	- 多态：父类通用行为可以被子类更特殊行为重写，从而扩展子类的行为

### 4.2 类的机制
- 如果把类比做建筑中的图纸，通过图纸（类）建造出来的房子就是实例
- 构造函数：
	- 用于构造类实例，一个特殊的类方法，通常和类同名
	- 返回一个对象（即：类实例）

### 4.3 类的继承
- 子类和父类是完全不同的类，子类会包含**父类原始行为的副本**，但也可以重复父类的行为甚至定义新的行为
- 多态：
	- 子类可以重写父类方法
	- 继承链中不同层次的方法名可以被多次定义
- 子类可以相对引用它继承的父类，这种相对引用称为 super
- JS 自身不提供多重继承

### 4.4 混入
- JS 中对象没有自动复制的行为
- 显式混入
	- 如果子对象中不存在对应属性则复制父亲对象属性
	  ```js
	  function mixin(souceObj, targetObj) {
		for (let key in souceObj) {
		  // 只会在不存在的情况下复制
		  if (!(key in targetObj)) {
			targetObj[key] = souceObj[key];
		  }
		}

		return targetObj;
	  }

	  let Vehicle = {
		engines: 1,

		ignition: function () {
		  console.log("发动引擎！");
		},
		drive: function () {
		  this.ignition();
		  console.log("启动！");
		},
	  };

	  let Car = mixin(Vehicle, {
		wheels: 4,

		drive: function () {
		  Vehicle.drive.call(this); // 显式多态
		  console.log(`启动${this.wheels}个轮子的这辆车`);
		},
	  });

	  console.log(Car.drive());
	  ```
	- 显示混入的变体：寄生继承
		- 先通过潜拷贝获取父对象，再添加新的方法
- 隐式混入
	- 通过 `call(this)` 方法把父对象方法绑定到子对象
- 尽量避使用混入，因为复制的是函数的引用而不是自身，可能会造成隐患

## 第五章：原型
### 5.1 [[Prototype]]
- `[[Prototype]]`：JS 对象的内置属性，是对于其他对象的引用
- 对于属性查找操作（如：`[[Get]]`，for...in，in），**如果在对象本身找不到需要的属性，就通过 `[[Prototype]]` 访问对象的原型链向上查找**，找不到就返回 undefined
- 所有普通的 `[[Prototype]]` 最终都会指向内置的 Object.prototype
- 原型链 = `[[Prototype]]` 链
- 属性设置
	- 对于 `obj.foo = "bar";` 赋值语句来说，如果 foo 属性不是存在 obj 自身，就会通过 `[[Prototype]]` 查找原型链，如果原型链找不到则直接赋值在 obj 上
	- 屏蔽属性：如果 foo 同时存在于 obj 和其原型链，则 obj 会屏蔽所有原型链上的所有 foo 属性（即：**选择最底层的属性**），但分为三种情况讨论
		- 原型链属性 `writable: true` ，会直接在底层对象新增一个属性，**属于屏蔽属性**
		- 原型链属性 `writable: false` ，原型链属性无法修改，也无法在底层对象新增属性，严格模式会报错，非严格模式会静默忽略赋值
		- 原型链存在该属性并且是一个 setter，则直接调用该 setter

### 5.2 ”类“
- JS 中不会把一个对象（类）复制到另一个对象（实例），**只是关联起来**
- `new Foo()` 会生成一个新对象，新对象的内部链接 `[[Prototype]]` 关联到的是 `Foo.prototype` 对象
- ![](https://notesimgs.oss-cn-shanghai.aliyuncs.com/img/20220205201345.png)
- 构造函数：
	- 函数原型的 construtor 默认指向自己，即：`Foo.prototype.constructor === Foo` 是 true
	- 调用 `new` 创建的对象的 constructor 属性指向 -> 创建这个对象的函数
	- 参考代码
	  ```js
	  function Foo() {}

	  console.log(Foo.prototype.constructor === Foo); // true

	  var a = new Foo();
	  console.log(a.constructor === Foo); // true
	  ```
	- JS 中的函数就是普通函数
	- new 会劫持所有所有的普通函数，并通过构造对象调用它
	- 注意：`.constructor` 仅仅是一个不可枚举，但是可以修改或配置的属性，因此在创建对象时可以被覆盖，也就不是上面的等式了

### 5.3 （原型）继承
- 在 ES6 之前，将子对象的 prototype 通过 `Object.create()` 指向父亲对象
- ES6 可以通过 `Object.setPrototypeOf` 直接修改子对象原型，两种方式效果相同
  ```js
  // ES 6 之前
  Bar.prototype = Object.create( Foo.prototype );
  // ES 6
  Object.setPrototypeOf( Bar.prototype, Foo.prototype );
  ```
- 查找”类“关系
	- 反射（内省）：查找一个实例（JS 中的对象）的继承祖先（JS 中的委托关联）
	- 通过 instanceof 查找反射（不建议使用）：`a instanceof Foo;` ，a 的整条原型链是否有指向 Foo.prototype 的对象
	- 通过 `Foo.prototype.isPrototypeOf( a );` 查找反射：a 的原型链是否出现过 Foo.prototype
	- `.__proto__` 不是一个属性，而是一个 getter / setter
		- 通过 ES6 的方式实现 `.__proto__` 参考
	 ```js
	  Object.defineProperty(Object.prototype, "__proto__", {
		get: function () {
		  return Object.getPrototypeOf(this);
		},
		set: function (o) {
		  Object.setPrototypeOf(this, o);
		  return o;
		},
	  });
	  ```

### 5.4 对象关联
- `let bar = Object.create( obj )` 可以将新对象 bar 原型指向 obj
- Object.create() 在 ES5 中的实现代码
  ```js
  if (!Object.create) {
	Object.create = function(o) {
	  function F(){}
	  F.prototype = o;
	  return new F();
	}
  }
  ```


## 第六章：行为委托
- 在面向类的设计模式中，鼓励使用继承和多态，通常先定义一个父类和通用方法，再定义子类和子类的特有方法，或者重写父类方法
- 委托理论
    - 对象找不到属性或方法时，会把这个请求委托给另一个对象，对象间是兄弟关系
    - 定义的都是对象，一个对象通过 `Object.create()` 创建，把 `[[Prototype]]` 委托给另一个对象
    - 示例代码
      ```js
      Task = {
      setID: function (ID) {
          this.id = ID;
      },
      outputID: function () {
          console.log(this.id);
      },
      };
      // 让XYZ委托Task
      XYZ = Object.create(Task);

      XYZ.prepareTask = function (ID, Label) {
      this.setID(ID);
      this.label = Label;
      };
      XYZ.outputTaskDetails = function () {
      this.outputID();
      console.log(this.label);
      };
      ```
- 这类编码风格称为 对象关联（OLOO，objects linked to other objects）
	- 对于实例化后的属性数据都存储于子对象上
	- 尽量避免原型链上存在相同的命名
	- 子对象包含 this 的方法在调用原型链上的方法是，触发了隐式绑定，this 还是指向子对象
- 禁止双向委托

## 附录：ES6 中的 Class
- ES6 中的 class 语法糖解决的问题
	- 不再使用 .prototype
	- 子类通过 extends 直接继承父类，不需要再通过 Object.create()
	- 可以通过 super() 实现相对多态，任何方法都可以引用原型链上的同名方法
	- class 语法不能声明属性（需要通过 constructor），避免错误
- class 语法糖存在的问题
	- 如果需要跟踪实例间的共享属性，只能使用 .prototype 的方式
	- super 不是动态绑定的，而是在声明时静态绑定的
