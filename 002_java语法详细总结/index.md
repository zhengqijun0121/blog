# Java 语法详细总结


# Java语法详细总结
Java是一门**静态强类型、面向对象、跨平台**的编程语言，核心特性是“一次编写，到处运行（Write Once, Run Anywhere）”，依托JVM实现跨平台能力。以下从基础到进阶，系统梳理Java核心语法体系。

## 一、Java程序基础结构
一个标准的Java程序，核心结构如下，这是所有Java程序的入口规范：
```java
// 包声明，必须放在代码第一行（可选）
package com.example.demo;

// 导入外部类（可选）
import java.util.Arrays;

// 类定义，public类名必须与文件名完全一致
public class HelloWorld {
    // 程序入口main方法，固定格式，JVM自动识别执行
    public static void main(String[] args) {
        // 执行语句，以分号结尾
        System.out.println("Hello Java!");
    }
}
```
### 核心规则
1. 一个Java源文件可包含多个类，但**最多只能有一个public类**，且public类名必须与文件名完全一致。
2. `main`方法是程序唯一入口，固定格式为`public static void main(String[] args)`，不可修改。
3. 所有代码必须写在类中，Java无全局变量/全局函数，是纯面向对象语言。
4. 执行语句必须以分号`;`结尾，大小写敏感。
5. 注释规范：
   - 单行注释：`// 注释内容`
   - 多行注释：`/* 注释内容 */`
   - 文档注释：`/** 注释内容 */`，可通过javadoc生成API文档。

## 二、标识符与关键字
### 1. 标识符命名规则
标识符是给类、方法、变量、常量等起的名字，必须遵守以下规则：
- 由**字母、数字、下划线_、美元符$**组成，不能以数字开头。
- 不能是Java关键字、保留字，不能包含空格、特殊符号。
- 严格区分大小写。

### 2. 命名规范（行业通用）
| 类型 | 规范 | 示例 |
|------|------|------|
| 类/接口/枚举 | 大驼峰命名法，每个单词首字母大写 | HelloWorld、UserService |
| 方法/变量 | 小驼峰命名法，首单词小写，后续单词首字母大写 | getUserName、userAge |
| 常量 | 全大写，单词间用下划线分隔 | MAX_PAGE_SIZE、PI |
| 包名 | 全小写，域名倒写，单词间用点分隔 | com.example.demo |

### 3. 核心关键字
Java关键字是被语言赋予特殊含义的保留字，不能作为标识符，核心分类如下：
| 分类 | 关键字 |
|------|--------|
| 访问修饰符 | public、protected、private、default（包私有，不写默认） |
| 类/方法/变量修饰符 | class、interface、enum、extends、implements、static、final、abstract、synchronized、volatile、native、transient、strictfp |
| 流程控制 | if、else、switch、case、for、while、do、break、continue、return、default |
| 数据类型 | byte、short、int、long、float、double、char、boolean、void |
| 异常处理 | try、catch、finally、throw、throws |
| 其他核心 | new、this、super、instanceof、import、package、assert |

> 注：`true`、`false`、`null`是字面量，不是关键字，但同样不能作为标识符。

## 三、数据类型
Java数据类型分为**基本数据类型**和**引用数据类型**两大类，核心区别是：基本类型直接存储值，引用类型存储堆内存中对象的地址。

### 1. 基本数据类型（8种）
Java提供8种内置基本类型，无需new创建，有固定的内存大小和默认值，分为4大类：

#### （1）整数型（无小数）
| 类型 | 占用字节 | 取值范围 | 默认值 | 说明 |
|------|----------|----------|--------|------|
| byte | 1字节 | -128 ~ 127 | 0 | 最小整数类型，常用于文件/网络流传输 |
| short | 2字节 | -32768 ~ 32767 | 0 | 短整型，使用场景较少 |
| int | 4字节 | -2³¹ ~ 2³¹-1（约±21亿） | 0 | 整数默认类型，最常用 |
| long | 8字节 | -2⁶³ ~ 2⁶³-1 | 0L | 长整型，声明时必须加`L/l`后缀，例：`long num = 100L;` |

#### （2）浮点型（带小数）
| 类型 | 占用字节 | 精度 | 默认值 | 说明 |
|------|----------|------|--------|------|
| float | 4字节 | 单精度，6~7位有效数字 | 0.0f | 单精度浮点，声明时必须加`F/f`后缀，例：`float f = 3.14f;` |
| double | 8字节 | 双精度，15~16位有效数字 | 0.0 | 浮点默认类型，精度更高，日常开发优先使用 |

> 注：浮点型存在精度丢失问题，金融场景优先使用`BigDecimal`。

#### （3）字符型
| 类型 | 占用字节 | 取值范围 | 默认值 | 说明 |
|------|----------|----------|--------|------|
| char | 2字节 | 0 ~ 65535 | '\u0000' | 无符号类型，用单引号包裹，可存储Unicode字符、中文，支持转义字符（`\n`换行、`\t`制表、`\\`反斜杠等） |

#### （4）布尔型
| 类型 | 占用空间 | 取值 | 默认值 | 说明 |
|------|----------|------|--------|------|
| boolean | JVM底层用int实现，数组用byte数组 | true/false | false | 仅能表示真假，不能与整数类型互转 |

### 2. 引用数据类型
引用类型指向堆内存中的对象，默认值为`null`，核心包括：
- 类（Class）：如String、Object、自定义类
- 接口（Interface）
- 数组（Array）
- 枚举（Enum）
- 注解（Annotation）

### 3. 类型转换
#### （1）自动类型转换（隐式）
小范围类型自动转为大范围类型，无精度丢失，转换顺序：
`byte → short → int → long → float → double`
`char → int`
> 注意：byte、short、char之间运算时，会自动提升为int类型。

#### （2）强制类型转换（显式）
大范围类型转为小范围类型，可能出现精度丢失或数据溢出，需手动声明，格式：
`(目标类型) 变量/值`
示例：
```java
double pi = 3.1415;
int intPi = (int) pi; // 结果3，丢失小数精度
```

## 四、运算符
Java运算符按功能分为7大类，优先级从高到低为：**单目 > 算术 > 移位 > 比较 > 位运算 > 逻辑 > 三元 > 赋值**，括号优先级最高。

### 1. 算术运算符
| 运算符 | 说明 | 注意事项 |
|--------|------|----------|
| + | 加法、字符串拼接 | 任意类型与字符串+，都会转为字符串拼接 |
| - | 减法 | - |
| * | 乘法 | - |
| / | 除法 | 整数相除结果为整数，例：5/2=2；5.0/2=2.5 |
| % | 取余（取模） | 结果符号与被除数一致，例：5%2=1，-5%2=-1 |
| ++ | 自增1 | 前置++：先自增，再运算；后置++：先运算，再自增 |
| -- | 自减1 | 与++规则一致 |

### 2. 赋值运算符
| 运算符 | 说明 | 示例 |
|--------|------|------|
| = | 基本赋值 | int a = 10; |
| +=、-=、*=、/=、%= | 复合赋值 | a += 5; 等价于 a = a + 5; 自动强转类型 |
| &=、|=、^=、<<=、>>=、>>>= | 位运算复合赋值 | 同上，先运算再赋值 |

> 注：复合赋值运算符会自动处理类型强转，例：`byte b=10; b+=5;`不会报错，而`b=b+5`会因int类型提升编译报错。

### 3. 比较运算符
所有比较运算符的结果都是boolean类型（true/false）。
| 运算符 | 说明 |
|--------|------|
| == | 等于：基本类型比较值，引用类型比较内存地址 |
| != | 不等于 |
| >、< | 大于、小于 |
| >=、<= | 大于等于、小于等于 |
| instanceof | 判断对象是否是指定类/接口的实例，例：`"abc" instanceof String` |

### 4. 逻辑运算符
用于多个boolean表达式的逻辑运算，核心区别是**短路与/或**会中断执行，效率更高。
| 运算符 | 说明 | 规则 |
|--------|------|------|
| && | 短路与 | 左边为false时，右边代码不执行，全真才为真 |
| || | 短路或 | 左边为true时，右边代码不执行，全假才为假 |
| ! | 逻辑非 | 取反，!true=false，!false=true |
| & | 逻辑与 | 无论左边结果，右边始终执行，全真才为真 |
| | | 逻辑或 | 无论左边结果，右边始终执行，全假才为假 |
| ^ | 异或 | 两边结果不同为true，相同为false |

### 5. 位运算符
直接对二进制位进行运算，执行效率极高。
| 运算符 | 说明 | 示例 |
|--------|------|------|
| & | 按位与 | 两位都为1，结果为1 |
| | | 按位或 | 两位有一个为1，结果为1 |
| ^ | 按位异或 | 两位不同为1，相同为0 |
| ~ | 按位取反 | 0变1，1变0 |
| << | 左移 | 左移n位，等价于 *2ⁿ，例：2<<3=16 |
| >> | 带符号右移 | 右移n位，等价于 /2ⁿ，负数用1补高位 |
| >>> | 无符号右移 | 无论正负，都用0补高位，仅对正数有效 |

### 6. 三元运算符
Java唯一的三目运算符，可简化简单的if-else逻辑，格式：
`条件表达式 ? 表达式1 : 表达式2`
- 条件为true，执行表达式1，返回其结果；
- 条件为false，执行表达式2，返回其结果；
- 必须有返回值，可嵌套使用。
示例：`int max = a > b ? a : b;`

## 五、流程控制语句
Java流程控制分为**顺序结构、分支结构、循环结构**三大类，是程序执行的核心逻辑。

### 1. 分支结构
#### （1）if-else 条件分支
```java
// 单分支
if (条件表达式) {
    // 条件为true执行
}

// 双分支
if (条件表达式) {
    // 条件为true执行
} else {
    // 条件为false执行
}

// 多分支
if (条件1) {
    // 条件1为true执行
} else if (条件2) {
    // 条件2为true执行
} else {
    // 所有条件都不满足执行
}
```
> 注意：else永远匹配离它最近的、未匹配的if。

#### （2）switch 选择分支
用于固定值的多分支匹配，Java 7+支持String类型，Java 14+支持switch表达式。
```java
// 传统switch
switch (表达式) {
    case 常量1:
        // 匹配常量1执行
        break; // 可选，无break会发生case穿透
    case 常量2:
        // 匹配常量2执行
        break;
    default:
        // 所有case都不匹配执行，可选，位置任意
        break;
}

// Java14+ switch表达式（有返回值，无需break）
int result = switch (day) {
    case 1,2,3,4,5 -> 1; // 工作日
    case 6,7 -> 0; // 休息日
    default -> -1;
};
```
**核心规则**：
- 表达式支持的类型：byte、short、int、char、枚举、String（Java7+）；
- case后的常量不能重复，不能是变量；
- break用于终止switch，无break会发生case穿透，后续case无论是否匹配都会执行。

### 2. 循环结构
用于重复执行某段代码，核心分为4种：

#### （1）for循环
适合循环次数已知的场景，格式：
```java
for (初始化表达式; 条件表达式; 更新表达式) {
    // 循环体
}
```
- 初始化表达式：仅在循环开始时执行1次，用于声明循环变量；
- 条件表达式：每次循环前判断，为true执行循环体，为false终止循环；
- 更新表达式：每次循环体执行完毕后执行，用于更新循环变量。

#### （2）增强for循环（foreach）
Java5+引入，用于遍历数组/集合，无索引，简化遍历代码，格式：
```java
for (元素类型 变量名 : 数组/集合) {
    // 循环体，变量名代表当前遍历的元素
}
```
> 注意：仅能遍历读取元素，不能修改元素值，无法获取索引；不能遍历过程中增删集合元素。

#### （3）while循环
适合循环次数未知的场景，先判断后执行，循环体可能一次都不执行，格式：
```java
while (条件表达式) {
    // 循环体
}
```

#### （4）do-while循环
先执行后判断，循环体**至少执行一次**，格式：
```java
do {
    // 循环体
} while (条件表达式); // 结尾必须有分号
```

### 3. 跳转语句
| 语句 | 作用 |
|------|------|
| break | 跳出当前循环/switch，结束整个循环；支持带标签的break，跳出指定外层循环 |
| continue | 跳过本次循环的剩余代码，直接进入下一次循环；支持带标签的continue |
| return | 结束当前方法，返回指定值（若方法有返回值），终止方法内所有后续代码 |

## 六、面向对象核心（OOP）
Java是纯面向对象语言，面向对象的三大核心特性：**封装、继承、多态**。

### 1. 类与对象
- 类：对象的抽象模板，定义了对象的属性（成员变量）和行为（成员方法）；
- 对象：类的具体实例，是类的具象化，通过`new`关键字创建。

#### （1）类的定义
```java
// 类定义
public class User {
    // 成员变量（属性）
    private String username;
    private int age;

    // 无参构造方法
    public User() {}

    // 有参构造方法
    public User(String username, int age) {
        this.username = username;
        this.age = age;
    }

    // 成员方法（行为）
    public void showInfo() {
        System.out.println("用户名：" + username + "，年龄：" + age);
    }

    // get/set方法
    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

#### （2）对象的创建与使用
```java
// 创建对象
User user = new User("张三", 20);
// 调用成员方法
user.showInfo();
// 调用set/get方法
user.setUsername("李四");
System.out.println(user.getUsername());
```

#### （3）成员变量 vs 局部变量
| 维度 | 成员变量 | 局部变量 |
|------|----------|----------|
| 定义位置 | 类中，方法/代码块外 | 方法、代码块、形参中 |
| 内存位置 | 堆内存（随对象存储） | 栈内存 |
| 生命周期 | 随对象创建而存在，随对象回收而销毁 | 随方法调用创建，方法执行完毕销毁 |
| 默认值 | 有对应数据类型的默认值 | 无默认值，必须先赋值再使用 |
| 修饰符 | 可使用访问修饰符、static、final等 | 不能使用访问修饰符、static，可使用final |

### 2. 封装
**核心思想**：隐藏对象的属性和实现细节，仅对外暴露公共的访问方式，提高代码安全性和复用性。
- 实现方式：用`private`修饰成员变量，对外提供`public`的`get/set`方法，可在方法中添加数据校验逻辑。
- 访问修饰符（权限从大到小）：

| 修饰符 | 同类 | 同包 | 不同包子类 | 不同包非子类 |
|--------|------|------|------------|--------------|
| public | ✅ | ✅ | ✅ | ✅ |
| protected | ✅ | ✅ | ✅ | ❌ |
| default（包私有，不写） | ✅ | ✅ | ❌ | ❌ |
| private | ✅ | ❌ | ❌ | ❌ |

### 3. 继承
**核心思想**：子类继承父类的非私有属性和方法，实现代码复用，Java是**单继承**，一个类只能有一个直接父类，支持多层继承，所有类默认直接/间接继承`Object`类。

#### （1）继承的定义
```java
// 父类
public class Animal {
    protected String name;
    public void eat() {
        System.out.println("动物吃东西");
    }
}

// 子类，继承Animal
public class Cat extends Animal {
    // 子类特有方法
    public void catchMouse() {
        System.out.println("猫抓老鼠");
    }

    // 重写父类方法
    @Override
    public void eat() {
        System.out.println("猫吃鱼");
    }
}
```

#### （2）super关键字
代表父类对象的引用，核心用法：
1. `super.成员变量`：访问父类的成员变量；
2. `super.成员方法`：调用父类的成员方法；
3. `super(参数)`：调用父类的构造方法，**必须放在子类构造方法的第一行**。
> 注意：子类构造方法默认会自动调用父类的无参构造`super()`，若父类没有无参构造，必须手动调用父类的有参构造。

#### （3）方法重写（Override）
子类对父类的方法进行重新实现，是多态的前提，**重写规则**：
- 方法名、参数列表必须与父类完全一致；
- 返回值类型：子类返回值类型必须小于等于父类（协变返回类型）；
- 访问修饰符：子类权限必须大于等于父类；
- 不能抛出比父类更大的编译时异常；
- 父类的`private`、`static`、`final`方法不能被重写；
- `@Override`注解用于校验重写语法是否正确。

#### （4）重写 vs 重载
| 维度 | 重写（Override） | 重载（Overload） |
|------|-------------------|-------------------|
| 作用范围 | 父子类之间 | 同一个类中 |
| 方法名 | 必须相同 | 必须相同 |
| 参数列表 | 必须完全相同 | 必须不同（个数、类型、顺序） |
| 返回值 | 必须一致（协变） | 无要求 |
| 修饰符 | 权限不能更小，不能用static/final | 无要求 |
| 多态类型 | 运行时多态 | 编译时多态 |

### 4. 多态
**核心定义**：同一个行为，不同的对象具有不同的表现形式，多态的前提：
1. 继承/实现关系；
2. 方法重写；
3. 父类引用指向子类对象。

#### （1）多态的使用
```java
// 父类引用指向子类对象，向上转型
Animal animal = new Cat();
// 编译看左边，运行看右边：编译时只能调用Animal类的方法，运行时执行Cat重写的方法
animal.eat(); // 输出：猫吃鱼
// 无法调用子类特有方法，编译报错
// animal.catchMouse();

// 向下转型，强制类型转换，可调用子类特有方法
if (animal instanceof Cat) { // 先判断类型，避免类型转换异常
    Cat cat = (Cat) animal;
    cat.catchMouse(); // 输出：猫抓老鼠
}
```

#### （2）向上转型 vs 向下转型
- **向上转型**：子类对象→父类引用，自动转换，会丢失子类特有方法，实现通用化处理；
- **向下转型**：父类引用→子类对象，强制转换，可恢复子类特有方法，必须用`instanceof`判断类型，避免`ClassCastException`。

### 5. 核心关键字
#### （1）static关键字
`static`修饰的成员属于**类本身**，不属于某个对象，被所有对象共享，在类加载时初始化，优先于对象存在。
- 核心用法：
  1. **静态变量（类变量）**：通过`类名.变量名`访问，无需创建对象，常用于全局常量；
  2. **静态方法（类方法）**：通过`类名.方法名`调用，静态方法中只能访问静态成员，不能使用`this/super`，不能访问非静态成员；
  3. **静态代码块**：`static { 代码 }`，类加载时仅执行一次，用于初始化静态资源；
  4. **静态内部类**：修饰内部类，属于外部类本身。
> 注意：static不能修饰局部变量，不能修饰外部类。

#### （2）final关键字
`final`代表“最终的、不可修改的”，核心用法：
1. **修饰类**：该类不能被继承，例：String、Math、Integer等包装类；
2. **修饰方法**：该方法不能被子类重写；
3. **修饰变量**：变为常量，赋值后不可修改；
   - 修饰基本类型：值不可修改；
   - 修饰引用类型：地址不可修改，对象的内容可修改；
   - 成员常量：必须在定义时、代码块、构造方法中赋值，三选一；
   - 静态常量：`static final`，必须在定义时或静态代码块中赋值，全大写命名。

### 6. 抽象类与接口
#### （1）抽象类
用`abstract`修饰的类，用于抽取子类的通用特性，**不能实例化**，只能被继承。
- 核心规则：
  1. 包含抽象方法的类，必须是抽象类；抽象类可以没有抽象方法；
  2. 抽象方法：`abstract`修饰，无方法体，子类必须重写所有抽象方法，否则子类也必须是抽象类；
  3. 抽象类可包含普通方法、成员变量、构造方法、代码块；
  4. `abstract`不能与`final`、`private`、`static`共用。
```java
// 抽象类
public abstract class Shape {
    // 抽象方法，无方法体
    public abstract double getArea();
    // 普通方法
    public void show() {
        System.out.println("这是一个图形");
    }
}

// 子类继承抽象类，必须重写所有抽象方法
public class Circle extends Shape {
    private double radius;
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
}
```

#### （2）接口
用`interface`修饰，是行为的规范/契约，定义了类必须具备的能力，Java8+后功能大幅扩展。
- 核心规则：
  1. 接口不能实例化，只能被类`implements`实现，一个类可实现多个接口，解决单继承限制；
  2. 成员变量：默认是`public static final`，只能是常量，必须初始化；
  3. 抽象方法：默认是`public abstract`，实现类必须重写所有抽象方法，否则为抽象类；
  4. 默认方法（Java8+）：`default`修饰，有方法体，实现类可重写可不重写，解决接口新增方法的兼容性问题；
  5. 静态方法（Java8+）：`static`修饰，有方法体，只能通过`接口名.方法名`调用，实现类不能继承；
  6. 私有方法（Java9+）：`private`修饰，有方法体，用于接口内方法的代码复用；
  7. 接口可继承多个接口，格式：`interface A extends B,C {}`。
```java
// 接口定义
public interface Flyable {
    // 常量，默认public static final
    int MAX_SPEED = 1000;
    // 抽象方法，默认public abstract
    void fly();
    // 默认方法
    default void show() {
        System.out.println("可以飞行");
    }
    // 静态方法
    static void staticMethod() {
        System.out.println("接口静态方法");
    }
}

// 类实现接口
public class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("鸟用翅膀飞行");
    }
}
```

#### （3）抽象类 vs 接口
| 维度 | 抽象类 | 接口 |
|------|--------|------|
| 继承/实现 | 单继承，一个类只能extends一个抽象类 | 多实现，一个类可implements多个接口 |
| 成员变量 | 可普通变量，可常量 | 只能是public static final常量 |
| 方法 | 可抽象方法、普通方法、静态方法、final方法 | Java7：仅抽象方法；Java8+：默认、静态方法；Java9+：私有方法 |
| 构造方法 | 有构造方法 | 无构造方法 |
| 设计目的 | 代码复用，抽取子类通用的属性和行为 | 行为规范，定义类的能力，解耦 |

### 7. 枚举（enum）
Java5+引入，用于定义固定的常量集合，例：季节、星期、订单状态等，本质是继承`Enum`的final类。
```java
// 枚举定义
public enum Season {
    // 枚举常量，默认public static final，是枚举类的实例
    SPRING("春天", "春暖花开"),
    SUMMER("夏天", "烈日炎炎"),
    AUTUMN("秋天", "秋高气爽"),
    WINTER("冬天", "冰天雪地");

    // 成员变量
    private final String name;
    private final String desc;

    // 构造方法，必须private，默认private
    Season(String name, String desc) {
        this.name = name;
        this.desc = desc;
    }

    // get方法
    public String getName() {
        return name;
    }
}

// 枚举使用
public class Test {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        System.out.println(spring.getName());
        // 遍历枚举
        for (Season season : Season.values()) {
            System.out.println(season);
        }
    }
}
```
- 核心方法：
  - `values()`：返回所有枚举常量的数组；
  - `valueOf(String name)`：返回指定名称的枚举常量；
  - `ordinal()`：返回枚举常量的序号，从0开始。

## 七、数组与字符串
### 1. 数组
数组是**相同数据类型元素的有序集合**，长度固定，一旦创建不可修改，可存储基本类型和引用类型。

#### （1）数组的声明与初始化
```java
// 声明，推荐第一种
int[] arr1;
int arr2[];

// 静态初始化：指定元素，系统计算长度
int[] arr3 = {1,2,3,4,5};
int[] arr4 = new int[]{1,2,3,4,5};

// 动态初始化：指定长度，系统赋默认值
int[] arr5 = new int[5]; // 长度5，默认值0
String[] arr6 = new String[3]; // 长度3，默认值null
```

#### （2）数组的访问与遍历
```java
// 访问：数组名[索引]，索引从0开始，最大索引=长度-1
int[] arr = {1,2,3};
System.out.println(arr[0]); // 输出1
arr[1] = 10; // 修改元素

// 长度：数组名.length属性
System.out.println(arr.length); // 输出3

// 遍历1：普通for循环
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}

// 遍历2：增强for循环
for (int num : arr) {
    System.out.println(num);
}
```

#### （3）二维数组
数组的数组，每个元素是一个一维数组，长度可不一致。
```java
// 动态初始化
int[][] arr1 = new int[3][2]; // 3行2列
// 静态初始化
int[][] arr2 = {{1,2}, {3,4,5}, {6}};
// 访问
System.out.println(arr2[0][1]); // 输出2
```

#### （4）数组工具类Arrays
`java.util.Arrays`提供了数组操作的常用方法：
- `sort()`：数组排序；
- `binarySearch()`：二分查找元素，返回索引；
- `copyOf()`：复制数组，生成新数组；
- `fill()`：用指定值填充数组；
- `toString()`：将数组转为字符串，方便打印；
- `equals()`：比较两个数组的元素是否完全一致。

### 2. 字符串
Java字符串核心分为3类：`String`、`StringBuffer`、`StringBuilder`，位于`java.lang`包。

#### （1）String类
不可变字符序列，一旦创建，内容不可修改，任何修改都会创建新的String对象。
```java
// 直接赋值，存放在字符串常量池，复用对象
String s1 = "abc";
// new创建，堆内存创建新对象
String s2 = new String("abc");
```
**核心常用方法**：
| 方法 | 作用 |
|------|------|
| `length()` | 返回字符串长度 |
| `charAt(int index)` | 返回指定索引的字符 |
| `equals(Object obj)` | 比较字符串内容是否相等，重写了Object的方法 |
| `equalsIgnoreCase(String str)` | 忽略大小写比较内容 |
| `contains(CharSequence s)` | 判断是否包含指定子串 |
| `indexOf(String str)` | 返回子串第一次出现的索引，无则返回-1 |
| `substring(int beginIndex)` | 截取从beginIndex到结尾的子串 |
| `substring(int beginIndex, int endIndex)` | 截取[beginIndex, endIndex)范围的子串，左闭右开 |
| `replace(char oldChar, char newChar)` | 替换所有指定字符 |
| `split(String regex)` | 按正则表达式分割为字符串数组 |
| `trim()` | 去除首尾空格 |
| `toUpperCase()/toLowerCase()` | 转为全大写/全小写 |
| `toCharArray()` | 转为字符数组 |
| `valueOf(Xxx x)` | 静态方法，将其他类型转为字符串 |

> 注意：String重写了`equals()`和`hashCode()`，`==`比较的是内存地址，`equals()`比较的是内容。

#### （2）StringBuffer & StringBuilder
可变字符序列，内容修改不会创建新对象，直接修改原对象，适合频繁修改字符串的场景。

| 维度 | StringBuffer | StringBuilder |
|------|--------------|---------------|
| 线程安全 | 线程安全，方法加了`synchronized` | 线程不安全，无同步锁 |
| 执行效率 | 低 | 高 |
| 版本 | JDK1.0 | JDK1.5 |

**核心常用方法**：
- `append(Xxx x)`：追加内容，支持所有类型，链式调用；
- `insert(int offset, Xxx x)`：在指定位置插入内容；
- `delete(int start, int end)`：删除指定范围的字符；
- `replace(int start, int end, String str)`：替换指定范围的内容；
- `reverse()`：反转字符序列；
- `toString()`：转为String对象。

## 八、异常处理
Java用面向对象的方式处理程序运行时的错误，所有异常都是`Throwable`的子类，异常处理的核心是“捕获错误、保证程序不崩溃”。

### 1. 异常体系
```
Throwable
├─ Error：严重错误，程序无法处理，例：OutOfMemoryError、StackOverflowError，只能终止程序
└─ Exception：异常，程序可捕获处理
   ├─ 编译时异常（受检异常）：Exception的直接子类，除RuntimeException，编译时必须处理，否则报错，例：IOException、SQLException
   └─ 运行时异常（非受检异常）：RuntimeException及其子类，编译时不强制处理，运行时抛出，例：NullPointerException、ArrayIndexOutOfBoundsException
```

### 2. 异常处理5个核心关键字
`try`、`catch`、`finally`、`throw`、`throws`

#### （1）try-catch-finally：捕获并处理异常
```java
try {
    // 可能抛出异常的代码
} catch (NullPointerException e) {
    // 捕获空指针异常，处理逻辑
    e.printStackTrace(); // 打印异常堆栈信息
} catch (Exception e) {
    // 捕获其他异常，子类异常在前，父类在后
} finally {
    // 无论是否发生异常，都会执行的代码，除非JVM退出（System.exit(0)）
    // 通常用于释放资源：关闭流、数据库连接等
}
```
> 注意：若try/catch中有return，finally会在return之前执行；finally中不建议写return，会覆盖try/catch的返回值。

#### （2）throws：声明异常
用在方法声明上，声明该方法可能抛出的异常，告诉调用者需要处理该异常。
```java
// 声明方法可能抛出IOException，编译时异常，调用者必须处理
public void readFile() throws IOException {
    // 可能抛出异常的代码
}
```
- 规则：可声明多个异常；子类重写方法，抛出的异常不能大于父类方法声明的异常；运行时异常声明后，不强制调用者处理。

#### （3）throw：手动抛出异常
用在方法体内，手动抛出一个异常对象。
```java
public void setAge(int age) {
    if (age < 0 || age > 150) {
        // 手动抛出运行时异常
        throw new IllegalArgumentException("年龄不合法");
    }
    this.age = age;
}
```
- 规则：throw抛出的是异常对象，只能抛出一个；抛出编译时异常，必须在方法上throws声明，或try-catch处理。

### 3. 自定义异常
继承`Exception`（编译时异常）或`RuntimeException`（运行时异常），实现业务专属异常。
```java
// 自定义运行时异常
public class BusinessException extends RuntimeException {
    // 无参构造
    public BusinessException() {}
    // 带异常信息的构造方法
    public BusinessException(String message) {
        super(message);
    }
}
```

## 九、集合框架
Java集合框架位于`java.util`包，是用于存储和操作对象的容器，长度动态可变，只能存储引用类型，分为**Collection单列集合**和**Map双列集合**两大体系。

### 1. Collection体系
Collection是所有单列集合的根接口，核心子接口为`List`和`Set`。

#### （1）List接口
有序、可重复、有索引，元素存入顺序与取出顺序一致，可通过索引直接访问元素。
| 实现类 | 底层结构 | 特点 | 线程安全 |
|--------|----------|------|----------|
| ArrayList | 动态数组 | 查询快，增删慢，默认初始容量10，扩容1.5倍 | 不安全 |
| LinkedList | 双向链表 | 查询慢，增删快，实现了Deque接口，可作为队列/栈 | 不安全 |
| Vector | 动态数组 | 线程安全，效率低，默认扩容2倍，已被淘汰 | 安全 |

**核心常用方法**：
- 增：`add(E e)`、`add(int index, E e)`、`addAll(Collection c)`
- 删：`remove(int index)`、`remove(Object o)`、`clear()`
- 改：`set(int index, E e)`
- 查：`get(int index)`、`indexOf(Object o)`、`size()`

#### （2）Set接口
无序、不可重复、无索引，最多存储一个null元素，元素唯一性依赖`hashCode()`和`equals()`方法。
| 实现类 | 底层结构 | 特点 | 排序 |
|--------|----------|------|------|
| HashSet | 哈希表（HashMap） | 无序，不可重复，效率高，允许null | 无序 |
| LinkedHashSet | 哈希表+双向链表 | 有序（存入顺序），不可重复，查询略慢于HashSet | 插入有序 |
| TreeSet | 红黑树（TreeMap） | 可排序，不可重复，不允许null | 自然排序/定制排序 |

> 注意：TreeSet元素必须实现`Comparable`接口，或创建时传入`Comparator`比较器，保证排序和唯一性。

#### （3）迭代器Iterator
遍历Collection集合的统一接口，所有Collection集合都实现了`Iterable`接口，可通过`iterator()`获取迭代器。
```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
// 获取迭代器
Iterator<String> it = list.iterator();
// 遍历
while (it.hasNext()) { // 判断是否有下一个元素
    String s = it.next(); // 获取下一个元素
    System.out.println(s);
    // 遍历过程中，只能用迭代器的remove()删除元素，不能用集合的增删方法，否则会报并发修改异常
    it.remove();
}
```

### 2. Map体系
Map是双列集合的根接口，存储`key-value`键值对，key不可重复，value可重复，每个key对应唯一的value。

| 实现类 | 底层结构 | 特点 | 线程安全 | key/value null |
|--------|----------|------|----------|----------------|
| HashMap | 哈希表（数组+链表+红黑树） | 无序，效率高，JDK1.8链表长度>8转红黑树 | 不安全 | 允许key为null（最多1个），value为null |
| LinkedHashMap | 哈希表+双向链表 | 有序（存入顺序），继承HashMap | 不安全 | 同上 |
| TreeMap | 红黑树 | 可按键排序，不允许key为null | 不安全 | key不能为null，value可以 |
| Hashtable | 哈希表 | 线程安全，效率低，已淘汰 | 安全 | 都不允许为null |

**核心常用方法**：
- 增/改：`put(K key, V value)`、`putAll(Map m)`
- 删：`remove(Object key)`、`clear()`
- 查：`get(Object key)`、`containsKey(Object key)`、`containsValue(Object value)`、`size()`

**Map的4种遍历方式**：
```java
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);

// 1. 遍历key，通过key获取value（推荐，仅需key时使用）
for (String key : map.keySet()) {
    System.out.println(key + ":" + map.get(key));
}

// 2. 遍历entry键值对（推荐，效率最高，同时需要key和value时使用）
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ":" + entry.getValue());
}

// 3. 遍历value，仅能获取value，无法获取key
for (Integer value : map.values()) {
    System.out.println(value);
}

// 4. Lambda遍历（Java8+，最简洁）
map.forEach((k, v) -> System.out.println(k + ":" + v));
```

### 3. 集合工具类Collections
`java.util.Collections`提供了集合操作的静态方法：
- `sort(List list)`：对List集合排序；
- `reverse(List list)`：反转List集合；
- `shuffle(List list)`：随机打乱List集合；
- `max()/min()`：获取集合的最大值/最小值；
- `synchronizedList()`：将线程不安全的集合转为线程安全的集合；
- `unmodifiableList()`：返回不可修改的集合，防止数据被篡改。

## 十、泛型
Java5+引入，参数化类型，将类型作为参数传递，**编译期检查类型安全**，避免强制类型转换，解决集合的类型安全问题。

### 1. 泛型的核心使用
#### （1）泛型类
在类上定义泛型，实例化时指定具体类型。
```java
// 泛型类，T为类型参数
public class GenericClass<T> {
    private T data;
    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}

// 使用，指定泛型为String
GenericClass<String> gc = new GenericClass<>();
gc.setData("test");
String data = gc.getData(); // 无需强制类型转换
```

#### （2）泛型方法
在方法上独立定义泛型，与类的泛型无关，调用方法时确定类型，支持静态方法。
```java
public class GenericMethod {
    // 泛型方法，<T>声明泛型
    public <T> void print(T t) {
        System.out.println(t);
    }

    // 静态泛型方法
    public static <T> T getInstance(Class<T> clazz) throws Exception {
        return clazz.newInstance();
    }
}
```

#### （3）泛型接口
在接口上定义泛型，实现类可指定具体类型，或继续使用泛型。
```java
// 泛型接口
public interface GenericInterface<T> {
    T get();
}

// 实现类指定具体类型
public class StringImpl implements GenericInterface<String> {
    @Override
    public String get() {
        return "test";
    }
}

// 实现类继续使用泛型
public class GenericImpl<T> implements GenericInterface<T> {
    @Override
    public T get() {
        return null;
    }
}
```

### 2. 泛型通配符
`?`代表不确定的类型，用于接收任意泛型类型，分为3种：
1. **无界通配符 `?`**：可接收任意泛型类型，例：`List<?>`，只能读取元素，不能添加元素（除了null）；
2. **上限通配符 `? extends 类型`**：只能接收该类型及其子类，例：`List<? extends Number>`，可读取Number类型，不能添加元素；
3. **下限通配符 `? super 类型`**：只能接收该类型及其父类，例：`List<? super Number>`，可添加Number及其子类，读取只能用Object接收。

### 3. 泛型擦除
泛型仅在**编译期有效**，运行时会擦除所有泛型信息，泛型参数会被替换为上限类型（无上限则为Object），所以运行时无法获取泛型的具体类型。

## 十一、多线程
进程是操作系统资源分配的最小单位，线程是CPU调度的最小单位，一个进程可包含多个线程，Java天生支持多线程。

### 1. 线程的3种创建方式
#### （1）继承Thread类
继承`Thread`类，重写`run()`方法，调用`start()`启动线程。
```java
public class MyThread extends Thread {
    // 线程执行体
    @Override
    public void run() {
        System.out.println("线程执行：" + Thread.currentThread().getName());
    }
}

// 启动线程
public class Test {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start(); // 启动线程，不能直接调用run()，否则不会开启新线程
    }
}
```

#### （2）实现Runnable接口
实现`Runnable`接口，重写`run()`方法，避免单继承限制，推荐使用。
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("线程执行");
    }
}

// 启动线程
public class Test {
    public static void main(String[] args) {
        new Thread(new MyRunnable()).start();
        // Lambda简化（Runnable是函数式接口）
        new Thread(() -> System.out.println("线程执行")).start();
    }
}
```

#### （3）实现Callable接口
Java5+引入，有返回值，可抛出异常，配合`FutureTask`使用。
```java
public class MyCallable implements Callable<Integer> {
    // 线程执行体，有返回值，可抛出异常
    @Override
    public Integer call() throws Exception {
        System.out.println("线程执行");
        return 100;
    }
}

// 启动线程
public class Test {
    public static void main(String[] args) throws Exception {
        FutureTask<Integer> task = new FutureTask<>(new MyCallable());
        new Thread(task).start();
        // get()会阻塞，直到线程执行完毕，获取返回值
        Integer result = task.get();
        System.out.println(result);
    }
}
```

### 2. 线程的生命周期（6种状态）
`Thread.State`枚举定义了线程的6种状态，状态流转如下：
1. **新建（New）**：线程对象已创建，未调用`start()`；
2. **就绪（Runnable）**：调用`start()`后，等待CPU调度，包含就绪和运行中两个子状态；
3. **阻塞（Blocked）**：线程等待锁资源，无法进入同步代码块，获取锁后回到就绪状态；
4. **等待（Waiting）**：无限期等待，调用`wait()`、`join()`等方法，需其他线程唤醒；
5. **超时等待（Timed Waiting）**：有时间限制的等待，调用`sleep(long)`、`wait(long)`等方法，时间到自动唤醒；
6. **终止（Terminated）**：`run()`方法执行完毕，或抛出未捕获的异常，线程结束，不可再次启动。

### 3. 线程的核心常用方法
| 方法 | 作用 |
|------|------|
| `start()` | 启动线程，进入就绪状态，JVM调用run()方法 |
| `run()` | 线程执行体，定义线程的业务逻辑 |
| `sleep(long millis)` | 静态方法，当前线程休眠指定毫秒，进入超时等待，**不会释放锁** |
| `join()` | 等待该线程执行完毕，当前线程进入等待状态 |
| `yield()` | 静态方法，当前线程让出CPU，回到就绪状态 |
| `interrupt()` | 中断线程，设置中断标记，不是强制停止线程 |
| `setDaemon(boolean on)` | 设置为守护线程（后台线程），必须在start()前调用，所有用户线程结束后，守护线程自动结束 |
| `setPriority(int newPriority)` | 设置线程优先级，范围1~10，默认5，优先级越高，CPU调度概率越高 |
| `currentThread()` | 静态方法，获取当前正在执行的线程对象 |

### 4. 线程同步（解决线程安全问题）
多线程共享资源时，会出现线程安全问题（例：超卖、数据脏读），同步机制保证操作的**原子性、可见性、有序性**。

#### （1）synchronized关键字
Java内置锁，可重入，自动获取和释放锁，保证原子性、可见性、有序性。
- 修饰实例方法：锁当前对象`this`，同一时间只有一个线程能进入该对象的同步方法；
- 修饰静态方法：锁当前类的Class对象，同一时间只有一个线程能进入该类的静态同步方法；
- 修饰代码块：锁括号内的对象，粒度更细，推荐使用。
```java
public class Ticket implements Runnable {
    private int ticket = 100;
    private final Object lock = new Object();

    @Override
    public void run() {
        while (true) {
            // 同步代码块
            synchronized (lock) {
                if (ticket <= 0) break;
                System.out.println(Thread.currentThread().getName() + "卖出第" + ticket + "张票");
                ticket--;
            }
        }
    }
}
```

#### （2）Lock锁
Java5+引入，手动锁，更灵活，支持公平锁、可中断、超时获取锁，核心实现类`ReentrantLock`。
```java
public class Ticket implements Runnable {
    private int ticket = 100;
    // 创建Lock锁，true为公平锁，默认false非公平锁
    private final Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            lock.lock(); // 获取锁
            try {
                if (ticket <= 0) break;
                System.out.println(Thread.currentThread().getName() + "卖出第" + ticket + "张票");
                ticket--;
            } finally {
                lock.unlock(); // 释放锁，必须在finally中，避免死锁
            }
        }
    }
}
```

#### （3）volatile关键字
修饰成员变量，保证**可见性和有序性**，不保证原子性。
- 可见性：一个线程修改了变量值，其他线程能立即看到最新值，禁止变量缓存在工作内存；
- 有序性：禁止指令重排序，通过内存屏障实现；
- 常用场景：单例模式双重检查锁（DCL）、状态标记量。

### 5. 线程池
避免频繁创建销毁线程带来的性能开销，控制线程数量，避免OOM，Java5+位于`java.util.concurrent`包。

#### （1）ThreadPoolExecutor核心参数
手动创建线程池（推荐，阿里规范禁止使用Executors创建），核心构造参数：
```java
public ThreadPoolExecutor(
    int corePoolSize, // 核心线程数，常驻线程
    int maximumPoolSize, // 最大线程数
    long keepAliveTime, // 非核心线程空闲存活时间
    TimeUnit unit, // 时间单位
    BlockingQueue<Runnable> workQueue, // 任务队列，存放等待执行的任务
    ThreadFactory threadFactory, // 线程工厂，创建线程
    RejectedExecutionHandler handler // 拒绝策略，任务满时的处理方式
) {}
```

#### （2）线程池常用方法
- `execute(Runnable command)`：执行无返回值的任务；
- `submit(Callable<T> task)`：执行有返回值的任务，返回`Future<T>`；
- `shutdown()`：关闭线程池，不再接收新任务，等待已提交任务执行完毕；
- `shutdownNow()`：立即关闭线程池，尝试中断正在执行的任务，返回未执行的任务列表。

## 十二、Java8³¹-1（约±21亿） | 0 | 整数默认类型，最常用 |
| long | 8字节 | -2⁶³ ~ 2⁶³-1 | 0L | 长整型，声明时必须加`L/l`后缀，例：`long num = 100L;` |

#### （2）浮点型（带小数）
| 类型 | 占用字节 | 精度 | 默认值 | 说明 |
|------|----------|------|--------|------|
| float | 4字节 | 单精度，6~7位有效数字 | 0.0f | 单精度浮点，声明时必须加`F/f`后缀，例：`float f = 3.14f;` |
| double | 8字节 | 双精度，15~16位有效数字 | 0.0 | 浮点默认类型，精度更高，日常开发优先使用 |

> 注：浮点型存在精度丢失问题，金融场景优先使用`BigDecimal`。

#### （3）字符型
| 类型 | 占用字节 | 取值范围 | 默认值 | 说明 |
|------|----------|----------|--------|------|
| char | 2字节 | 0 ~ 65535 | '\u0000' | 无符号类型，用单引号包裹，可存储Unicode字符、中文，支持转义字符（`\n`换行、`\t`制表、`\\`反斜杠等） |

#### （4）布尔型
| 类型 | 占用空间 | 取值 | 默认值 | 说明 |
|------|----------|------|--------|------|
| boolean | JVM底层用int实现，数组用byte数组 | true/false | false | 仅能表示真假，不能与整数类型互转 |

### 2. 引用数据类型
引用类型指向堆内存中的对象，默认值为`null`，核心包括：
- 类（Class）：如String、Object、自定义类
- 接口（Interface）
- 数组（Array）
- 枚举（Enum）
- 注解（Annotation）

### 3. 类型转换
#### （1）自动类型转换（隐式）
小范围类型自动转为大范围类型，无精度丢失，转换顺序：
`byte → short → int → long → float → double`
`char → int`
> 注意：byte、short、char之间运算时，会自动提升为int类型。

#### （2）强制类型转换（显式）
大范围类型转为小范围类型，可能出现精度丢失或数据溢出，需手动声明，格式：
`(目标类型) 变量/值`
示例：
```java
double pi = 3.1415;
int intPi = (int) pi; // 结果3，丢失小数精度
```

## 四、运算符
Java运算符按功能分为7大类，优先级从高到低为：**单目 > 算术 > 移位 > 比较 > 位运算 > 逻辑 > 三元 > 赋值**，括号优先级最高。

### 1. 算术运算符
| 运算符 | 说明 | 注意事项 |
|--------|------|----------|
| + | 加法、字符串拼接 | 任意类型与字符串+，都会转为字符串拼接 |
| - | 减法 | - |
| * | 乘法 | - |
| / | 除法 | 整数相除结果为整数，例：5/2=2；5.0/2=2.5 |
| % | 取余（取模） | 结果符号与被除数一致，例：5%2=1，-5%2=-1 |
| ++ | 自增1 | 前置++：先自增，再运算；后置++：先运算，再自增 |
| -- | 自减1 | 与++规则一致 |

### 2. 赋值运算符
| 运算符 | 说明 | 示例 |
|--------|------|------|
| = | 基本赋值 | int a = 10; |
| +=、-=、*=、/=、%= | 复合赋值 | a += 5; 等价于 a = a + 5; 自动强转类型 |
| &=、|=、^=、<<=、>>=、>>>= | 位运算复合赋值 | 同上，先运算再赋值 |

> 注：复合赋值运算符会自动处理类型强转，例：`byte b=10; b+=5;`不会报错，而`b=b+5`会因int类型提升编译报错。

### 3. 比较运算符
所有比较运算符的结果都是boolean类型（true/false）。
| 运算符 | 说明 |
|--------|------|
| == | 等于：基本类型比较值，引用类型比较内存地址 |
| != | 不等于+核心新特性
### 1. Lambda表达式
函数式编程，简化函数式接口的匿名内部类写法，格式：
`(参数列表) -> { 代码体 }`
- 简化规则：参数类型可省略；单个参数可省略括号；代码体只有一行，可省略大括号、return、分号。
```java
// 匿名内部类
Collections.sort(list, new Comparator |
| >、< | 大于、小于 |
| >=、<= | 大于等于、小于等于 |
| instanceof | 判断对象是否是指定类/接口的实例，例：`"abc" instanceof String` |

### 4. 逻辑运算符
用于多个boolean表达式的逻辑运算，核心区别是**短路与/或**会中断执行，效率更高。
| 运算符 | 说明 | 规则 |
|--------|------|------|
| && | 短路与 | 左边为false时，右边代码不执行，全真才为真 |
| || | 短路或 | 左边为true时，右边代码不执行，全假才为假 |
| ! | 逻辑非 | 取反，!true=false，!false=true |
| & | 逻辑与 | 无论左边结果，右边始终执行，全真才为真 |
| | | 逻辑或 | 无论左边结果，右边始终执行，全假才为假 |
| ^ | 异或 | 两边结果不同为true，相同为false |

### 5. 位运算符
直接对二进制位进行运算，执行效率极高。
| 运算符 | 说明 | 示例 |
|--------|------|------|
| & | 按位与 | 两位都为1，结果为1 |
| | | 按位或 | 两位有一个为1，结果为1 |
| ^ | 按位异或 | 两位不同为1，相同为0 |
| ~ | 按位取反 | 0变1，1变0 |
| << | 左移 | 左移n位，等价于 *2ⁿ，例：2<<3=16 |
| >> | 带符号右移 | 右移n位，等价于 /2ⁿ，负数用1补高位 |
| >>> | 无符号右移 | 无论正负，都用0补高位，仅对正数有效 |

### 6. 三元运算符
Java唯一的三目运算符，可简化简单的if-else逻辑，格式：
`条件表达式 ? 表达式1 : 表达式2`
- 条件为true，执行表达式1，返回其结果；
- 条件为false，执行表达式2，返回其结果；
- 必须有返回值，可嵌套使用。
示例：`int max = a > b ? a : b;`

## 五、流程控制语句
Java流程控制分为**顺序结构、分支结构、循环结构**三大类，是程序执行的核心逻辑。

### 1. 分支结构
#### （1）if-else 条件分支
```java
// 单分支
if (条件表达式) {
    // 条件为true执行
}

// 双分支
if (条件表达式) {
    // 条件为true执行
} else {
    // 条件为false执行
}

// 多分支
if (条件1) {
    // 条件1为true执行
} else if (条件2) {
    // 条件2为true执行
} else {
    // 所有条件都不满足执行
}
```
> 注意：else永远匹配离它最近的、未匹配的if。

#### （2）switch 选择分支
用于固定值的多分支匹配，Java 7+支持String类型，Java 14+支持switch表达式。
```java
// 传统switch
switch (表达式) {
    case 常量1:
        // 匹配常量1执行
        break; // 可选，无break会发生case穿透
    case 常量2:
        // 匹配常量2执行
        break;
    default:
        // 所有case都不匹配执行，可选，位置任意
        break;
}

// Java14+ switch表达式（有返回值，无需break）
int result = switch (day) {
    case 1,2,3,4,5 -> 1; // 工作日
    case 6,7 -> 0; // 休息日
    default -> -1;
};
```
**核心规则**：
- 表达式支持的类型：byte、short、int、char、枚举、String（Java7+）；
- case后的常量不能重复，不能是变量；
- break用于终止switch，无break会发生case穿透，后续case无论是否匹配都会执行。

### 2. 循环结构
用于重复执行某段代码，核心分为4种：

#### （1）for循环
适合循环次数已知的场景，格式：
```java
for (初始化表达式; 条件表达式; 更新表达式) {
    // 循环体
}
```
- 初始化表达式：仅在循环开始时执行1次，用于声明循环变量；
- 条件表达式：每次循环前判断，为true执行循环体，为false终止循环；
- 更新表达式：每次循环体执行完毕后执行，用于更新循环变量。

#### （2）增强for循环（foreach）
Java5+引入，用于遍历数组/集合，无索引，简化遍历代码，格式：
```java
for (元素类型 变量名 : 数组/集合) {
    // 循环体，变量名代表当前遍历的元素
}
```
> 注意：仅能遍历读取元素，不能修改元素值，无法获取索引；不能遍历过程中增删集合元素。

#### （3）while循环
适合循环次数未知的场景，先判断后执行，循环体可能一次都不执行，格式：
```java
while (条件表达式) {
    // 循环体
}
```

#### （4）do-while循环
先执行后判断，循环体**至少执行一次**，格式：
```java
do {
    // 循环体
} while (条件表达式); // 结尾必须有分号
```

### 3. 跳转语句
| 语句 | 作用 |
|------|------|
| break | 跳出当前循环/switch，结束整个循环；支持带标签的break，跳出指定外层循环 |
| continue | 跳过本次循环的剩余代码，直接进入下一次循环；支持带标签的continue |
| return | 结束当前方法，返回指定值（若方法有返回值），终止方法内所有后续代码 |

## 六、面向对象核心（OOP）
Java是纯面向对象语言，面向对象的三大核心特性：**封装、继承、多态**。

### 1. 类与对象
- 类：对象的抽象模板，定义了对象的属性（成员变量）和行为（成员方法）；
- 对象：类的具体实例，是类的具象化，通过`new`关键字创建。

#### （1）类的定义
```java
// 类定义
public class User {
    // 成员变量（属性）
    private String username;
    private int age;

    // 无参构造方法
    public User() {}

    // 有参构造方法
    public User(String username, int age) {
        this.username = username;
        this.age = age;
    }

    // 成员方法（行为）
    public void showInfo() {
        System.out.println("用户名：" + username + "，年龄：" + age);
    }

    // get/set方法
    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

#### （2）对象的创建与使用
```java
// 创建对象
User user = new User("张三", 20);
// 调用成员方法
user.showInfo();
// 调用set/get方法
user.setUsername("李四");
System.out.println(user.getUsername());
```

#### （3）成员变量 vs 局部变量
| 维度 | 成员变量 | 局部变量 |
|------|----------|----------|
| 定义位置 | 类中，方法/代码块外 | 方法、代码块、形参中 |
| 内存位置 | 堆内存（随对象存储） | 栈内存 |
| 生命周期 | 随对象创建而存在，随对象回收而销毁 | 随方法调用创建，方法执行完毕销毁 |
| 默认值 | 有对应数据类型的默认值 | 无默认值，必须先赋值再使用 |
| 修饰符 | 可使用访问修饰符、static、final等 | 不能使用访问修饰符、static，可使用final |

### 2. 封装
**核心思想**：隐藏对象的属性和实现细节，仅对外暴露公共的访问方式，提高代码安全性和复用性。
- 实现方式：用`private`修饰成员变量，对外提供`public`的`get/set`方法，可在方法中添加数据校验逻辑。
- 访问修饰符（权限从大到小）：

| 修饰符 | 同类 | 同包 | 不同包子类 | 不同包非子类 |
|--------|------|------|------------|--------------|
| public | ✅ | ✅ | ✅ | ✅ |
| protected | ✅ | ✅ | ✅ | ❌ |
| default（包私有，不写） | ✅ | ✅ | ❌ | ❌ |
| private | ✅ | ❌ | ❌ | ❌ |

### 3. 继承
**核心思想**：子类继承父类的非私有属性和方法，实现代码复用，Java是**单继承**，一个类只能有一个直接父类，支持多层继承，所有类默认直接/间接继承`Object`类。

#### （1）继承的定义
```java
// 父类
public class Animal {
    protected String name;
    public void eat() {
        System.out.println("动物吃东西");
    }
}

// 子类，继承Animal
public class Cat extends Animal {
    // 子类特有方法
    public void catchMouse() {
        System.out.println("猫抓老鼠");
    }

    // 重写父类方法
    @Override
    public void eat() {
        System.out.println("猫吃鱼");
    }
}
```

#### （2）super关键字
代表父类对象的引用，核心用法：
1. `super.成员变量`：访问父类的成员变量；
2. `super.成员方法`：调用父类的成员方法；
3. `super(参数)`：调用父类的构造方法，**必须放在子类构造方法的第一行**。
> 注意：子类构造方法默认会自动调用父类的无参构造`super()`，若父类没有无参构造，必须手动调用父类的有参构造。

#### （3）方法重写（Override）
子类对父类的方法进行重新实现，是多态的前提，**重写规则**：
- 方法名、参数列表必须与父类完全一致；
- 返回值类型：子类返回值类型必须小于等于父类（协变返回类型）；
- 访问修饰符：子类权限必须大于等于父类；
- 不能抛出比父类更大的编译时异常；
- 父类的`private`、`static`、`final`方法不能被重写；
- `@Override`注解用于校验重写语法是否正确。

#### （4）重写 vs 重载
| 维度 | 重写（Override） | 重载（Overload） |
|------|-------------------|-------------------|
| 作用范围 | 父子类之间 | 同一个类中 |
| 方法名 | 必须相同 | 必须相同 |
| 参数列表 | 必须完全相同 | 必须不同（个数、类型、顺序） |
| 返回值 | 必须一致（协变） | 无要求 |
| 修饰符 | 权限不能更小，不能用static/final | 无要求 |
| 多态类型 | 运行时多态 | 编译时多态 |

### 4. 多态
**核心定义**：同一个行为，不同的对象具有不同的表现形式，多态的前提：
1. 继承/实现关系；
2. 方法重写；
3. 父类引用指向子类对象。

#### （1）多态的使用
```java
// 父类引用指向子类对象，向上转型
Animal animal = new Cat();
// 编译看左边，运行看右边：编译时只能调用Animal类的方法，运行时执行Cat重写的方法
animal.eat(); // 输出：猫吃鱼
// 无法调用子类特有方法，编译报错
// animal.catchMouse();

// 向下转型，强制类型转换，可调用子类特有方法
if (animal instanceof Cat) { // 先判断类型，避免类型转换异常
    Cat cat = (Cat) animal;
    cat.catchMouse(); // 输出：猫抓老鼠
}
```

#### （2）向上转型 vs 向下转型
- **向上转型**：子类对象→父类引用，自动转换，会丢失子类特有方法，实现通用化处理；
- **向下转型**：父类引用→子类对象，强制转换，可恢复子类特有方法，必须用`instanceof`判断类型，避免`ClassCastException`。

### 5. 核心关键字
#### （1）static关键字
`static`修饰的成员属于**类本身**，不属于某个对象，被所有对象共享，在类加载时初始化，优先于对象存在。
- 核心用法：
  1. **静态变量（类变量）**：通过`类名.变量名`访问，无需创建对象，常用于全局常量；
  2. **静态方法（类方法）**：通过`类名.方法名`调用，静态方法中只能访问静态成员，不能使用`this/super`，不能访问非静态成员；
  3. **静态代码块**：`static { 代码 }`，类加载时仅执行一次，用于初始化静态资源；
  4. **静态内部类**：修饰内部类，属于外部类本身。
> 注意：static不能修饰局部变量，不能修饰外部类。

#### （2）final关键字
`final`代表“最终的、不可修改的”，核心用法：
1. **修饰类**：该类不能被继承，例：String、Math、Integer等包装类；
2. **修饰方法**：该方法不能被子类重写；
3. **修饰变量**：变为常量，赋值后不可修改；
   - 修饰基本类型：值不可修改；
   - 修饰引用类型：地址不可修改，对象的内容可修改；
   - 成员常量：必须在定义时、代码块、构造方法中赋值，三选一；
   - 静态常量：`static final`，必须在定义时或静态代码块中赋值，全大写命名。

### 6. 抽象类与接口
#### （1）抽象类
用`abstract`修饰的类，用于抽取子类的通用特性，**不能实例化**，只能被继承。
- 核心规则：
  1. 包含抽象方法的类，必须是抽象类；抽象类可以没有抽象方法；
  2. 抽象方法：`abstract`修饰，无方法体，子类必须重写所有抽象方法，否则子类也必须是抽象类；
  3. 抽象类可包含普通方法、成员变量、构造方法、代码块；
  4. `abstract`不能与`final`、`private`、`static`共用。
```java
// 抽象类
public abstract class Shape {
    // 抽象方法，无方法体
    public abstract double getArea();
    // 普通方法
    public void show() {
        System.out.println("这是一个图形");
    }
}

// 子类继承抽象类，必须重写所有抽象方法
public class Circle extends Shape {
    private double radius;
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
}
```

#### （2）接口
用`interface`修饰，是行为的规范/契约，定义了类必须具备的能力，Java8+后功能大幅扩展。
- 核心规则：
  1. 接口不能实例化，只能被类`implements`实现，一个类可实现多个接口，解决单继承限制；
  2. 成员变量：默认是`public static final`，只能是常量，必须初始化；
  3. 抽象方法：默认是`public abstract`，实现类必须重写所有抽象方法，否则为抽象类；
  4. 默认方法（Java8+）：`default`修饰，有方法体，实现类可重写可不重写，解决接口新增方法的兼容性问题；
  5. 静态方法（Java8+）：`static`修饰，有方法体，只能通过`接口名.方法名`调用，实现类不能继承；
  6. 私有方法（Java9+）：`private`修饰，有方法体，用于接口内方法的代码复用；
  7. 接口可继承多个接口，格式：`interface A extends B,C {}`。
```java
// 接口定义
public interface Flyable {
    // 常量，默认public static final
    int MAX_SPEED = 1000;
    // 抽象方法，默认public abstract
    void fly();
    // 默认方法
    default void show() {
        System.out.println("可以飞行");
    }
    // 静态方法
    static void staticMethod() {
        System.out.println("接口静态方法");
    }
}

// 类实现接口
public class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("鸟用翅膀飞行");
    }
}
```

#### （3）抽象类 vs 接口
| 维度 | 抽象类 | 接口 |
|------|--------|------|
| 继承/实现 | 单继承，一个类只能extends一个抽象类 | 多实现，一个类可implements多个接口 |
| 成员变量 | 可普通变量，可常量 | 只能是public static final常量 |
| 方法 | 可抽象方法、普通方法、静态方法、final方法 | Java7：仅抽象方法；Java8+：默认、静态方法；Java9+：私有方法 |
| 构造方法 | 有构造方法 | 无构造方法 |
| 设计目的 | 代码复用，抽取子类通用的属性和行为 | 行为规范，定义类的能力，解耦 |

### 7. 枚举（enum）
Java5+引入，用于定义固定的常量集合，例：季节、星期、订单状态等，本质是继承`Enum`的final类。
```java
// 枚举定义
public enum Season {
    // 枚举常量，默认public static final，是枚举类的实例
    SPRING("春天", "春暖花开"),
    SUMMER("夏天", "烈日炎炎"),
    AUTUMN("秋天", "秋高气爽"),
    WINTER("冬天", "冰天雪地");

    // 成员变量
    private final String name;
    private final String desc;

    // 构造方法，必须private，默认private
    Season(String name, String desc) {
        this.name = name;
        this.desc = desc;
    }

    // get方法
    public String getName() {
        return name;
    }
}

// 枚举使用
public class Test {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        System.out.println(spring.getName());
        // 遍历枚举
        for (Season season : Season.values()) {
            System.out.println(season);
        }
    }
}
```
- 核心方法：
  - `values()`：返回所有枚举常量的数组；
  - `valueOf(String name)`：返回指定名称的枚举常量；
  - `ordinal()`：返回枚举常量的序号，从0开始。

## 七、数组与字符串
### 1. 数组
数组是**相同数据类型元素的有序集合**，长度固定，一旦创建不可修改，可存储基本类型和引用类型。

#### （1）数组的声明与初始化
```java
// 声明，推荐第一种
int[] arr1;
int arr2[];

// 静态初始化：指定元素，系统计算长度
int[] arr3 = {1,2,3,4,5};
int[] arr4 = new int[]{1,2,3,4,5};

// 动态初始化：指定长度，系统赋默认值
int[] arr5 = new int[5]; // 长度5，默认值0
String[] arr6 = new String[3]; // 长度3，默认值null
```

#### （2）数组的访问与遍历
```java
// 访问：数组名[索引]，索引从0开始，最大索引=长度-1
int[] arr = {1,2,3};
System.out.println(arr[0]); // 输出1
arr[1] = 10; // 修改元素

// 长度：数组名.length属性
System.out.println(arr.length); // 输出3

// 遍历1：普通for循环
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}

// 遍历2：增强for循环
for (int num : arr) {
    System.out.println(num);
}
```

#### （3）二维数组
数组的数组，每个元素是一个一维数组，长度可不一致。
```java
// 动态初始化
int[][] arr1 = new int[3][2]; // 3行2列
// 静态初始化
int[][] arr2 = {{1,2}, {3,4,5}, {6}};
// 访问
System.out.println(arr2[0][1]); // 输出2
```

#### （4）数组工具类Arrays
`java.util.Arrays`提供了数组操作的常用方法：
- `sort()`：数组排序；
- `binarySearch()`：二分查找元素，返回索引；
- `copyOf()`：复制数组，生成新数组；
- `fill()`：用指定值填充数组；
- `toString()`：将数组转为字符串，方便打印；
- `equals()`：比较两个数组的元素是否完全一致。

### 2. 字符串
Java字符串核心分为3类：`String`、`StringBuffer`、`StringBuilder`，位于`java.lang`包。

#### （1）String类
不可变字符序列，一旦创建，内容不可修改，任何修改都会创建新的String对象。
```java
// 直接赋值，存放在字符串常量池，复用对象
String s1 = "abc";
// new创建，堆内存创建新对象
String s2 = new String("abc");
```
**核心常用方法**：
| 方法 | 作用 |
|------|------|
| `length()` | 返回字符串长度 |
| `charAt(int index)` | 返回指定索引的字符 |
| `equals(Object obj)` | 比较字符串内容是否相等，重写了Object的方法 |
| `equalsIgnoreCase(String str)` | 忽略大小写比较内容 |
| `contains(CharSequence s)` | 判断是否包含指定子串 |
| `indexOf(String str)` | 返回子串第一次出现的索引，无则返回-1 |
| `substring(int beginIndex)` | 截取从beginIndex到结尾的子串 |
| `substring(int beginIndex, int endIndex)` | 截取[beginIndex, endIndex)范围的子串，左闭右开 |
| `replace(char oldChar, char newChar)` | 替换所有指定字符 |
| `split(String regex)` | 按正则表达式分割为字符串数组 |
| `trim()` | 去除首尾空格 |
| `toUpperCase()/toLowerCase()` | 转为全大写/全小写 |
| `toCharArray()` | 转为字符数组 |
| `valueOf(Xxx x)` | 静态方法，将其他类型转为字符串 |

> 注意：String重写了`equals()`和`hashCode()`，`==`比较的是内存地址，`equals()`比较的是内容。

#### （2）StringBuffer & StringBuilder
可变字符序列，内容修改不会创建新对象，直接修改原对象，适合频繁修改字符串的场景。

| 维度 | StringBuffer | StringBuilder |
|------|--------------|---------------|
| 线程安全 | 线程安全，方法加了`synchronized` | 线程不安全，无同步锁 |
| 执行效率 | 低 | 高 |
| 版本 | JDK1.0 | JDK1.5 |

**核心常用方法**：
- `append(Xxx x)`：追加内容，支持所有类型，链式调用；
- `insert(int offset, Xxx x)`：在指定位置插入内容；
- `delete(int start, int end)`：删除指定范围的字符；
- `replace(int start, int end, String str)`：替换指定范围的内容；
- `reverse()`：反转字符序列；
- `toString()`：转为String对象。

## 八、异常处理
Java用面向对象的方式处理程序运行时的错误，所有异常都是`Throwable`的子类，异常处理的核心是“捕获错误、保证程序不崩溃”。

### 1. 异常体系
```
Throwable
├─ Error：严重错误，程序无法处理，例：OutOfMemoryError、StackOverflowError，只能终止程序
└─ Exception：异常，程序可捕获处理
   ├─ 编译时异常（受检异常）：Exception的直接子类，除RuntimeException，编译时必须处理，否则报错，例：IOException、SQLException
   └─ 运行时异常（非受检异常）：RuntimeException及其子类，编译时不强制处理，运行时抛出，例：NullPointerException、ArrayIndexOutOfBoundsException
```

### 2. 异常处理5个核心关键字
`try`、`catch`、`finally`、`throw`、`throws`

#### （1）try-catch-finally：捕获并处理异常
```java
try {
    // 可能抛出异常的代码
} catch (NullPointerException e) {
    // 捕获空指针异常，处理逻辑
    e.printStackTrace(); // 打印异常堆栈信息
} catch (Exception e) {
    // 捕获其他异常，子类异常在前，父类在后
} finally {
    // 无论是否发生异常，都会执行的代码，除非JVM退出（System.exit(0)）
    // 通常用于释放资源：关闭流、数据库连接等
}
```
> 注意：若try/catch中有return，finally会在return之前执行；finally中不建议写return，会覆盖try/catch的返回值。

#### （2）throws：声明异常
用在方法声明上，声明该方法可能抛出的异常，告诉调用者需要处理该异常。
```java
// 声明方法可能抛出IOException，编译时异常，调用者必须处理
public void readFile() throws IOException {
    // 可能抛出异常的代码
}
```
- 规则：可声明多个异常；子类重写方法，抛出的异常不能大于父类方法声明的异常；运行时异常声明后，不强制调用者处理。

#### （3）throw：手动抛出异常
用在方法体内，手动抛出一个异常对象。
```java
public void setAge(int age) {
    if (age < 0 || age > 150) {
        // 手动抛出运行时异常
        throw new IllegalArgumentException("年龄不合法");
    }
    this.age = age;
}
```
- 规则：throw抛出的是异常对象，只能抛出一个；抛出编译时异常，必须在方法上throws声明，或try-catch处理。

### 3. 自定义异常
继承`Exception`（编译时异常）或`RuntimeException`（运行时异常），实现业务专属异常。
```java
// 自定义运行时异常
public class BusinessException extends RuntimeException {
    // 无参构造
    public BusinessException() {}
    // 带异常信息的构造方法
    public BusinessException(String message) {
        super(message);
    }
}
```

## 九、集合框架
Java集合框架位于`java.util`包，是用于存储和操作对象的容器，长度动态可变，只能存储引用类型，分为**Collection单列集合**和**Map双列集合**两大体系。

### 1. Collection体系
Collection是所有单列集合的根接口，核心子接口为`List`和`Set`。

#### （1）List接口
有序、可重复、有索引，元素存入顺序与取出顺序一致，可通过索引直接访问元素。
| 实现类 | 底层结构 | 特点 | 线程安全 |
|--------|----------|------|----------|
| ArrayList | 动态数组 | 查询快，增删慢，默认初始容量10，扩容1.5倍 | 不安全 |
| LinkedList | 双向链表 | 查询慢，增删快，实现了Deque接口，可作为队列/栈 | 不安全 |
| Vector | 动态数组 | 线程安全，效率低，默认扩容2倍，已被淘汰 | 安全 |

**核心常用方法**：
- 增：`add(E e)`、`add(int index, E e)`、`addAll(Collection c)`
- 删：`remove(int index)`、`remove(Object o)`、`clear()`
- 改：`set(int index, E e)`
- 查：`get(int index)`、`indexOf(Object o)`、`size()`

#### （2）Set接口
无序、不可重复、无索引，最多存储一个null元素，元素唯一性依赖`hashCode()`和`equals()`方法。
| 实现类 | 底层结构 | 特点 | 排序 |
|--------|----------|------|------|
| HashSet | 哈希表（HashMap） | 无序，不可重复，效率高，允许null | 无序 |
| LinkedHashSet | 哈希表+双向链表 | 有序（存入顺序），不可重复，查询略慢于HashSet | 插入有序 |
| TreeSet | 红黑树（TreeMap） | 可排序，不可重复，不允许null | 自然排序/定制排序 |

> 注意：TreeSet元素必须实现`Comparable`接口，或创建时传入`Comparator`比较器，保证排序和唯一性。

#### （3）迭代器Iterator
遍历Collection集合的统一接口，所有Collection集合都实现了`Iterable`接口，可通过`iterator()`获取迭代器。
```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
// 获取迭代器
Iterator<String> it = list.iterator();
// 遍历
while (it.hasNext()) { // 判断是否有下一个元素
    String s = it.next(); // 获取下一个元素
    System.out.println(s);
    // 遍历过程中，只能用迭代器的remove()删除元素，不能用集合的增删方法，否则会报并发修改异常
    it.remove();
}
```

### 2. Map体系
Map是双列集合的根接口，存储`key-value`键值对，key不可重复，value可重复，每个key对应唯一的value。

| 实现类 | 底层结构 | 特点 | 线程安全 | key/value null |
|--------|----------|------|----------|----------------|
| HashMap | 哈希表（数组+链表+红黑树） | 无序，效率高，JDK1.8链表长度>8转红黑树 | 不安全 | 允许key为null（最多1个），value为null |
| LinkedHashMap | 哈希表+双向链表 | 有序（存入顺序），继承HashMap | 不安全 | 同上 |
| TreeMap | 红黑树 | 可按键排序，不允许key为null | 不安全 | key不能为null，value可以 |
| Hashtable | 哈希表 | 线程安全，效率低，已淘汰 | 安全 | 都不允许为null |

**核心常用方法**：
- 增/改：`put(K key, V value)`、`putAll(Map m)`
- 删：`remove(Object key)`、`clear()`
- 查：`get(Object key)`、`containsKey(Object key)`、`containsValue(Object value)`、`size()`

**Map的4种遍历方式**：
```java
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);

// 1. 遍历key，通过key获取value（推荐，仅需key时使用）
for (String key : map.keySet()) {
    System.out.println(key + ":" + map.get(key));
}

// 2. 遍历entry键值对（推荐，效率最高，同时需要key和value时使用）
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ":" + entry.getValue());
}

// 3. 遍历value，仅能获取value，无法获取key
for (Integer value : map.values()) {
    System.out.println(value);
}

// 4. Lambda遍历（Java8+，最简洁）
map.forEach((k, v) -> System.out.println(k + ":" + v));
```

### 3. 集合工具类Collections
`java.util.Collections`提供了集合操作的静态方法：
- `sort(List list)`：对List集合排序；
- `reverse(List list)`：反转List集合；
- `shuffle(List list)`：随机打乱List集合；
- `max()/min()`：获取集合的最大值/最小值；
- `synchronizedList()`：将线程不安全的集合转为线程安全的集合；
- `unmodifiableList()`：返回不可修改的集合，防止数据被篡改。

## 十、泛型
Java5+引入，参数化类型，将类型作为参数传递，**编译期检查类型安全**，避免强制类型转换，解决集合的类型安全问题。

### 1. 泛型的核心使用
#### （1）泛型类
在类上定义泛型，实例化时指定具体类型。
```java
// 泛型类，T为类型参数
public class GenericClass<T> {
    private T data;
    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}

// 使用，指定泛型为String
GenericClass<String> gc = new GenericClass<>();
gc.setData("test");
String data = gc.getData(); // 无需强制类型转换
```

#### （2）泛型方法
在方法上独立定义泛型，与类的泛型无关，调用方法时确定类型，支持静态方法。
```java
public class GenericMethod {
    // 泛型方法，<T>声明泛型
    public <T> void print(T t) {
        System.out.println(t);
    }

    // 静态泛型方法
    public static <T> T getInstance(Class<T> clazz) throws Exception {
        return clazz.newInstance();
    }
}
```

#### （3）泛型接口
在接口上定义泛型，实现类可指定具体类型，或继续使用泛型。
```java
// 泛型接口
public interface GenericInterface<T> {
    T get();
}

// 实现类指定具体类型
public class StringImpl implements GenericInterface<String> {
    @Override
    public String get() {
        return "test";
    }
}

// 实现类继续使用泛型
public class GenericImpl<T> implements GenericInterface<T> {
    @Override
    public T get() {
        return null;
    }
}
```

### 2. 泛型通配符
`?`代表不确定的类型，用于接收任意泛型类型，分为3种：
1. **无界通配符 `?`**：可接收任意泛型类型，例：`List<?>`，只能读取元素，不能添加元素（除了null）；
2. **上限通配符 `? extends 类型`**：只能接收该类型及其子类，例：`List<? extends Number>`，可读取Number类型，不能添加元素；
3. **下限通配符 `? super 类型`**：只能接收该类型及其父类，例：`List<? super Number>`，可添加Number及其子类，读取只能用Object接收。

### 3. 泛型擦除
泛型仅在**编译期有效**，运行时会擦除所有泛型信息，泛型参数会被替换为上限类型（无上限则为Object），所以运行时无法获取泛型的具体类型。

## 十一、多线程
进程是操作系统资源分配的最小单位，线程是CPU调度的最小单位，一个进程可包含多个线程，Java天生支持多线程。

### 1. 线程的3种创建方式
#### （1）继承Thread类
继承`Thread`类，重写`run()`方法，调用`start()`启动线程。
```java
public class MyThread extends Thread {
    // 线程执行体
    @Override
    public void run() {
        System.out.println("线程执行：" + Thread.currentThread().getName());
    }
}

// 启动线程
public class Test {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start(); // 启动线程，不能直接调用run()，否则不会开启新线程
    }
}
```

#### （2）实现Runnable接口
实现`Runnable`接口，重写`run()`方法，避免单继承限制，推荐使用。
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("线程执行");
    }
}

// 启动线程
public class Test {
    public static void main(String[] args) {
        new Thread(new MyRunnable()).start();
        // Lambda简化（Runnable是函数式接口）
        new Thread(() -> System.out.println("线程执行")).start();
    }
}
```

#### （3）实现Callable接口
Java5+引入，有返回值，可抛出异常，配合`FutureTask`使用。
```java
public class MyCallable implements Callable<Integer> {
    // 线程执行体，有返回值，可抛出异常
    @Override
    public Integer call() throws Exception {
        System.out.println("线程执行");
        return 100;
    }
}

// 启动线程
public class Test {
    public static void main(String[] args) throws Exception {
        FutureTask<Integer> task = new FutureTask<>(new MyCallable());
        new Thread(task).start();
        // get()会阻塞，直到线程执行完毕，获取返回值
        Integer result = task.get();
        System.out.println(result);
    }
}
```

### 2. 线程的生命周期（6种状态）
`Thread.State`枚举定义了线程的6种状态，状态流转如下：
1. **新建（New）**：线程对象已创建，未调用`start()`；
2. **就绪（Runnable）**：调用`start()`后，等待CPU调度，包含就绪和运行中两个子状态；
3. **阻塞（Blocked）**：线程等待锁资源，无法进入同步代码块，获取锁后回到就绪状态；
4. **等待（Waiting）**：无限期等待，调用`wait()`、`join()`等方法，需其他线程唤醒；
5. **超时等待（Timed Waiting）**：有时间限制的等待，调用`sleep(long)`、`wait(long)`等方法，时间到自动唤醒；
6. **终止（Terminated）**：`run()`方法执行完毕，或抛出未捕获的异常，线程结束，不可再次启动。

### 3. 线程的核心常用方法
| 方法 | 作用 |
|------|------|
| `start()` | 启动线程，进入就绪状态，JVM调用run()方法 |
| `run()` | 线程执行体，定义线程的业务逻辑 |
| `sleep(long millis)` | 静态方法，当前线程休眠指定毫秒，进入超时等待，**不会释放锁** |
| `join()` | 等待该线程执行完毕，当前线程进入等待状态 |
| `yield()` | 静态方法，当前线程让出CPU，回到就绪状态 |
| `interrupt()` | 中断线程，设置中断标记，不是<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2 - o1;
    }
});

// Lambda简化
Collections.sort(list, (o1, o2) -> o2 - o1);
```

### 2. 函数式接口
只有一个抽象方法的接口，用`@FunctionalInterface`注解标记，`java.util.function `setDaemon(boolean on)` | 设置为守护线程（后台线程），必须在start()前调用，所有用户线程结束后，守护线程自动结束 |
| `setPriority(int newPriority)` | 设置线程优先级，范围1~10，默认5，优先级越高，CPU调度概率越高 |
| `currentThread()` | 静态方法，获取当前正在执行的线程对象 |

### 4. 线程同步（解决线程安全问题）
多线程共享资源时，会出现线程安全问题（例：超卖、数据脏读），同步机制保证操作的**原子性、可见性、有序性**。

#### （1）synchronized关键字
Java内置锁，可重入，自动获取和释放锁，保证原子性、可见性、有序性。
- 修饰实例方法：锁当前对象`this`，同一时间只有一个线程能进入该对象的同步方法；
- 修饰静态方法：锁当前类的Class对象，同一时间只有一个线程能进入该类的静态同步方法；
- 修饰代码块：锁括号内的对象，粒度更细，推荐使用。
```java
public class Ticket implements Runnable {
    private int ticket = 100;
    private final Object lock = new Object();

    @Override
    public void run() {
        while (true) {
            // 同步代码块
            synchronized (lock) {
                if (ticket <= 0) break;
                System.out.println(Thread.currentThread().getName() + "卖出第" + ticket + "张票");
                ticket--;
            }
        }
    }
}
```

#### （2）Lock锁
Java5+引入，手动锁，更灵活，支持公平锁、可中断、超时获取锁，核心实现类`ReentrantLock`。
```java
public class Ticket implements Runnable {
    private int ticket = 100;
    // 创建Lock锁，true为公平锁，默认false非公平锁
    private final Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            lock.lock(); // 获取锁
            try {
                if (ticket <= 0) break;
                System.out.println(Thread.currentThread().getName() + "卖出第" + ticket + "张票");
                ticket--;
            } finally {
                lock.unlock(); // 释放锁，必须在finally中，避免死锁
            }
        }
    }
}
```

#### （3）volatile关键字
修饰成员变量，保证**可见性和有序性**，不保证原子性。
- 可见性：一个线程修改了变量值，其他线程能立即看到最新值，禁止变量缓存在工作内存；
- 有序性：禁止指令重排序，通过内存屏障实现；
- 常用场景：单例模式双重检查锁（DCL）、状态标记量。

### 5. 线程池
避免频繁创建销毁线程带来的性能开销，控制线程数量，避免OOM，Java5+位于`java.util.concurrent`包。

#### （1）ThreadPoolExecutor核心参数
手动创建线程池（推荐，阿里规范禁止使用Executors创建），核心构造参数：
```java
public ThreadPoolExecutor(
    int corePoolSize, // 核心线程数，常驻线程
    int maximumPoolSize, // 最大线程数
    long keepAliveTime, // 非核心线程空闲存活时间
    TimeUnit unit, // 时间单位
    BlockingQueue<Runnable> workQueue, // 任务队列，存放等待执行的任务
    ThreadFactory threadFactory, // 线程工厂，创建线程
    RejectedExecutionHandler handler // 拒绝策略，任务满时的处理方式
) {}
```

#### （2）线程池常用方法
- `execute(Runnable command)`：执行无返回值的任务；
- `submit(Callable<T> task)`：执行有返回值的任务，返回`Future<T>`；
- `shutdown()`：关闭线程池，不再接收新任务，等待已提交任务执行完毕；
- `shutdownNow()`：立即关闭线程池，尝试中断正在执行的任务，返回未执行的任务列表。

## 十二、Java8+核心新特性
### 1. Lambda表达式
函数式编程，简化函数式接口的匿名内部类写法，格式：
`(参数列表) -> { 代码体 }`
- 简化规则：参数类型可省略；单个参数可省略括号；代码体只有一行，可省略大括号、return、分号。
```java
// 匿名内部类
Collections.sort(list, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2 - o1;
    }
});

// Lambda简化
Collections.sort(list, (o1, o2) -> o2 - o1);
```

### 2. 函数式接口
只有一个抽象方法的接口，用`@FunctionalInterface`注解标记，`java.util.function`包提供了4大核心内置函数式接口：
| 接口 | 方法 | 作用 |
|------|------|------|
| `Consumer<T>` 消费型接口 | `void accept(T t)` | 接收参数，无返回值 |
| ``包提供了4大核心内置函数式接口：
| 接口 | 方法 | 作用 |
|------|------|------|
| `Consumer<T>` 消费型接口 | `void accept(T t)` | 接收参数，无返回值 |
| `Supplier<T>` 供给型接口 | `T get()` | 无参数，有返回值 |
| `Function<T,R>` 函数Supplier<T>` 供给型接口 | `T get()` | 无参数，有返回值 |
| `Function<T,R>` 函数型接口 | `R apply(T t)` | 接收参数，返回结果 |
| `Predicate<T>` 断言型接口 | `boolean test(T t)` | 接收参数，返回boolean |

### 3. Stream流
用于处理集合/数组，支持链式调用，型接口 | `R apply(T t)` | 接收参数，返回结果 |
| `Predicate<T>` 断言型接口 | `boolean test(T t)` | 接收参数，返回boolean |

### 3. Stream流
用于处理集合/数组，支持链式调用，函数式编程，分为**中间操作**（返回Stream，可链式调用）和**终止操作**（触发执行，返回结果），函数式编程，分为**中间操作**（返回Stream，可链式调用）和**终止操作**（触发执行，返回结果），流只能消费一次。
```java
List<String> list = Arrays.asList("a", "bb", "ccc", "bb", "dd");
List<String> result = list.stream()
        .filter(s -> s.length() > 1) // 过滤，中间操作
流只能消费一次。
```java
List<String> list = Arrays.asList("a", "bb", "ccc", "bb", "dd");
List<String> result = list.stream()
        .filter(s -> s.length() > 1) // 过滤，中间操作
        .map(String::toUpperCase) // 映射，中间操作
        .distinct() // 去重，中间操作
        .s        .map(String::toUpperCase) // 映射，中间操作
        .distinct() // 去重，中间操作
        .sorted() // 排序，中间操作
        .collect(Collectors.toList()); // 收集，终止操作
```

### 4. Optional类
用于解决空指针异常（NPE），是一个可容纳null/非null值的容器，避免频繁的`iforted() // 排序，中间操作
        .collect(Collectors.toList()); // 收集，终止操作
```

### 4. Optional类
用于解决空指针异常（NPE），是一个可容纳null/非null值的容器，避免频繁的`if (obj != null)`判断。
```java
// 传统写法
String name = null;
if (user != null) {
 (obj != null)`判断。
```java
// 传统写法
String name = null;
if (user != null) {
    if (user.getAddress() != null) {
        name = user.getAddress().getCity();
    }
}

// Optional简化
String name = Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
    if (user.getAddress() != null) {
        name = user.getAddress().getCity();
    }
}

// Optional简化
String name = Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
        .orElse("未知");
```

### 5. 全新日期时间API
Java8之前的`Date`、`Calendar        .orElse("未知");
```

### 5. 全新日期时间API
Java8之前的`Date`、`Calendar`是可变、线程不安全的，Java8引入`java.time`包，不可变、线程安全，核心类：
- `LocalDate`：本地日期（年月日）；
- `LocalTime`：本地时间（时分秒）；
- `LocalDateTime`：`是可变、线程不安全的，Java8引入`java.time`包，不可变、线程安全，核心类：
- `LocalDate`：本地日期（年月日）；
- `LocalTime`：本地时间（时分秒）；
- `LocalDateTime`：本地日期时间；
- `Instant`：时间戳；
- `DateTimeFormatter`：日期时间格式化，线程安全。

## 本地日期时间；
- `Instant`：时间戳；
- `DateTimeFormatter`：日期时间格式化，线程安全。

## 十三、反射
Java反射机制允许程序在**运行时**获取类的所有信息，动态创建对象、调用方法、访问属性，无视访问修饰符，是框架的核心底层原理，位于`java.lang.reflect`包。

### 1. 获取Class对象的3十三、反射
Java反射机制允许程序在**运行时**获取类的所有信息，动态创建对象、调用方法、访问属性，无视访问修饰符，是框架的核心底层原理，位于`java.lang.reflect`包。

### 1. 获取Class对象的3种方式
Class对象是反射的入口，每个类加载后都会生成唯一的Class对象。
```java
// 1. 种方式
Class对象是反射的入口，每个类加载后都会生成唯一的Class对象。
```java
// 1. 类名.class，编译期获取
Class<User> clazz1 = User.class;

// 2. 对象.getClass()，运行期获取
User user = new User();
Class<? extends User> clazz2 = user.getClass();

// 3. Class.forName("全类名")，动态加载，最常用
Class<?> clazz3 = Class.forName("com.example.demo.User");
```

### 2.类名.class，编译期获取
Class<User> clazz1 = User.class;

// 2. 对象.getClass()，运行期获取
User user = new User();
Class<? extends User> clazz2 = user.getClass();

// 3. Class.forName("全类名")，动态加载，最常用
Class<?> clazz3 = Class.forName("com.example.demo.User");
```

### 2. 反射的核心操作
#### （1）动态创建对象
```java
 反射的核心操作
#### （1）动态创建对象
```java
Class<?> clazz = Class.forName("com.example.demo.User");
// 调用无参构造创建对象
User user1 = (User) clazz.getDeclaredConstructor().newInstance();
// 调用有参构造创建对象
User user2 = (User) clazz.getDeclaredClass<?> clazz = Class.forName("com.example.demo.User");
// 调用无参构造创建对象
User user1 = (User) clazz.getDeclaredConstructor().newInstance();
// 调用有参构造创建对象
User user2 = (User) clazz.getDeclaredConstructor(String.class, int.class).newInstance("张三", 20);
```

#### （2）访问成员变量
```java
Constructor(String.class, int.class).newInstance("张三", 20);
```

#### （2）访问成员变量
```java
Class<?> clazz = User.class;
User user = (User) clazz.getDeclaredConstructor().newInstance();
// 获取private成员变量
Field usernameField = clazz.getDeclaredField("username");
// 暴力反射，取消访问检查，可访问private成员
usernameField.setAccessibleClass<?> clazz = User.class;
User user = (User) clazz.getDeclaredConstructor().newInstance();
// 获取private成员变量
Field usernameField = clazz.getDeclaredField("username");
// 暴力反射，取消访问检查，可访问private成员
usernameField.setAccessible(true);
// 设置值
usernameField.set(user, "李四");
// 获取值
String username = (String) usernameField.get(user(true);
// 设置值
usernameField.set(user, "李四");
// 获取值
String username = (String) usernameField.get(user);
```

#### （3）调用成员方法
```java
Class<?> clazz = User.class;
User user = (User) clazz.getDeclaredConstructor().newInstance();
// 获取方法，参数为方法名+参数类型
Method showInfoMethod = clazz.getDeclaredMethod("showInfo");
// 调用方法
showInfoMethod.invoke(user);

// 调用带参数的private方法
Method setAgeMethod = clazz.getDeclaredMethod("setAge", int.class);
setAgeMethod.setAccessible(true);
setAgeMethod.invoke(user, 20);
```

## 十四、注解
注解（Annotation）是代码中的特殊标记，可在编译、类加载、运行时被读取，用于配置、标记、生成代码，不影响代码逻辑。

### 1. 元注解
用于修饰注解的注解，位于`java.lang.annotation`包：
- `@Target`：指定注解可修饰的目标元素（类、方法、变量等）；
- `@Retention`：指定注解的保留策略，`SOURCE`（源码）、`CLASS`（字节码，默认）、`RUNTIME`（运行时，自定义注解常用）；
- `@Documented`：标记注解会被javadoc生成文档；
- `@Inherited`：标记注解可被子类继承；
- `@Repeatable`：标记注解可重复修饰同一个元素。

### 2. 自定义注解
```java
// 自定义注解
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    // 注解属性，格式：类型 属性名() default 默认值;
    String value() default ""; // value属性，使用时可省略属性名
    int age() default 18;
    String[] hobbies() default {"读书", "运动"};
}

// 使用注解
@MyAnnotation(value = "测试", age = 20)
public class User {
    @MyAnnotation("方法注解")
    public void showInfo() {}
}
```

### 3. 注解解析
运行时注解通过反射获取注解的属性值：
```java
// 获取类上的注解
Class<User> clazz = User.class;
MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);
if (annotation != null) {
    System.out.println(annotation.value());
    System.out.println(annotation.age());
}

// 获取方法上的注解
Method method = clazz.getDeclaredMethod("showInfo");
MyAnnotation methodAnnotation = method.getAnnotation(MyAnnotation.class);
if (methodAnnotation != null) {
    System.out.println(methodAnnotation.value());
}
```

## 十五、其他核心语法
### 1. 包与import
- `package`：包声明，必须放在源文件第一行，用于分类管理类，解决类名冲突，命名规范为域名倒写；
- `import`：导入其他包的类，`import 全类名`导入单个类，`import 包名.*`导入包下所有类，`import static`静态导入类的静态成员。

### 2. 内部类
定义在一个类内部的类，分为4种：
1. **成员内部类**：类的成员位置，非static修饰，属于外部类对象，可访问外部类所有成员；
2. **静态内部类**：static修饰的成员内部类，属于外部类本身，只能访问外部类的静态成员；
3. **局部内部类**：定义在方法/代码块中的类，作用域仅限当前方法/代码块；
4. **匿名内部类**：没有名字的局部内部类，用于快速创建接口/抽象类的实例，Java8+可用Lambda简化函数式接口的匿名内部类。

### 3. 代码块
用`{}`包裹的代码，分为4种：
1. **局部代码块**：方法中，限制变量作用域，方法调用时执行；
2. **构造代码块**：类中方法外，创建对象时执行，优先于构造方法，每次创建对象都执行；
3. **静态代码块**：static修饰，类加载时仅执行一次，优先于构造代码块，用于初始化静态资源；
4. **同步代码块**：synchronized修饰，用于线程同步。

### 4. 装箱与拆箱
Java5+支持自动装箱拆箱，实现基本类型与对应包装类的自动转换：
- 装箱：基本类型→包装类，例：`Integer i = 100;`，底层调用`Integer.valueOf(100)`；
- 拆箱：包装类→基本类型，例：`int a = i;`，底层调用`i.intValue()`；
> 注意：Integer缓存-128~127之间的数值，该范围内`==`比较地址为true，超出范围为false，包装类比较值必须用`equals()`。


