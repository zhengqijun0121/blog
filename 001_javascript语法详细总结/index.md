# JavaScript 语法详细总结


# JavaScript 语法详细总结
JavaScript（简称JS）是一门**弱类型、解释型、单线程、基于原型**的脚本语言，遵循ECMAScript（ES）语言标准，是Web前端开发的核心语言，同时可通过Node.js实现服务端开发。本文基于最新ES标准（ES2015+），从基础到进阶全面梳理JS核心语法。

---

## 一、基础语法结构
### 1. 语句与分号
JS以语句为基本执行单元，语句结尾的分号`;`为可选特性，JS引擎会通过**自动分号插入(ASI)** 机制补全，但强烈建议手动添加，避免换行导致的语法歧义。
```javascript
// 合法语句
let a = 10;
console.log(a);

// 无分号也可执行（不推荐）
let b = 20
console.log(b)
```

### 2. 注释
用于代码说明，不会被引擎执行，分为两种：
```javascript
// 单行注释：双斜杠开头，仅对当前行生效

/*
  多行注释：斜杠+星号开头，星号+斜杠结尾
  可跨越多行，适合大段说明
*/
```

### 3. 标识符与命名规范
标识符是变量、函数、属性名的合法名称，命名规则如下：
1.  首字符必须是**字母、下划线`_`、美元符号`$`**，不能以数字开头
2.  后续字符可包含字母、数字、下划线、美元符号
3.  严格区分大小写（`age`和`Age`是两个不同标识符）
4.  不能使用JS关键字/保留字（如`let`、`function`、`class`等）

**行业通用命名规范**：
- 变量、函数、方法：小驼峰式（`userName`、`getUserInfo`）
- 常量：全大写+下划线（`MAX_SIZE`、`BASE_URL`）
- 类、构造函数：大驼峰式（`User`、`OrderService`）

### 4. 严格模式
ES5引入的严格模式，用于修正JS语法的不合理设计，提升代码安全性和执行效率，禁用部分不安全语法。
- 开启方式：在脚本/函数顶部添加字符串 `"use strict";`
- 核心规则：
  1.  禁止未声明的变量直接赋值（避免意外创建全局变量）
  2.  禁止修改只读属性、不可扩展对象
  3.  禁止函数参数重名、`with`语句
  4.  普通函数内`this`指向`undefined`（非严格模式指向全局对象）
  5.  禁用八进制字面量、`arguments`对象的自动同步

```javascript
// 全局开启严格模式
"use strict";
num = 10; // 报错：num未声明

// 函数内开启严格模式（推荐，避免全局污染）
function fn() {
  "use strict";
  let a = 10;
}
```

---

## 二、变量声明
JS提供3种变量声明方式，核心差异体现在作用域、提升规则、可修改性上。

| 特性 | var | let | const |
| :--- | :--- | :--- | :--- |
| 作用域 | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 声明提升，初始化不提升（值为undefined） | 无提升，存在暂时性死区 | 无提升，存在暂时性死区 |
| 重复声明 | 同一作用域内允许 | 同一作用域内禁止 | 同一作用域内禁止 |
| 重新赋值 | 允许 | 允许 | 不允许修改引用地址 |
| 全局声明 | 挂载到window对象 | 不挂载到window对象 | 不挂载到window对象 |

### 1. var（ES5及之前，不推荐使用）
存在变量提升、无块级作用域、易污染全局等问题，现代开发已基本被let/const替代。
```javascript
console.log(a); // 不报错，输出undefined（变量提升）
var a = 10;

// 无块级作用域
if (true) {
  var b = 20;
}
console.log(b); // 输出20，块内声明的变量泄露到全局
```

### 2. let（ES6+，变量声明首选）
拥有块级作用域，解决了var的核心缺陷，适用于值会发生变化的变量。
```javascript
// console.log(c); // 报错：Cannot access 'c' before initialization
let c = 30;

// 块级作用域生效
if (true) {
  let d = 40;
}
// console.log(d); // 报错：d is not defined
```

### 3. const（ES6+，默认首选）
用于声明**常量**，声明时必须同时赋值，且不能修改变量的引用地址（并非值不可变）。
```javascript
// 声明必须赋值
// const e; // 报错：Missing initializer in const declaration
const e = 50;
// e = 60; // 报错：Assignment to constant variable

// 引用类型：可修改属性值，不可修改引用地址
const obj = { name: "张三" };
obj.name = "李四"; // 合法
// obj = { name: "王五" }; // 报错：修改了引用地址
```

**开发最佳实践**：默认使用const，仅当确定变量值会发生变化时使用let，完全禁用var。

---

## 三、数据类型
JS数据类型分为**7种原始类型（基本类型）** 和**1种引用类型**，共8大类型。

### 1. 原始类型（值类型）
原始类型的值存储在栈内存中，按值访问，不可变（修改值本质是创建新值）。

| 类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| String | 字符串类型，用于表示文本，单引号/双引号/反引号包裹 | `'hello'`、`"world"`、`` `name:${name}` `` |
| Number | 数字类型，包含整数、浮点数，范围±2^53-1，存在精度问题 | `10`、`3.14`、`NaN`、`Infinity` |
| BigInt | 大整数类型，用于表示超出Number安全范围的整数，结尾加n | `9007199254740993n`、`10n` |
| Boolean | 布尔类型，仅两个值：真/假 | `true`、`false` |
| Undefined | 未定义类型，仅一个值undefined，声明未赋值的变量默认值 | `let a; console.log(a); // undefined` |
| Null | 空类型，仅一个值null，表示空对象指针 | `let obj = null;` |
| Symbol | 符号类型，ES6新增，用于创建唯一、不可变的值，常作为对象属性名 | `Symbol('id')`、`Symbol.iterator` |

#### 关键注意点
1.  **NaN**：属于Number类型，表示"非数字"，特点是`NaN !== NaN`，判断NaN需用`Number.isNaN()`
2.  **null与undefined的区别**：
    - null：主动赋值的空值，代表"空对象"，typeof null === 'object'（历史bug）
    - undefined：被动的未定义，代表"缺少值"，变量声明未赋值、函数无返回值、对象不存在的属性均返回undefined
3.  **Symbol的唯一性**：即使传入相同描述，两个Symbol也不相等：`Symbol('a') === Symbol('a') // false`

### 2. 引用类型
引用类型的值存储在堆内存中，栈内存仅存储指向堆内存的引用地址，按引用访问，值可变。核心是`Object`类型，其派生类型包括：
- 基础引用：`Object`、`Array`、`Function`
- 特殊引用：`Date`、`RegExp`、`Map`、`Set`、`WeakMap`、`WeakSet`
- 错误类型：`Error`、`TypeError`、`ReferenceError`等

```javascript
// 引用类型：赋值传递的是引用地址
let obj1 = { age: 20 };
let obj2 = obj1;
obj2.age = 30;
console.log(obj1.age); // 30，两个变量指向同一个堆内存地址
```

### 3. 类型判断
#### （1）typeof
用于判断原始类型（null除外）和函数，返回字符串类型名。
```javascript
typeof 'abc' // 'string'
typeof 123 // 'number'
typeof true // 'boolean'
typeof undefined // 'undefined'
typeof Symbol() // 'symbol'
typeof 10n // 'bigint'
typeof function(){} // 'function'
typeof {} // 'object'
typeof [] // 'object'
typeof null // 'object'（历史bug，无法修正）
```

#### （2）instanceof
用于判断引用类型的原型归属，检测构造函数的`prototype`是否出现在实例的原型链上。
```javascript
[] instanceof Array // true
{} instanceof Object // true
new Map() instanceof Map // true
function(){} instanceof Function // true
```

#### （3）Object.prototype.toString.call()
**最精准的类型判断方法**，返回固定格式的`[object 类型名]`字符串，兼容所有类型。
```javascript
Object.prototype.toString.call('abc') // '[object String]'
Object.prototype.toString.call(null) // '[object Null]'
Object.prototype.toString.call([]) // '[object Array]'
Object.prototype.toString.call(function(){}) // '[object Function]'
Object.prototype.toString.call(new Map()) // '[object Map]'
```

### 4. 类型转换
JS是弱类型语言，运算时会自动进行类型转换，分为**显式转换**和**隐式转换**。

#### （1）显式转换
手动调用方法转换类型：
```javascript
// 转数字
Number('123') // 123
Number('abc') // NaN
parseInt('123px') // 123
parseFloat('3.14em') // 3.14

// 转字符串
String(123) // '123'
(123).toString() // '123'

// 转布尔值
Boolean(0) // false
Boolean('abc') // true
```

#### （2）隐式转换
JS引擎自动执行的转换，常见场景：
1.  算术运算：`+` 遇到字符串会转为拼接，`- * / %` 会将值转为数字
    ```javascript
    1 + '2' // '12'（字符串拼接）
    10 - '5' // 5（转数字运算）
    '3' * '4' // 12
    ```
2.  逻辑运算：`if`、`&&`、`||`、`!` 会将值转为布尔值
3.  相等比较：`==` 会先转换类型再比较，`===` 不转换类型，直接比较值和类型

#### （3）falsy值
转为布尔值后为`false`的值，共8个：`false`、`0`、`-0`、`0n`、`''`、`null`、`undefined`、`NaN`，其余所有值均为truthy值。

---

## 四、运算符
### 1. 算术运算符
用于数值计算，包括：`+`（加）、`-`（减）、`*`（乘）、`/`（除）、`%`（取余）、`**`（幂运算，ES7）、`++`（自增）、`--`（自减）。
```javascript
10 % 3 // 1（取余）
2 ** 3 // 8（2的3次方）
let a = 10;
a++; // 后置自增，先返回值再+1，结果10，a变为11
++a; // 前置自增，先+1再返回值，a变为12，结果12
```

### 2. 赋值运算符
用于给变量赋值，基础`=`，复合赋值包括：`+=`、`-=`、`*=`、`/=`、`%=`、`**=`等。
```javascript
let a = 10;
a += 5; // 等价于 a = a + 5，结果15
a *= 2; // 等价于 a = a * 2，结果30
```

### 3. 比较运算符
用于值的比较，返回布尔值。
- 严格比较（推荐）：`===`（全等）、`!==`（不全等），**不转换类型**，同时比较值和类型
- 非严格比较：`==`（相等）、`!=`（不等），会先隐式转换类型再比较值
- 大小比较：`>`、`<`、`>=`、`<=`

```javascript
10 === '10' // false（类型不同）
10 == '10' // true（字符串转数字后比较）
null === undefined // false
null == undefined // true（特殊规则）
NaN === NaN // false（NaN不等于任何值，包括自身）
```

### 4. 逻辑运算符
用于布尔值运算，支持短路特性。
- `&&`（逻辑与）：全真才真，遇假短路，返回第一个假值，全为真返回最后一个真值
- `||`（逻辑或）：全假才假，遇真短路，返回第一个真值，全为假返回最后一个假值
- `!`（逻辑非）：取反，转为布尔值后反转
- `??`（空值合并，ES2020）：仅当左侧为`null/undefined`时，返回右侧值，否则返回左侧值
- `?.`（可选链，ES2020）：安全访问对象深层属性，属性不存在时返回undefined，不报错

```javascript
// 短路特性
true && 10 && 'abc' // 'abc'
0 || false || 20 // 20

// 空值合并与逻辑或的区别
0 ?? 100 // 0（0不是null/undefined，返回左侧）
0 || 100 // 100（0是falsy值，返回右侧）

// 可选链
const obj = { user: { name: '张三' } };
obj.user?.age?.num // undefined，不会报错
obj?.address?.city // undefined
```

### 5. 三元运算符
JS中唯一的三元运算符，用于简化if-else分支，语法：`条件 ? 表达式1 : 表达式2`
```javascript
let age = 18;
let isAdult = age >= 18 ? '成年' : '未成年'; // '成年'
```

### 6. 其他常用运算符
- 逗号运算符`,`：依次执行多个表达式，返回最后一个表达式的结果
- `typeof`：返回值的类型字符串
- `delete`：删除对象的属性或数组元素
- `void`：执行表达式，返回undefined

---

## 五、流程控制语句
### 1. 分支语句
#### （1）if...else 语句
单分支、双分支、多分支判断，满足条件执行对应代码块。
```javascript
// 单分支
if (score >= 60) {
  console.log('及格');
}

// 双分支
if (score >= 60) {
  console.log('及格');
} else {
  console.log('不及格');
}

// 多分支
if (score >= 90) {
  console.log('优秀');
} else if (score >= 60) {
  console.log('及格');
} else {
  console.log('不及格');
}
```

#### （2）switch...case 语句
用于多值匹配场景，基于**全等`===`** 匹配，必须配合`break`避免穿透。
```javascript
let day = 1;
switch (day) {
  case 1:
    console.log('星期一');
    break; // 终止执行，避免穿透
  case 2:
    console.log('星期二');
    break;
  default: // 所有case都不匹配时执行
    console.log('其他');
}
```

### 2. 循环语句
用于重复执行代码块，支持`break`终止循环、`continue`跳过当前循环。

#### （1）for 循环
已知循环次数时首选，语法：`for(初始化表达式; 条件表达式; 更新表达式) { 循环体 }`
```javascript
// 循环输出0-4
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

#### （2）while 循环
先判断条件，再执行循环体，未知循环次数时使用。
```javascript
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}
```

#### （3）do...while 循环
先执行一次循环体，再判断条件，**至少执行一次**。
```javascript
let i = 0;
do {
  console.log(i);
  i++;
} while (i < 5);
```

#### （4）for...in 循环
用于遍历**对象的可枚举属性**（包括原型链上的可枚举属性），不推荐遍历数组。
```javascript
const user = { name: '张三', age: 20, gender: '男' };
for (let key in user) {
  console.log(key, user[key]); // 输出属性名和属性值
}
```

#### （5）for...of 循环
ES6新增，用于遍历**可迭代对象**（数组、字符串、Map、Set、NodeList等），支持break/continue。
```javascript
const arr = [10, 20, 30];
for (let item of arr) {
  console.log(item); // 输出10、20、30
}
```

---

## 六、函数
函数是JS的一等公民，是可重复执行的代码块，同时也是特殊的对象，可作为参数、返回值传递。

### 1. 函数的定义方式
#### （1）函数声明
具备**函数提升**特性，可在声明前调用。
```javascript
// 语法：function 函数名(形参) { 函数体 }
function sum(a, b) {
  return a + b;
}
sum(1, 2); // 3
```

#### （2）函数表达式
将匿名函数赋值给变量，无函数提升，必须赋值后才能调用。
```javascript
const sum = function(a, b) {
  return a + b;
};
sum(1, 2); // 3
```

#### （3）箭头函数（ES6+）
简化函数写法，核心特性是**不绑定自己的this**，继承外层词法作用域的this。
```javascript
// 基础写法
const sum = (a, b) => {
  return a + b;
};

// 简写：单个参数可省略括号，单行返回可省略大括号和return
const double = num => num * 2;
double(10); // 20
```

#### （4）Function 构造函数
不推荐使用，需传入字符串格式的参数和函数体，性能差且有安全风险。
```javascript
const sum = new Function('a', 'b', 'return a + b');
sum(1, 2); // 3
```

### 2. 函数参数
#### （1）形参与实参
形参：函数定义时声明的参数；实参：函数调用时传入的实际值。实参数量可与形参不一致，未传的形参默认值为undefined。

#### （2）默认参数（ES6+）
给形参设置默认值，未传参/传undefined时使用默认值。
```javascript
function sum(a = 0, b = 0) {
  return a + b;
}
sum(1); // 1，b使用默认值0
sum(); // 0，a、b均使用默认值
```

#### （3）剩余参数（ES6+）
`...`语法，将多余的实参收集为一个数组，替代arguments对象。
```javascript
function sum(...nums) {
  return nums.reduce((total, num) => total + num, 0);
}
sum(1, 2, 3, 4); // 10
```

#### （4）arguments 对象
函数内的类数组对象，存储所有实参，箭头函数内无arguments。
```javascript
function sum() {
  console.log(arguments); // [1,2,3] 类数组
  return Array.from(arguments).reduce((a, b) => a + b);
}
sum(1, 2, 3); // 6
```

### 3. 作用域与作用域链
#### （1）作用域
作用域是变量和函数的可访问范围，JS采用**词法作用域（静态作用域）**，作用域在函数定义时就已确定，而非执行时。
- 全局作用域：页面/程序运行全程有效，全局变量可在任意位置访问
- 函数作用域：仅在函数内部有效，函数外无法访问内部变量
- 块级作用域：`{}`包裹的代码块内有效，由let/const实现

#### （2）作用域链
当访问变量时，JS引擎会先在当前作用域查找，找不到则向上级作用域查找，直到全局作用域，这条查找链路就是作用域链。
```javascript
const a = 10; // 全局作用域
function fn1() {
  const b = 20; // fn1函数作用域
  function fn2() {
    const c = 30; // fn2函数作用域
    console.log(a + b + c); // 60，沿作用域链向上查找a、b
  }
  fn2();
}
fn1();
```

### 4. 闭包
闭包是指**有权访问另一个函数作用域内变量的函数**，本质是作用域链的特性。
#### 核心特性
1.  可以访问函数定义时的词法作用域变量，即使函数在外部执行
2.  延长变量的生命周期，让函数内部的变量不会被垃圾回收机制回收
3.  实现变量私有化，避免全局污染

#### 常见使用场景
```javascript
// 1. 实现私有变量
function createCounter() {
  let count = 0; // 私有变量，外部无法直接访问
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
}
const counter = createCounter();
counter.increment(); // 1
counter.getCount(); // 1

// 2. 函数柯里化
function add(a) {
  return function(b) {
    return a + b;
  };
}
const add5 = add(5);
add5(3); // 8
```

#### 注意事项
闭包会导致变量无法被回收，滥用闭包会造成内存泄漏，使用完毕后需手动释放（赋值为null）。

### 5. this 指向
this是函数执行时的上下文对象，其指向**取决于函数的调用方式**，而非定义位置（箭头函数除外）。

| 调用方式 | this指向 |
| :--- | :--- |
| 普通函数全局调用 | 非严格模式：window/globalThis；严格模式：undefined |
| 对象方法调用 | 调用方法的对象本身 |
| 构造函数调用（new） | 新创建的实例对象 |
| call/apply/bind调用 | 手动指定的第一个参数 |
| 箭头函数 | 继承外层词法作用域的this，无法被修改 |

```javascript
// 1. 全局调用
function fn() {
  console.log(this);
}
fn(); // 非严格模式：window

// 2. 对象方法调用
const obj = {
  name: '张三',
  sayName() {
    console.log(this.name);
  }
};
obj.sayName(); // 张三，this指向obj

// 3. 构造函数
function User(name) {
  this.name = name; // this指向新创建的实例
}
const user = new User('李四');
console.log(user.name); // 李四

// 4. call/apply/bind 修改this
function sayHi() {
  console.log(`你好，${this.name}`);
}
const user1 = { name: '王五' };
sayHi.call(user1); // 你好，王五，call立即执行，参数逐个传递
sayHi.apply(user1); // 你好，王五，apply立即执行，参数以数组传递
const bindFn = sayHi.bind(user1); // bind返回新函数，不会立即执行
bindFn(); // 你好，王五

// 5. 箭头函数的this
const obj2 = {
  name: '赵六',
  sayName: () => {
    console.log(this.name); // 继承外层this，此处为window
  }
};
obj2.sayName(); // 空（window无name属性）
```

### 6. 高阶函数
高阶函数是指**接收函数作为参数，或返回函数的函数**，是函数式编程的核心。
- 数组遍历方法：`map`、`filter`、`reduce`、`forEach`等都是高阶函数
- 常见场景：防抖、节流、函数柯里化、回调函数

```javascript
// map：遍历数组，返回处理后的新数组
const arr = [1, 2, 3];
const newArr = arr.map(item => item * 2); // [2,4,6]

// filter：过滤数组，返回符合条件的新数组
const filterArr = arr.filter(item => item > 1); // [2,3]

// reduce：累计计算，返回最终结果
const total = arr.reduce((sum, item) => sum + item, 0); // 6
```

### 7. 立即执行函数（IIFE）
定义后立即执行的函数，用于创建独立的私有作用域，避免全局变量污染，ES6块级作用域和模块化出现后使用频率降低。
```javascript
// 经典写法
(function() {
  const a = 10;
  console.log(a); // 10
})();

// 箭头函数写法
(() => {
  console.log('立即执行');
})();
```

---

## 七、对象与面向对象
JS是基于原型的面向对象语言，ES6新增class语法糖，让面向对象写法更接近传统类语言。

### 1. 对象的创建
对象是键值对的集合，属性名可以是字符串或Symbol，属性值可以是任意类型。
```javascript
// 1. 字面量方式（最常用）
const user = {
  name: '张三',
  age: 20,
  sayHi() {
    console.log(`你好，我是${this.name}`);
  }
};

// 2. new Object() 构造函数
const obj = new Object();
obj.name = '李四';

// 3. Object.create()：基于指定原型创建对象
const proto = { gender: '男' };
const obj2 = Object.create(proto);
console.log(obj2.gender); // 男，继承自proto
```

### 2. 对象属性的访问与操作
```javascript
const user = { name: '张三', age: 20 };

// 1. 属性访问
console.log(user.name); // 点语法，属性名必须是合法标识符
console.log(user['age']); // 方括号语法，支持变量、特殊字符、动态属性名

// 2. 属性添加/修改
user.gender = '男'; // 添加
user.age = 21; // 修改

// 3. 属性删除
delete user.age;
```

### 3. 属性描述符
JS中对象的属性分为**数据属性**和**访问器属性**，通过属性描述符定义属性的行为，可通过`Object.defineProperty()`配置。

#### （1）数据属性描述符
| 配置项 | 说明 | 默认值 |
| :--- | :--- | :--- |
| value | 属性的值 | undefined |
| writable | 是否可修改值 | false |
| enumerable | 是否可被for...in/Object.keys()遍历 | false |
| configurable | 是否可删除属性、修改描述符 | false |

#### （2）访问器属性描述符
通过`get`和`set`方法控制属性的读取和赋值，不能同时设置value和writable。
```javascript
const user = {
  _name: '张三',
  get name() {
    console.log('读取属性');
    return this._name;
  },
  set name(newVal) {
    console.log('修改属性');
    this._name = newVal;
  }
};
user.name; // 触发get，返回'张三'
user.name = '李四'; // 触发set，修改_name
```

### 4. 对象的遍历
| 方法 | 遍历范围 | 说明 |
| :--- | :--- | :--- |
| for...in | 自身+原型链上的可枚举属性 | 可通过hasOwnProperty()过滤原型属性 |
| Object.keys() | 自身可枚举属性 | 返回属性名组成的数组 |
| Object.values() | 自身可枚举属性 | 返回属性值组成的数组 |
| Object.entries() | 自身可枚举属性 | 返回[属性名, 属性值]组成的二维数组 |
| Object.getOwnPropertyNames() | 自身所有属性（包括不可枚举） | 不含Symbol属性 |
| Object.getOwnPropertySymbols() | 自身所有Symbol属性 | 仅Symbol属性 |
| Reflect.ownKeys() | 自身所有属性 | 包括不可枚举、Symbol属性 |

```javascript
const user = { name: '张三', age: 20 };
console.log(Object.keys(user)); // ['name', 'age']
console.log(Object.values(user)); // ['张三', 20]
console.log(Object.entries(user)); // [['name','张三'], ['age',20]]
```

### 5. 原型与原型链
原型是JS实现继承的核心，所有函数都有`prototype`（原型对象），所有对象都有`__proto__`（隐式原型），指向其构造函数的`prototype`。

#### 核心规则
1.  函数的`prototype`是一个对象，默认包含`constructor`属性，指向函数本身
2.  实例对象的`__proto__` === 构造函数的`prototype`
3.  原型对象本身也是对象，也有自己的`__proto__`，最终指向`Object.prototype`
4.  `Object.prototype.__proto__` === `null`，是原型链的终点
5.  当访问对象的属性/方法时，会先在自身查找，找不到则沿`__proto__`向上查找，这条链路就是**原型链**

```javascript
// 构造函数
function User(name) {
  this.name = name;
}
// 原型上添加方法，所有实例共享
User.prototype.sayName = function() {
  console.log(this.name);
};

// 创建实例
const user1 = new User('张三');
user1.sayName(); // 张三

// 原型链验证
console.log(user1.__proto__ === User.prototype); // true
console.log(User.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null); // true
```

### 6. ES6 Class 类
class是ES6新增的语法糖，底层依然基于原型实现，让面向对象写法更清晰、更符合传统面向对象编程习惯。

#### （1）类的基础用法
```javascript
class User {
  // 构造方法，new实例时自动调用
  constructor(name, age) {
    this.name = name; // 实例属性
    this.age = age;
  }

  // 实例方法，挂载到类的prototype上
  sayHi() {
    console.log(`你好，我是${this.name}，今年${this.age}岁`);
  }

  // 静态方法，通过类本身调用，不能通过实例调用
  static sayHello() {
    console.log('Hello');
  }

  // 访问器属性
  get userInfo() {
    return `姓名：${this.name}，年龄：${this.age}`;
  }

  set userInfo(newVal) {
    const [name, age] = newVal.split(',');
    this.name = name;
    this.age = age;
  }
}

// 创建实例
const user1 = new User('张三', 20);
user1.sayHi(); // 你好，我是张三，今年20岁
User.sayHello(); // Hello，静态方法调用
console.log(user1.userInfo); // 触发get
user1.userInfo = '李四,21'; // 触发set
```

#### （2）类的继承
通过`extends`关键字实现继承，`super`关键字调用父类的构造方法和方法。
```javascript
// 父类
class Person {
  constructor(name) {
    this.name = name;
  }
  sayName() {
    console.log(this.name);
  }
}

// 子类继承父类
class Student extends Person {
  constructor(name, grade) {
    super(name); // 必须先调用super，才能使用this
    this.grade = grade;
  }

  // 重写父类方法
  sayName() {
    super.sayName(); // 调用父类方法
    console.log(`我是${this.grade}年级的学生`);
  }
}

const student = new Student('王五', 3);
student.sayName(); // 王五  我是3年级的学生
```

---

## 八、数组
数组是有序的元素集合，JS数组是动态的，可存储任意类型数据，底层基于对象实现，继承自Array.prototype。

### 1. 数组的创建
```javascript
// 1. 字面量方式（最常用）
const arr1 = [1, 2, 3, 4];

// 2. 构造函数
const arr2 = new Array(1, 2, 3); // [1,2,3]
const arr3 = new Array(4); // 长度为4的空数组，[empty × 4]

// 3. ES6新增方法
const arr4 = Array.of(1, 2, 3); // [1,2,3]，解决new Array单个数字的歧义
const arr5 = Array.from('123'); // ['1','2','3']，将类数组/可迭代对象转为数组
```

### 2. 数组常用方法
按功能分为4大类，标注⭐的为高频方法。

#### （1）遍历/迭代方法（不修改原数组）
| 方法 | 说明 |
| :--- | :--- |
| ⭐forEach((item, index, arr)=>{}) | 遍历数组，无返回值，不支持break |
| ⭐map((item, index, arr)=>{}) | 遍历数组，返回处理后的新数组，长度与原数组一致 |
| ⭐filter((item, index, arr)=>{}) | 过滤数组，返回符合条件的元素组成的新数组 |
| ⭐reduce((prev, curr, index, arr)=>{}, init) | 累计器，对数组元素执行累计计算，返回最终结果 |
| find((item, index, arr)=>{}) | 返回第一个符合条件的元素，找不到返回undefined |
| findIndex((item, index, arr)=>{}) | 返回第一个符合条件的元素索引，找不到返回-1 |
| some((item, index, arr)=>{}) | 检测是否有至少一个元素符合条件，返回布尔值 |
| every((item, index, arr)=>{}) | 检测是否所有元素都符合条件，返回布尔值 |

#### （2）修改原数组的方法
| 方法 | 说明 |
| :--- | :--- |
| ⭐push(item1, item2...) | 数组末尾添加元素，返回新长度 |
| ⭐pop() | 删除数组末尾元素，返回被删除的元素 |
| unshift(item1, item2...) | 数组开头添加元素，返回新长度 |
| shift() | 删除数组开头元素，返回被删除的元素 |
| ⭐splice(start, deleteCount, item1...) | 增/删/改数组元素，返回被删除的元素数组 |
| sort((a,b)=>{}) | 数组排序，默认按Unicode编码排序，修改原数组 |
| reverse() | 反转数组，修改原数组 |
| fill(value, start, end) | 用固定值填充数组，修改原数组 |

#### （3）不修改原数组的查询/拼接方法
| 方法 | 说明 |
| :--- | :--- |
| slice(start, end) | 截取数组[start, end)区间的元素，返回新数组 |
| concat(arr1, arr2...) | 拼接多个数组，返回新数组 |
| join(separator) | 将数组所有元素拼接为字符串，返回字符串 |
| includes(value) | 检测数组是否包含指定值，返回布尔值，支持NaN |
| indexOf(value, start) | 返回指定值的第一个索引，找不到返回-1 |
| lastIndexOf(value, start) | 返回指定值的最后一个索引，找不到返回-1 |
| flat(depth) | 扁平化数组，depth为扁平化深度，默认1，Infinity为全量扁平化 |
| flatMap(callback) | 先执行map，再执行flat(1)，返回新数组 |

#### （4）其他方法
- keys()：返回数组索引的迭代器
- values()：返回数组元素的迭代器
- entries()：返回[index, item]的迭代器
- Array.isArray()：检测是否为数组，返回布尔值

### 3. 数组解构与扩展运算符
#### （1）解构赋值（ES6+）
快速从数组中提取值，赋值给变量。
```javascript
const [a, b, c] = [1, 2, 3];
console.log(a); // 1
console.log(b); // 2

// 剩余参数
const [d, ...rest] = [1, 2, 3, 4];
console.log(rest); // [2,3,4]

// 默认值
const [e, f = 10] = [5];
console.log(f); // 10
```

#### （2）扩展运算符`...`（ES6+）
将数组展开为逗号分隔的元素序列。
```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
// 数组合并
const newArr = [...arr1, ...arr2]; // [1,2,3,4,5,6]
// 数组拷贝
const copyArr = [...arr1]; // 浅拷贝
// 函数传参
Math.max(...arr1); // 3，等价于Math.max(1,2,3)
```

---

## 九、异步编程
JS是单线程语言，异步编程是处理耗时操作（网络请求、定时器、文件读写）的核心方案，避免主线程阻塞。

### 1. 事件循环（Event Loop）
JS的异步执行机制，核心分为**调用栈**、**任务队列**、**Web API**。
- 同步任务：直接进入调用栈，主线程立即执行
- 异步任务：先进入Web API，等待触发后进入任务队列，等待调用栈清空后，主线程读取任务队列的任务执行

任务队列分为两类，执行优先级：**微任务 > 宏任务**
- 微任务：Promise.then/catch/finally、async/await、queueMicrotask()、MutationObserver
- 宏任务：setTimeout/setInterval、AJAX、DOM事件、script整体代码、setImmediate(Node.js)

**执行规则**：先执行同步代码，调用栈清空后，执行所有微任务，微任务全部执行完毕后，执行一个宏任务，循环往复。

### 2. 异步编程方案演进
#### （1）回调函数
最基础的异步方案，将后续操作封装为函数，作为参数传入异步任务，缺点是多层嵌套会导致**回调地狱**，代码可读性差、维护困难。
```javascript
// 回调地狱示例
setTimeout(() => {
  console.log('第一层');
  setTimeout(() => {
    console.log('第二层');
    setTimeout(() => {
      console.log('第三层');
    }, 1000);
  }, 1000);
}, 1000);
```

#### （2）Promise（ES6+）
Promise是专门处理异步操作的对象，解决了回调地狱问题，支持链式调用。
##### 核心特性
- 三种状态：`pending`（进行中）、`fulfilled`（已成功）、`rejected`（已失败）
- 状态一旦改变，就无法再修改，只能从pending→fulfilled，或pending→rejected
- 支持链式调用，then方法返回新的Promise，可无限链式调用

##### 基础用法
```javascript
// 创建Promise实例
const promise = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    const isSuccess = true;
    if (isSuccess) {
      resolve('操作成功'); // 成功，状态改为fulfilled，触发then
    } else {
      reject('操作失败'); // 失败，状态改为rejected，触发catch
    }
  }, 1000);
});

// 链式调用
promise
  .then(res => {
    console.log(res); // 操作成功
    return '下一步处理';
  })
  .then(res => {
    console.log(res); // 下一步处理
  })
  .catch(err => {
    console.error(err); // 捕获错误
  })
  .finally(() => {
    console.log('无论成功失败都会执行');
  });
```

##### Promise 静态方法
| 方法 | 说明 |
| :--- | :--- |
| Promise.resolve() | 返回一个fulfilled状态的Promise |
| Promise.reject() | 返回一个rejected状态的Promise |
| Promise.all([p1,p2,p3]) | 所有Promise都成功，返回结果数组；任意一个失败，立即触发catch |
| Promise.allSettled([p1,p2,p3]) | 等待所有Promise完成，返回每个Promise的结果和状态，无论成功失败 |
| Promise.race([p1,p2,p3]) | 返回第一个完成的Promise的结果，无论成功失败 |
| Promise.any([p1,p2,p3]) | 返回第一个成功的Promise的结果；所有都失败，才触发catch |

#### （3）async/await（ES2017）
async/await是Promise的语法糖，让异步代码写起来像同步代码，是目前异步编程的终极方案。

##### 核心规则
1.  `async`关键字用于声明异步函数，函数返回值自动包装为Promise
2.  `await`关键字只能在async函数内使用，后面接Promise对象，会暂停函数执行，等待Promise状态改变后再继续
3.  await会返回Promise成功的结果，错误需通过try/catch捕获

```javascript
// 基础用法
// 封装异步函数
function getData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('请求成功的数据');
    }, 1000);
  });
}

// async/await 调用
async function fetchData() {
  try {
    console.log('开始请求');
    const res = await getData(); // 等待异步完成
    console.log(res); // 请求成功的数据
    console.log('请求结束');
  } catch (err) {
    console.error('请求失败', err);
  }
}

fetchData();
```

---

## 十、模块化
模块化是将代码拆分为独立的文件，每个文件是一个模块，有自己的作用域，按需导入导出，解决了全局变量污染、代码复用、依赖管理等问题。

JS主流模块化规范分为两种：**ES6 Module（ESM）**（浏览器和Node.js通用，官方标准）、**CommonJS**（Node.js原生规范）。

### 1. ES6 Module（ESM）
ES6官方推出的模块化标准，静态编译，在代码编译阶段就确定导入导出关系，而非运行时。

#### 核心规则
1.  每个模块有独立的作用域，模块内的变量/函数/类，外部无法直接访问，必须通过export导出
2.  模块默认开启严格模式，无需手动添加`"use strict"`
3.  浏览器中使用ESM，需给script标签添加`type="module"`属性
4.  导入导出分为**命名导出/导入**和**默认导出/导入**

#### （1）命名导出/导入
一个模块可多次命名导出，导入时名称必须与导出名称一致。
```javascript
// 模块文件：user.js
// 导出方式1：声明时导出
export const name = '张三';
export function sayHi() {
  console.log('你好');
}

// 导出方式2：统一导出
const age = 20;
const getAge = () => age;
export { age, getAge };

// 导入文件：index.js
// 方式1：按需导入
import { name, sayHi, age } from './user.js';
console.log(name); // 张三
sayHi();

// 方式2：整体导入，别名接收
import * as user from './user.js';
console.log(user.age); // 20
user.sayHi();

// 方式3：导入时重命名
import { name as userName } from './user.js';
console.log(userName); // 张三
```

#### （2）默认导出/导入
一个模块只能有一个默认导出，导入时可自定义名称，无需大括号。
```javascript
// 模块文件：utils.js
// 默认导出
export default function sum(a, b) {
  return a + b;
}

// 导入文件：index.js
// 自定义名称导入，无需大括号
import sum from './utils.js';
console.log(sum(1, 2)); // 3
```

#### （3）混合导出与导入
```javascript
// 模块文件：math.js
export const PI = 3.14;
export default function add(a, b) {
  return a + b;
}

// 导入
import add, { PI } from './math.js';
```

#### （4）其他特性
- 动态导入：`import()`函数，支持运行时动态加载模块，返回Promise，可实现按需加载
  ```javascript
  // 按需加载
  button.onclick = async () => {
    const module = await import('./utils.js');
    module.sum(1, 2);
  };
  ```
- 导出重定向：`export * from './user.js'`，将其他模块的导出转发出去

### 2. CommonJS
Node.js原生的模块化规范，动态加载，运行时确定导入导出关系，浏览器不原生支持，需借助webpack等工具打包。
```javascript
// 导出：module.js
const name = '张三';
function sayHi() {
  console.log('你好');
}
module.exports = { name, sayHi };

// 导入：index.js
const { name, sayHi } = require('./module.js');
console.log(name);
sayHi();
```

---

## 十一、异常处理
JS提供了完善的异常处理机制，用于捕获代码运行时的错误，避免程序整体崩溃。

### 1. try/catch/finally 语句
核心异常处理语法，try块内编写可能出错的代码，catch块捕获并处理错误，finally块无论是否出错都会执行。
```javascript
try {
  // 可能出错的代码
  const num = 10;
  num.toUpperCase(); // 数字没有toUpperCase方法，会抛出错误
  console.log('不会执行，出错后直接跳转到catch');
} catch (err) {
  // 捕获错误，处理异常
  console.error('发生错误：', err.message);
} finally {
  // 无论成功失败都会执行，常用于释放资源
  console.log('执行完毕');
}
```

### 2. throw 主动抛出异常
用于手动抛出自定义错误，throw后面可接任意类型的值，通常抛出Error对象。
```javascript
function checkAge(age) {
  if (typeof age !== 'number') {
    // 主动抛出错误
    throw new TypeError('年龄必须是数字');
  }
  if (age < 18) {
    throw new Error('年龄必须满18岁');
  }
  console.log('年龄合法');
}

try {
  checkAge(16);
} catch (err) {
  console.error(err.message); // 年龄必须满18岁
}
```

### 3. 内置错误类型
JS内置了多种错误类型，均继承自Error对象，用于精准区分错误类型：
- `SyntaxError`：语法错误，代码书写不符合JS语法规范
- `ReferenceError`：引用错误，访问了未声明的变量
- `TypeError`：类型错误，值的类型不符合预期（如调用非函数值、修改只读属性）
- `RangeError`：范围错误，数值超出有效范围（如数组长度为负数）
- `URIError`：URI处理错误，调用encodeURI()/decodeURI()时传入无效参数
- `EvalError`：eval()函数执行错误，已基本废弃

---

## 十二、常用内置对象与API
### 1. Math 对象
用于数学计算，提供数学常量和方法，无构造函数，直接调用静态属性和方法。
```javascript
// 常用常量
Math.PI // 圆周率3.1415926...
Math.E // 自然常数e

// 常用方法
Math.abs(-10) // 10，绝对值
Math.floor(3.9) // 3，向下取整
Math.ceil(3.1) // 4，向上取整
Math.round(3.5) // 4，四舍五入
Math.max(1, 5, 3) // 5，最大值
Math.min(1, 5, 3) // 1，最小值
Math.random() // 0-1之间的随机数（不含1）
Math.pow(2, 3) // 8，幂运算
Math.sqrt(9) // 3，平方根
```

### 2. Date 对象
用于处理日期和时间，提供日期的获取、设置、格式化方法。
```javascript
// 创建日期对象
const now = new Date(); // 当前时间
const targetDate = new Date('2025-01-01'); // 指定日期
const customDate = new Date(2025, 0, 1); // 年、月(0-11)、日

// 常用获取方法
now.getFullYear(); // 4位年份
now.getMonth(); // 月份(0-11)
now.getDate(); // 日期(1-31)
now.getDay(); // 星期(0-6，0代表周日)
now.getHours(); // 小时(0-23)
now.getMinutes(); // 分钟(0-59)
now.getSeconds(); // 秒(0-59)
now.getTime(); // 时间戳，1970-01-01至今的毫秒数

// 常用设置方法
now.setFullYear(2026); // 设置年份
now.setMonth(11); // 设置月份
now.setDate(25); // 设置日期
```

### 3. RegExp 正则对象
用于字符串的匹配、查找、替换、验证，支持两种创建方式。
```javascript
// 创建正则对象
// 方式1：字面量（常用）
const reg1 = /^\d{6}$/g; // 匹配6位数字，g为全局修饰符
// 方式2：构造函数
const reg2 = new RegExp('^\\d{6}$', 'g');

// 常用修饰符
// g：全局匹配，i：忽略大小写，m：多行匹配，s：允许.匹配换行符

// 正则实例方法
reg1.test('123456'); // true，检测字符串是否匹配正则
reg1.exec('123456'); // 返回匹配结果数组，匹配不到返回null

// 字符串的正则方法
'abc123def456'.match(/\d+/g); // ['123','456']，匹配所有符合规则的内容
'abc123'.replace(/\d+/, '***'); // 'abc***'，替换匹配内容
'abc123'.search(/\d+/); // 3，返回第一个匹配的索引，匹配不到返回-1
'a,b,c,d'.split(/,/); // ['a','b','c','d']，按正则分割字符串
```

### 4. JSON 对象
用于处理JSON格式数据，提供序列化和反序列化方法。
```javascript
const obj = { name: '张三', age: 20 };

// JSON.stringify()：JS对象转为JSON字符串
const jsonStr = JSON.stringify(obj);
console.log(jsonStr); // '{"name":"张三","age":20}'

// JSON.parse()：JSON字符串转为JS对象
const newObj = JSON.parse(jsonStr);
console.log(newObj); // { name: '张三', age: 20 }
```


