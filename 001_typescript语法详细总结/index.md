# TypeScript 语法详细总结


# TypeScript 语法详细总结
TypeScript（简称TS）是JavaScript的强类型超集，在JS基础上新增**静态类型系统**和ES6+高级特性，编译后输出纯净的JS代码，可运行在任何支持JS的环境中。其核心价值是通过类型检查提前捕获代码错误，提升代码可维护性、可读性和健壮性。

本文从基础到进阶，系统梳理TS核心语法与特性，覆盖日常开发全场景。

---

## 一、核心基础：类型注解与类型推断
### 1. 类型注解
类型注解是TS最基础的语法，用于显式声明变量、函数、对象的类型，语法格式为`标识符: 类型`。
```typescript
// 变量类型注解
let name: string = "TS";
let age: number = 18;
let isDone: boolean = false;

// 函数参数与返回值注解
function add(a: number, b: number): number {
  return a + b;
}
```

### 2. 类型推断
TS具备强大的类型推断能力，当变量有初始赋值、函数有明确返回值时，无需手动写类型注解，TS会自动推导其类型。
```typescript
let num = 123; // 自动推断为 number 类型
let str = "hello"; // 自动推断为 string 类型
const sum = (a: number, b: number) => a + b; // 返回值自动推断为 number
```
> 最佳实践：能稳定推断类型时，无需冗余注解；仅当推断不符合预期、无初始赋值时，手动添加注解。

### 3. 类型断言
用于手动指定值的类型，告诉TS编译器“我比你更清楚这个值的类型”，不会改变运行时行为，仅影响编译阶段。
两种语法格式：
```typescript
// 1. as 语法（推荐，JSX环境下唯一可用）
const value: unknown = "hello world";
const strLength: number = (value as string).length;

// 2. 尖括号语法（非JSX环境可用）
const strLength2: number = (<string>value).length;
```

### 4. 非空断言 !
用于断言某个值**一定不是 null/undefined**，跳过编译器的空值检查。
```typescript
const dom = document.getElementById("app");
dom!.innerText = "hello"; // 断言dom一定存在，不会报“可能为null”的错误
```

---

## 二、基础原始类型
TS完全兼容JS的原始类型，并扩展了专用的类型注解，是TS类型系统的最小单元。

| 类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| `string` | 字符串类型 | `let a: string = "hi"` |
| `number` | 数字类型，支持整数、浮点数、进制数 | `let a: number = 100` |
| `boolean` | 布尔类型，仅 true/false | `let a: boolean = true` |
| `null` | 空值类型，仅 null | `let a: null = null` |
| `undefined` | 未定义类型，仅 undefined | `let a: undefined = undefined` |
| `symbol` | 唯一符号类型 | `let a: symbol = Symbol("key")` |
| `bigint` | 大整数类型 | `let a: bigint = 100n` |
| `void` | 无返回值类型，仅用于函数 | `function fn(): void {}` |
| `never` | 永不存在的值类型，用于永远不会正常返回的场景 | `function fn(): never { throw new Error() }` |
| `any` | 任意类型，关闭TS类型检查，可赋值给任何类型 | `let a: any = 123; a = "str"` |
| `unknown` | 类型安全的any，所有类型都可赋值给unknown，但unknown仅能赋值给unknown/any，必须类型缩小后才能操作 | `let a: unknown = 123; if(typeof a === "number") a++` |

> 关键注意点：
> 1. 开启`strictNullChecks`（严格模式）后，`null`和`undefined`只能赋值给自身、`unknown`、`any`，避免空值运行时错误。
> 2. `never`是所有类型的子类型，可赋值给任何类型；但没有类型（除了never）可以赋值给never。
> 3. 日常开发**禁止滥用any**，会丢失TS的类型保护能力，不确定类型时优先使用`unknown`。

---

## 三、进阶复合类型
基于原始类型组合而成的复杂类型，用于描述更复杂的数据结构。

### 1. 数组类型
用于描述同类型元素的集合，有两种核心写法：
```typescript
// 1. 类型[] 写法（推荐，简洁直观）
let numArr: number[] = [1, 2, 3];
let strArr: string[] = ["a", "b", "c"];

// 2. 数组泛型写法 Array<类型>
let boolArr: Array<boolean> = [true, false];

// 只读数组：禁止修改数组元素
const readonlyArr: readonly number[] = [1, 2, 3];
const readonlyArr2: ReadonlyArray<number> = [1, 2, 3];
```

### 2. 元组 Tuple
固定长度、固定每个位置类型的数组，严格限制元素数量和对应类型，越界访问会报错。
```typescript
// 定义元组：第一个元素是string，第二个是number，长度固定为2
let user: [string, number] = ["张三", 18];

// 可选元素
let userInfo: [string, number?] = ["李四"]; // 第二个元素可选

// 剩余元素
let list: [string, ...number[]] = ["a", 1, 2, 3];

// 只读元组
const readonlyTuple: readonly [string, number] = ["a", 1];
```

### 3. 枚举 Enum
用于定义一组命名的常量，限制变量的取值范围，提升代码可读性。分为3种类型：
```typescript
// 1. 数字枚举（默认，从0开始自增，可手动赋值）
enum Direction {
  Up, // 0
  Down, // 1
  Left = 10, // 手动赋值10
  Right, // 11（基于上一个值自增）
}
console.log(Direction.Up); // 0
console.log(Direction[0]); // 反向映射：Up

// 2. 字符串枚举（无反向映射）
enum Msg {
  Success = "操作成功",
  Fail = "操作失败",
}

// 3. 常量枚举 const enum（编译后会被内联，减少代码体积）
const enum Status {
  Enable = 1,
  Disable = 0,
}
const code = Status.Enable; // 编译后直接变为 const code = 1;
```

### 4. 联合类型 |
表示取值可以是**多个类型中的任意一个**，用`|`分隔。
```typescript
let value: string | number;
value = "hello"; // 合法
value = 123; // 合法
// value = true; // 报错，不支持boolean类型

// 联合类型默认只能访问所有类型的共有属性
function getLength(arg: string | number[]) {
  return arg.length; // 合法，string和数组都有length属性
}
```

### 5. 交叉类型 &
将**多个类型合并为一个类型**，新类型包含所有类型的属性和方法，用`&`分隔。
```typescript
type Person = { name: string; age: number };
type Employee = { id: number; department: string };

// 合并两个类型，Staff 同时拥有 name、age、id、department 属性
type Staff = Person & Employee;

const staff: Staff = {
  name: "张三",
  age: 25,
  id: 1001,
  department: "技术部",
};
```
> 注意：若交叉类型中存在同名属性，且类型不兼容，会合并为`never`类型。

### 6. 字面量类型
将取值限定为**一个或多个固定的字面量**，常与联合类型搭配使用，实现枚举效果。
```typescript
// 字符串字面量类型
type Direction = "up" | "down" | "left" | "right";
const move: Direction = "up"; // 合法，仅能取上述4个值

// 数字字面量类型
type StatusCode = 200 | 400 | 500;
const code: StatusCode = 200;

// 布尔字面量类型
type IsEnable = true | false;
```

---

## 四、接口 Interface
接口用于**描述对象的形状（结构）**，定义对象需要包含的属性、方法及其类型，是TS中约束对象结构的核心方式。

### 1. 基础定义
```typescript
// 定义Person接口，约束对象必须包含name、age属性
interface Person {
  name: string;
  age: number;
}

// 实现接口：必须严格匹配结构，不能缺少/多传未定义的属性
const user: Person = {
  name: "张三",
  age: 18,
};
```

### 2. 进阶属性修饰
```typescript
interface User {
  readonly id: number; // 只读属性：声明时/构造函数中赋值后，不可修改
  name: string;
  age?: number; // 可选属性：该属性可以不存在
  [key: string]: any; // 索引签名：支持任意数量的额外属性，key为string，值为any
}

const user: User = {
  id: 1,
  name: "李四",
  gender: "男", // 索引签名支持的额外属性
};
user.id = 2; // 报错，只读属性不可修改
```

### 3. 接口描述函数
接口不仅可以描述对象，还可以约束函数的参数和返回值类型：
```typescript
interface AddFunc {
  // 语法：(参数名: 参数类型) : 返回值类型
  (a: number, b: number): number;
}

const add: AddFunc = (a, b) => a + b;
```

### 4. 接口继承
接口可以通过`extends`实现继承，复用已有接口的属性定义，支持单继承和多继承。
```typescript
interface Animal {
  name: string;
  eat(): void;
}

// 单继承
interface Dog extends Animal {
  bark(): void;
}

// 多继承
interface Cat extends Animal, Pet {
  miao(): void;
}

const husky: Dog = {
  name: "哈士奇",
  eat() { console.log("吃狗粮"); },
  bark() { console.log("汪汪叫"); },
};
```

### 5. 接口声明合并
同名接口会自动合并属性和方法，这是接口区别于类型别名`type`的核心特性，常用于扩展第三方库的类型。
```typescript
interface User {
  name: string;
}
interface User {
  age: number;
}

// 合并后，User接口同时拥有name和age属性
const user: User = {
  name: "张三",
  age: 18,
};
```

---

## 五、类型别名 Type Alias
类型别名用于给**任意类型起一个新名字**，简化复杂类型的书写，比接口的适用范围更广。

### 1. 基础用法
```typescript
// 给原始类型起别名
type Username = string;

// 给对象类型起别名
type User = {
  readonly id: number;
  name: string;
  age?: number;
};

// 给联合类型起别名
type Status = "pending" | "success" | "fail";

// 给函数类型起别名
type AddFunc = (a: number, b: number) => number;
```

### 2. Interface 与 Type 的核心区别
| 特性 | Interface 接口 | Type 类型别名 |
| :--- | :--- | :--- |
| 核心用途 | 描述对象/类的结构 | 给任意类型定义别名，支持所有类型 |
| 声明合并 | 支持同名接口自动合并 | 不支持同名类型别名，会报错 |
| 继承 | 用`extends`实现继承 | 用`&`交叉类型实现合并 |
| 支持类型 | 仅支持对象、函数、类 | 支持原始类型、联合、交叉、元组、字面量等所有类型 |
| 映射类型 | 不支持 | 支持`in`关键字实现映射类型 |

> 最佳实践：
> - 定义公共API、类的结构、需要扩展的类型，优先用`interface`
> - 定义联合、交叉、元组、复杂工具类型，优先用`type`

---

## 六、函数类型
TS对函数的参数、返回值、this、重载都提供了完整的类型支持。

### 1. 基础函数类型注解
```typescript
// 命名函数：参数注解 + 返回值注解
function add(a: number, b: number): number {
  return a + b;
}

// 函数表达式
const add2: (a: number, b: number) => number = (a, b) => {
  return a + b;
};

// 无返回值函数：void
function log(msg: string): void {
  console.log(msg);
}

// 永远不返回的函数：never
function throwError(msg: string): never {
  throw new Error(msg);
}
```

### 2. 特殊参数处理
```typescript
// 1. 可选参数：? ，必须放在必选参数之后
function greet(name: string, prefix?: string) {
  return prefix ? `${prefix} ${name}` : name;
}

// 2. 默认参数
function add(a: number, b: number = 0) {
  return a + b;
}

// 3. 剩余参数 ...rest
function sum(...nums: number[]): number {
  return nums.reduce((total, num) => total + num, 0);
}
```

### 3. 函数重载
函数重载用于定义**同一个函数的多种入参类型和返回值类型**，先写重载签名，再写实现签名。
```typescript
// 重载签名：定义不同的入参和返回值
function getInfo(id: number): { id: number; name: string };
function getInfo(name: string): { id: number; name: string };

// 实现签名：必须兼容所有重载签名
function getInfo(value: number | string) {
  if (typeof value === "number") {
    return { id: value, name: "用户" + value };
  } else {
    return { id: 1, name: value };
  }
}

// 调用时，会根据入参类型匹配对应的重载签名
getInfo(1001);
getInfo("张三");
```

### 4. this 类型注解
TS支持显式指定函数中`this`的类型，避免this指向错误，`this`必须放在函数参数的第一位。
```typescript
interface User {
  name: string;
  sayHi: () => void;
}

const user: User = {
  name: "张三",
  sayHi() {
    console.log(`Hi, ${this.name}`);
  },
};

// 显式指定this类型
function sayHi(this: User) {
  console.log(`Hi, ${this.name}`);
}
```
> 注意：箭头函数没有自己的this，无法给箭头函数指定this类型。

---

## 七、类 Class
TS在ES6类的基础上，扩展了访问修饰符、抽象类、接口实现等面向对象特性，完善了类的类型系统。

### 1. 基础类定义
```typescript
class Person {
  // 属性类型注解
  name: string;
  age: number;

  // 构造函数参数注解
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  // 方法参数与返回值注解
  sayHi(): void {
    console.log(`Hi, 我是${this.name}，今年${this.age}岁`);
  }
}

const user = new Person("张三", 18);
```

### 2. 访问修饰符
用于控制类的属性和方法的访问权限，共3种：
| 修饰符 | 访问范围 |
| :--- | :--- |
| `public`（默认） | 类内部、子类、外部实例都可访问 |
| `private` | 仅类内部可访问，子类和外部实例都不可访问 |
| `protected` | 类内部、子类可访问，外部实例不可访问 |

```typescript
class User {
  public name: string;
  private password: string;
  protected id: number;

  constructor(name: string, password: string, id: number) {
    this.name = name;
    this.password = password;
    this.id = id;
  }
}

class VipUser extends User {
  showId() {
    console.log(this.id); // 合法，protected子类可访问
    console.log(this.password); // 报错，private子类不可访问
  }
}

const user = new User("张三", "123456", 1001);
console.log(user.name); // 合法
console.log(user.password); // 报错，private外部不可访问
console.log(user.id); // 报错，protected外部不可访问
```

### 3. 核心修饰符
```typescript
class User {
  readonly id: number; // 只读属性：仅声明时/构造函数中可赋值
  static platform: string = "WEB"; // 静态属性：属于类本身，而非实例
  static getPlatform(): string { // 静态方法
    return User.platform;
  }

  constructor(id: number) {
    this.id = id;
  }
}

console.log(User.platform); // 静态成员通过类名访问
console.log(User.getPlatform());
```

### 4. 参数属性
构造函数参数前添加访问修饰符，可快速定义并初始化类的属性，简化代码：
```typescript
// 简化前
class User {
  public name: string;
  private age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

// 简化后（参数属性）
class User {
  constructor(
    public name: string,
    private age: number
  ) {}
}
```

### 5. 抽象类 abstract
抽象类是不能被实例化的基类，用于定义子类必须实现的属性和方法，作为派生类的模板。
```typescript
// 定义抽象类
abstract class Animal {
  abstract name: string; // 抽象属性：子类必须实现
  abstract eat(): void; // 抽象方法：子类必须实现，不能有方法体

  // 普通方法：可直接复用
  sleep(): void {
    console.log("睡觉");
  }
}

// 子类继承抽象类，必须实现所有抽象属性和方法
class Dog extends Animal {
  name: string = "狗";
  eat(): void {
    console.log("吃狗粮");
  }
}

const dog = new Dog();
dog.eat();
dog.sleep();
```

### 6. 类实现接口 implements
类可以通过`implements`实现接口，强制类必须符合接口的结构，一个类可以实现多个接口。
```typescript
interface Fly {
  fly(): void;
}
interface Run {
  run(): void;
}

// 实现多个接口
class Bird implements Fly, Run {
  fly() {
    console.log("飞翔");
  }
  run() {
    console.log("奔跑");
  }
}
```

---

## 八、泛型 Generic
泛型是TS实现**类型复用、类型安全**的核心特性，允许我们在定义函数、接口、类时，不预先指定具体类型，而是在使用时再指定类型，让代码更通用。

### 1. 泛型基础
```typescript
// 泛型函数：T 是类型参数，调用时指定具体类型
function identity<T>(value: T): T {
  return value;
}

// 1. 手动指定类型
const str = identity<string>("hello");
const num = identity<number>(123);

// 2. 类型推断：TS自动根据入参推断类型参数
const bool = identity(true); // 自动推断 T 为 boolean
```

### 2. 泛型约束
通过`extends`关键字限制类型参数的范围，确保类型参数必须包含指定的属性/方法。
```typescript
// 约束：传入的参数必须有 length 属性
interface HasLength {
  length: number;
}

function getLength<T extends HasLength>(value: T): number {
  return value.length;
}

getLength("hello"); // 合法，字符串有length
getLength([1, 2, 3]); // 合法，数组有length
getLength(123); // 报错，数字没有length
```

### 3. 泛型接口与泛型类
```typescript
// 泛型接口
interface ApiResponse<T> {
  code: number;
  msg: string;
  data: T; // 泛型数据，由调用方指定类型
}

// 使用：指定data为用户对象类型
const userRes: ApiResponse<{ name: string; id: number }> = {
  code: 200,
  msg: "success",
  data: { name: "张三", id: 1001 },
};

// 泛型类
class Container<T> {
  value: T;
  constructor(value: T) {
    this.value = value;
  }
  getValue(): T {
    return this.value;
  }
}

const numberContainer = new Container<number>(123);
```

### 4. 泛型默认类型
给泛型参数设置默认类型，调用时若未指定类型，会使用默认类型。
```typescript
interface ApiResponse<T = any> {
  code: number;
  msg: string;
  data: T;
}

// 未指定类型，默认使用 any
const res: ApiResponse = {
  code: 200,
  msg: "success",
  data: {},
};
```

### 5. TS 内置泛型工具类型
TS内置了大量常用的泛型工具类型，基于泛型实现，用于快速转换类型，以下是最常用的工具类型：

| 工具类型 | 作用 | 示例 |
| :--- | :--- | :--- |
| `Partial<T>` | 将T的所有属性变为可选 | `type PartialUser = Partial<{ name: string }>` |
| `Required<T>` | 将T的所有属性变为必选 | `type RequiredUser = Required<{ name?: string }>` |
| `Readonly<T>` | 将T的所有属性变为只读 | `type ReadonlyUser = Readonly<{ name: string }>` |
| `Pick<T, K>` | 从T中挑选指定的K属性，构建新类型 | `type UserName = Pick<User, "name">` |
| `Omit<T, K>` | 从T中排除指定的K属性，构建新类型 | `type UserWithoutAge = Omit<User, "age">` |
| `Exclude<T, U>` | 从T中排除可分配给U的类型，用于联合类型 | `type T = Exclude<"a"|"b", "a"> // "b"` |
| `Extract<T, U>` | 从T中提取可分配给U的类型，用于联合类型 | `type T = Extract<"a"|"b", "a"> // "a"` |
| `ReturnType<T>` | 获取函数T的返回值类型 | `type T = ReturnType<() => string> // string` |
| `Parameters<T>` | 获取函数T的参数类型，组成元组 | `type T = Parameters<(a:number)=>void> // [number]` |
| `Awaited<T>` | 获取Promise resolve的类型，用于异步场景 | `type T = Awaited<Promise<string>> // string` |
| `Record<K, T>` | 构建key为K类型，value为T类型的对象类型 | `type UserMap = Record<number, User>` |

---

## 九、类型守卫与类型缩小
类型缩小是指通过条件判断，将宽泛的类型（如联合类型、unknown）缩小到更具体的类型，让TS编译器精准识别类型，避免报错。类型守卫是实现类型缩小的核心手段。

### 1. 常用类型守卫方式
```typescript
// 1. typeof 守卫：用于原始类型
function print(value: string | number) {
  if (typeof value === "string") {
    console.log(value.length); // 缩小为string类型
  } else {
    console.log(value.toFixed(2)); // 缩小为number类型
  }
}

// 2. instanceof 守卫：用于类实例
class Dog { bark() {} }
class Cat { miao() {} }
function animalAction(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.miao();
  }
}

// 3. in 操作符守卫：判断对象是否包含指定属性
interface User { name: string }
interface Admin { name: string; permission: string[] }
function checkAuth(account: User | Admin) {
  if ("permission" in account) {
    console.log(account.permission); // 缩小为Admin类型
  }
}

// 4. 等值判断守卫：=== / !==
function getValue(value: string | null | undefined) {
  if (value !== null && value !== undefined) {
    console.log(value.length); // 缩小为string类型
  }
}

// 5. 真值判断守卫：if() 排除 falsy 值
function printMsg(msg: string | undefined | null) {
  if (msg) {
    console.log(msg.toUpperCase()); // 缩小为string类型
  }
}
```

### 2. 自定义类型守卫
通过**类型谓词 `parameterName is Type`** 自定义类型守卫，实现复杂的类型判断。
```typescript
// 自定义守卫：判断值是否为string类型
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// 自定义守卫：判断值是否为合法的用户对象
interface User {
  name: string;
  id: number;
}
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "name" in value &&
    "id" in value
  );
}

// 使用
function handleValue(value: unknown) {
  if (isString(value)) {
    console.log(value.length);
  }
  if (isUser(value)) {
    console.log(value.name);
  }
}
```

### 3. 可辨识联合
可辨识联合是TS中最常用的类型缩小技巧，核心是：联合类型的每个成员都包含**一个共同的字面量属性（可辨识标签）**，通过该标签缩小类型。
```typescript
// 定义可辨识联合：每个类型都有共同的type属性
interface Circle {
  type: "circle"; // 可辨识标签
  radius: number;
}
interface Square {
  type: "square"; // 可辨识标签
  sideLength: number;
}
type Shape = Circle | Square;

// 通过type标签缩小类型
function getArea(shape: Shape): number {
  switch (shape.type) {
    case "circle":
      return Math.PI * shape.radius ** 2; // 缩小为Circle类型
    case "square":
      return shape.sideLength ** 2; // 缩小为Square类型
  }
}
```

---

## 十、高级类型特性
### 1. 索引类型
- `keyof T`：索引类型查询，获取类型T的所有属性名，组成联合类型
- `T[K]`：索引访问类型，获取类型T中属性K对应的类型
```typescript
interface User {
  name: string;
  age: number;
  id: number;
}

// keyof 获取所有属性名联合类型
type UserKeys = keyof User; // "name" | "age" | "id"

// 索引访问类型
type UserNameType = User["name"]; // string
type UserIdType = User["id"]; // number
```

### 2. 映射类型
基于已有类型，通过`in`关键字遍历属性，创建新类型，TS内置工具类型大多基于映射类型实现。
```typescript
// 实现只读映射类型
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P]; // 遍历T的所有属性，添加readonly修饰
};

// 实现可选映射类型
type MyPartial<T> = {
  [P in keyof T]?: T[P]; // 遍历T的所有属性，添加?可选修饰
};
```

### 3. 条件类型
基于三元表达式实现类型的条件判断，语法：`T extends U ? X : Y`，若T可分配给U，返回X类型，否则返回Y类型。
```typescript
// 基础条件类型
type IsString<T> = T extends string ? true : false;
type T1 = IsString<string>; // true
type T2 = IsString<number>; // false

// 分布式条件类型：当T是联合类型时，会遍历每个成员执行判断
type MyExclude<T, U> = T extends U ? never : T;
type T3 = MyExclude<"a" | "b" | "c", "a" | "b">; // "c"
```

### 4. 模板字面量类型
基于字符串字面量，通过`${}`嵌入类型，实现字符串类型的动态拼接，TS还提供了内置的字符串操作类型。
```typescript
// 基础模板字面量
type Direction = "up" | "down";
type DirectionEvent = `on${Capitalize<Direction>}`; // "onUp" | "onDown"

// 内置字符串工具类型
type S1 = Uppercase<"hello">; // "HELLO"
type S2 = Lowercase<"HELLO">; // "hello"
type S3 = Capitalize<"hello">; // "Hello"
type S4 = Uncapitalize<"Hello">; // "hello"
```

### 5. 常用操作符
```typescript
// 1. 可选链操作符 ?.：安全访问嵌套属性，避免Cannot read properties of undefined
const userName = user?.info?.name;

// 2. 空值合并操作符 ??：仅当左侧为null/undefined时，返回右侧值
const pageSize = query.pageSize ?? 10;

// 3. satisfies 操作符：验证值是否符合类型，同时保留值的字面量类型
type Config = Record<string, string | number>;
const config = {
  host: "localhost",
  port: 3000,
} satisfies Config;
config.port.toFixed(2); // 保留了port的number类型，不会被拓宽为string|number
```

---

## 十一、模块与命名空间
### 1. ES模块
TS完全兼容ES6的`import/export`模块规范，支持类型的单独导入导出。
```typescript
// 导出类型和值
export type User = { name: string };
export const user = { name: "张三" };

// 导入类型和值
import { User, user } from "./user";

// 仅导入类型（编译后会被移除，减少体积，避免循环引用）
import type { User } from "./user";
```

### 2. 模块声明
用于给无类型的第三方JS库补充类型声明，通常写在`.d.ts`声明文件中。
```typescript
// 给第三方JS模块声明类型
declare module "untyped-js-lib" {
  export function add(a: number, b: number): number;
  export const version: string;
}

// 全局类型声明
declare global {
  interface Window {
    $: JQuery; // 给window扩展$属性
  }
}
```

### 3. 命名空间 namespace
TS早期的模块化方案，用于在全局作用域内隔离代码，避免命名冲突，现在推荐优先使用ES模块，仅在特殊场景使用。
```typescript
namespace Utils {
  export function add(a: number, b: number) {
    return a + b;
  }
  export const version = "1.0.0";
}

// 访问命名空间内的成员
console.log(Utils.add(1, 2));
console.log(Utils.version);
```

---

## 十二、装饰器 Decorator
装饰器是一种特殊类型的声明，用于扩展类、方法、属性、参数的功能，是AOP（面向切面编程）的核心实现方式。TS支持两种装饰器模式：**实验性装饰器**和**ES标准装饰器**。

### 1. 开启装饰器支持
在`tsconfig.json`中开启配置：
```json
{
  "compilerOptions": {
    // 实验性装饰器
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    // ES标准装饰器（TS 5.0+ 支持）
    "experimentalDecorators": false,
    "target": "ES2022"
  }
}
```

### 2. 装饰器分类（实验性）
```typescript
// 1. 类装饰器：作用于类，参数是类的构造函数
function ClassDecorator(target: Function) {
  console.log("类装饰器", target);
}

// 2. 方法装饰器：作用于类的方法
function MethodDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  console.log("方法装饰器", propertyKey);
}

// 3. 属性装饰器：作用于类的属性
function PropertyDecorator(target: any, propertyKey: string) {
  console.log("属性装饰器", propertyKey);
}

// 4. 参数装饰器：作用于方法的参数
function ParamDecorator(target: any, propertyKey: string, parameterIndex: number) {
  console.log("参数装饰器", parameterIndex);
}

// 使用装饰器
@ClassDecorator
class User {
  @PropertyDecorator
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  @MethodDecorator
  sayHi(@ParamDecorator msg: string) {
    console.log(msg);
  }
}
```

### 3. 装饰器工厂
返回装饰器的函数，支持传入自定义参数，灵活控制装饰器的行为。
```typescript
// 装饰器工厂
function Log(msg: string) {
  return function (target: Function) {
    console.log(`${msg}：${target.name}`);
  };
}

@Log("用户类被创建")
class User {}
```

---

## 十三、核心配置与严格模式
TS的行为由`tsconfig.json`配置文件控制，其中**严格模式**是保证类型安全的核心，以下是核心配置项：
```json
{
  "compilerOptions": {
    // 编译目标JS版本
    "target": "ES2020",
    // 模块系统
    "module": "ESNext",
    // 模块解析策略
    "moduleResolution": "NodeNext",
    // 开启严格模式（包含以下所有严格检查子项）
    "strict": true,
    // 严格空值检查，禁止null/undefined赋值给非空类型
    "strictNullChecks": true,
    // 禁止隐式any类型
    "noImplicitAny": true,
    // 严格检查函数的this类型
    "strictFunctionTypes": true,
    // 严格检查类的属性初始化
    "strictPropertyInitialization": true,
    // 兼容ES模块和CommonJS
    "esModuleInterop": true,
    // 跳过库文件类型检查
    "skipLibCheck": true,
    // 生成类型声明文件
    "declaration": true
  },
  // 编译包含的文件
  "include": ["src/**/*"],
  // 编译排除的文件
  "exclude": ["node_modules"]
}
```
> 最佳实践：新项目必须开启`strict: true`严格模式，最大化TS的类型检查能力，提前规避运行时错误。


