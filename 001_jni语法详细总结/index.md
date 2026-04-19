# JNI 语法详细总结


# JNI 语法详细总结
JNI（Java Native Interface，Java本地接口）是Java平台提供的标准化编程规范，用于实现**Java代码与C/C++本地代码的双向交互**：Java层可调用Native实现的高性能/系统级逻辑，Native层也可反向访问Java层的类、对象、方法和字段。

## 一、JNI开发完整流程
1. **Java层声明native方法**：用`native`关键字标记本地方法，通过静态代码块加载动态库
2. **生成JNI头文件**：JDK8+使用`javac -h 输出目录 源文件.java`生成头文件（替代旧版javah）
3. **Native层实现函数**：C/C++引入头文件，实现函数逻辑
4. **编译生成动态库**：Linux/Android编译为`.so`，Windows编译为`.dll`，Mac编译为`.jnilib`
5. **Java层调用**：运行Java程序，加载动态库后调用native方法

示例Java基础代码：
```java
public class JniTest {
    // 实例native方法
    public native String sayHello(String name);
    // 静态native方法
    public static native int add(int a, int b);

    // 加载动态库（无需写lib前缀和.so/.dll后缀）
    static {
        System.loadLibrary("test");
    }

    public static void main(String[] args) {
        JniTest test = new JniTest();
        System.out.println(test.sayHello("JNI"));
        System.out.println(add(1, 2));
    }
}
```
生成头文件命令：`javac -h . JniTest.java`，当前目录会生成`JniTest.h`头文件。

## 二、JNI数据类型映射
JNI定义了与Java完全对应的类型体系，分为**基本类型**（值传递）和**引用类型**（引用传递），所有引用类型的基类都是`jobject`。

### 2.1 基本类型映射
| Java基本类型 | JNI类型  | 签名标识 | C/C++对应类型   | 字节数 |
|--------------|----------|----------|-----------------|--------|
| boolean      | jboolean | Z        | unsigned char   | 1      |
| byte         | jbyte    | B        | signed char     | 1      |
| char         | jchar    | C        | unsigned short  | 2      |
| short        | jshort   | S        | short           | 2      |
| int          | jint     | I        | int             | 4      |
| long         | jlong    | J        | long long       | 8      |
| float        | jfloat   | F        | float           | 4      |
| double       | jdouble  | D        | double          | 8      |
| void         | void     | V        | void            | -      |

### 2.2 引用类型映射
| Java引用类型       | JNI类型         | 说明                                   |
|--------------------|-----------------|----------------------------------------|
| java.lang.Object   | jobject         | 所有Java对象的基类                     |
| java.lang.Class    | jclass          | Java类的Class对象                      |
| java.lang.String   | jstring         | Java字符串类型                         |
| java.lang.Throwable| jthrowable      | Java异常/Throwable类型                 |
| Object[]           | jobjectArray    | Java对象数组                           |
| boolean[]          | jbooleanArray   | boolean基本类型数组                    |
| byte[]             | jbyteArray      | byte基本类型数组                       |
| char[]             | jcharArray      | char基本类型数组                       |
| short[]            | jshortArray     | short基本类型数组                      |
| int[]              | jintArray       | int基本类型数组                        |
| long[]             | jlongArray      | long基本类型数组                       |
| float[]            | jfloatArray     | float基本类型数组                      |
| double[]           | jdoubleArray    | double基本类型数组                     |

## 三、JNI函数签名规则
JNI通过**函数签名**唯一标识一个方法，解决Java方法重载问题，签名不匹配会导致方法找不到、程序崩溃。

### 3.1 核心签名规则
1. **基本类型**：使用上表中的单个字符标识（如int→I，boolean→Z）
2. **类对象**：格式为`L+全类名(包名用/分隔)+;`，例：`java.lang.String` → `Ljava/lang/String;`
3. **数组**：格式为`[+元素类型签名`，多维数组每增加一维加一个`[`
   - 示例：int[] → `[I`，String[] → `[Ljava/lang/String;`，int[][] → `[[I`
4. **方法签名**：固定格式为`(参数类型签名列表)+返回值类型签名`，无参数时括号内为空
   - 注意：参数列表按顺序拼接无分隔符；void返回值必须写`V`

### 3.2 签名示例
| Java方法声明                          | JNI方法签名          |
|---------------------------------------|----------------------|
| void test()                           | ()V                  |
| int add(int a, int b)                 | (II)I                |
| String getName(long id)               | (J)Ljava/lang/String;|
| boolean setInfo(String name, int age) | (Ljava/lang/String;I)Z |
| int[] getArray(String[] arr)          | ([Ljava/lang/String;)[I |

### 3.3 快速生成签名
命令行执行`javap -s -p 全类名`，可直接输出类中所有方法/字段的完整签名，避免手写出错。
示例：`javap -s -p JniTest`

## 四、JNI核心结构体：JNIEnv与JavaVM
### 4.1 JNIEnv
- **定义**：JNIEnv是JNI函数表的指针，**线程私有**，每个线程有独立的JNIEnv实例，**绝对不能跨线程使用**
- **核心作用**：Native层通过JNIEnv调用所有JNI API，实现对Java层的所有访问操作
- **获取方式**：
  1. Native方法的第一个参数，JVM自动传入
  2. Native线程中，通过JavaVM的`AttachCurrentThread`获取
- **语法差异**：C语言中使用`(*env)->函数名(env, 参数)`，C++中可简化为`env->函数名(参数)`

### 4.2 JavaVM
- **定义**：JavaVM是Java虚拟机的全局代表，**一个进程只有一个JavaVM实例**，可全局共享、跨线程使用
- **核心作用**：管理虚拟机生命周期，获取/释放线程的JNIEnv，实现Native线程与虚拟机的绑定
- **获取方式**：
  1. JNI_OnLoad函数的第一个参数，加载so库时自动传入
  2. 通过JNIEnv的`GetJavaVM(env, &jvm)`函数获取
- **核心API**：
  | 函数名 | 功能说明 |
  |--------|----------|
  | `AttachCurrentThread(JavaVM *vm, JNIEnv **env, void *args)` | 将Native线程绑定到虚拟机，获取当前线程的JNIEnv |
  | `AttachCurrentThreadAsDaemon(JavaVM *vm, JNIEnv **env, void *args)` | 绑定为守护线程 |
  | `DetachCurrentThread(JavaVM *vm)` | 解绑线程，必须与Attach成对调用，否则线程退出时崩溃 |
  | `DestroyJavaVM(JavaVM *vm)` | 销毁Java虚拟机 |

## 五、Native方法注册方式
Native方法有两种注册方式，用于建立Java native方法与Native函数的映射关系。

### 5.1 静态注册
- **原理**：通过固定的函数命名规则，JVM加载so库时自动完成方法映射
- **函数命名规则**：`Java_+全类名(包名用_分隔)+_+方法名`
  - 特殊转义：类名/方法名中的下划线`_`需转义为`_1`，美元符`$`转义为`_00024`
- **固定参数规则**：
  1. 第一个参数：`JNIEnv *env`，JNI环境指针
  2. 第二个参数：实例方法传入`jobject thiz`（Java当前this对象）；静态方法传入`jclass clazz`（Java当前类的Class对象）
  3. 后续参数：与Java native方法的参数一一对应

示例头文件`JniTest.h`：
```c
#include <jni.h>
#ifndef _Included_JniTest
#define _Included_JniTest
#ifdef __cplusplus
extern "C" {
#endif

// 实例方法sayHello
JNIEXPORT jstring JNICALL Java_JniTest_sayHello(JNIEnv *, jobject, jstring);
// 静态方法add
JNIEXPORT jint JNICALL Java_JniTest_add(JNIEnv *, jclass, jint, jint);

#ifdef __cplusplus
}
#endif
#endif
```

C++实现示例：
```cpp
#include "JniTest.h"

JNIEXPORT jstring JNICALL Java_JniTest_sayHello(JNIEnv *env, jobject thiz, jstring name) {
    const char *c_name = env->GetStringUTFChars(name, NULL);
    char res[128];
    sprintf(res, "Hello, %s!", c_name);
    env->ReleaseStringUTFChars(name, c_name);
    return env->NewStringUTF(res);
}

JNIEXPORT jint JNICALL Java_JniTest_add(JNIEnv *env, jclass clazz, jint a, jint b) {
    return a + b;
}
```

- **优缺点**：
  - 优点：简单直观，工具自动生成声明，不易出错
  - 缺点：函数名冗长，运行时才匹配映射，修改包名/类名需重新生成头文件

### 5.2 动态注册
- **原理**：在`JNI_OnLoad`函数中，通过`RegisterNatives`手动建立方法映射，无需遵循固定命名规则
- **核心结构体**：`JNINativeMethod`，描述方法映射关系
```c
typedef struct {
    const char *name;       // Java native方法名
    const char *signature;  // Java方法的JNI签名
    void *fnPtr;            // Native层对应的函数指针
} JNINativeMethod;
```
- **核心API**：`RegisterNatives(JNIEnv *env, jclass clazz, const JNINativeMethod *methods, jint nMethods)`
  - 参数：env→JNI环境，clazz→目标Java类对象，methods→映射数组，nMethods→映射方法数量
  - 返回值：成功返回0，失败返回负数

完整示例：
```cpp
#include <jni.h>

// Native函数实现，无需遵循静态注册命名规则
jstring native_sayHello(JNIEnv *env, jobject thiz, jstring name) {
    const char *c_name = env->GetStringUTFChars(name, NULL);
    char res[128];
    sprintf(res, "Hello, %s!", c_name);
    env->ReleaseStringUTFChars(name, c_name);
    return env->NewStringUTF(res);
}

jint native_add(JNIEnv *env, jclass clazz, jint a, jint b) {
    return a + b;
}

// 方法映射表
static const JNINativeMethod gMethods[] = {
    {"sayHello", "(Ljava/lang/String;)Ljava/lang/String;", (void*)native_sayHello},
    {"add", "(II)I", (void*)native_add}
};

// so库加载时自动调用
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env = NULL;
    // 获取JNIEnv
    if (vm->GetEnv((void**)&env, JNI_VERSION_1_6) != JNI_OK) {
        return JNI_ERR;
    }
    // 找到目标Java类
    jclass clazz = env->FindClass("JniTest");
    if (clazz == NULL) {
        return JNI_ERR;
    }
    // 注册方法
    int methodNum = sizeof(gMethods) / sizeof(gMethods[0]);
    if (env->RegisterNatives(clazz, gMethods, methodNum) < 0) {
        return JNI_ERR;
    }
    // 返回JNI版本
    return JNI_VERSION_1_6;
}
```

- **优缺点**：
  - 优点：函数名简洁灵活，加载时完成映射，运行效率高，修改包名/类名只需修改映射表，适合大量native方法的项目
  - 缺点：需手动维护映射表，签名写错会直接导致崩溃

## 六、JNI核心API详解
### 6.1 字符串操作（jstring）
jstring不能直接当作C/C++的char*使用，必须通过JNI函数完成转换，且必须成对执行申请/释放操作。

| 函数名 | 功能说明 | 注意事项 |
|--------|----------|----------|
| `jstring NewStringUTF(JNIEnv *env, const char *bytes)` | 用UTF-8编码的C字符串创建jstring | 传入NULL会崩溃 |
| `const char* GetStringUTFChars(JNIEnv *env, jstring str, jboolean *isCopy)` | 将jstring转为UTF-8编码的C字符串 | isCopy标识是否拷贝了Java堆数据；**使用完必须调用ReleaseStringUTFChars释放** |
| `void ReleaseStringUTFChars(JNIEnv *env, jstring str, const char *utf)` | 释放GetStringUTFChars获取的C字符串 | 必须成对调用，否则内存泄漏 |
| `jsize GetStringUTFLength(JNIEnv *env, jstring str)` | 获取UTF-8编码的字符串字节长度 | 对应Java的`String.getBytes("UTF-8").length` |
| `jsize GetStringLength(JNIEnv *env, jstring str)` | 获取Unicode编码的字符串字符长度 | 对应Java的`String.length()` |
| `void GetStringUTFRegion(JNIEnv *env, jstring str, jsize start, jsize len, char *buf)` | 将jstring指定区间的UTF-8字符拷贝到buf | 无需释放，需提前分配buf内存，避免越界 |

### 6.2 数组操作
分为**基本类型数组**和**对象数组**，操作API差异较大。

#### 6.2.1 基本类型数组
以jintArray为例，其他基本类型数组（jbooleanArray、jbyteArray等）API格式完全一致，替换类型前缀即可。

| 函数名 | 功能说明 | 注意事项 |
|--------|----------|----------|
| `<Type>Array New<Type>Array(JNIEnv *env, jsize length)` | 创建指定长度的基本类型数组 | 例：`NewIntArray(env, 10)`创建长度10的int数组 |
| `jsize GetArrayLength(JNIEnv *env, jarray array)` | 获取数组长度 | 通用所有数组类型 |
| `<Type>* Get<Type>ArrayElements(JNIEnv *env, <Type>Array array, jboolean *isCopy)` | 获取数组元素指针 | 用完必须Release释放；isCopy标识是否拷贝数据 |
| `void Release<Type>ArrayElements(JNIEnv *env, <Type>Array array, <Type> *elems, jint mode)` | 释放数组元素指针 | mode：0→拷贝回原数组+释放缓冲区；JNI_COMMIT→拷贝回原数组+不释放；JNI_ABORT→不拷贝+释放 |
| `void Get<Type>ArrayRegion(JNIEnv *env, <Type>Array array, jsize start, jsize len, <Type> *buf)` | 批量拷贝数组元素到buf | 无需释放，推荐小数据量使用 |
| `void Set<Type>ArrayRegion(JNIEnv *env, <Type>Array array, jsize start, jsize len, const <Type> *buf)` | 批量将buf数据写入数组 | 无内存泄漏风险 |
| `void* GetPrimitiveArrayCritical(JNIEnv *env, jarray array, jboolean *isCopy)` | 获取数组临界区指针，尽量避免拷贝 | 临界区内**禁止调用任何其他JNI函数、不能阻塞**，否则会导致虚拟机死锁 |
| `void ReleasePrimitiveArrayCritical(JNIEnv *env, jarray array, void *carray, jint mode)` | 释放临界区指针 | 必须与Get成对调用 |

#### 6.2.2 对象数组
对象数组类型为`jobjectArray`，不支持批量操作，只能单个元素读写。

| 函数名 | 功能说明 |
|--------|----------|
| `jobjectArray NewObjectArray(JNIEnv *env, jsize length, jclass elementClass, jobject initialElement)` | 创建指定长度、指定元素类型的对象数组 | initialElement为元素初始值，通常为NULL |
| `jobject GetObjectArrayElement(JNIEnv *env, jobjectArray array, jsize index)` | 获取数组指定索引的元素 | 返回局部引用，用完可手动释放 |
| `void SetObjectArrayElement(JNIEnv *env, jobjectArray array, jsize index, jobject value)` | 给数组指定索引设置元素 |

### 6.3 类与对象操作
| 函数名 | 功能说明 | 注意事项 |
|--------|----------|----------|
| `jclass FindClass(JNIEnv *env, const char *name)` | 查找Java类，name为全类名（/分隔包名） | 例：`env->FindClass("java/util/ArrayList")`；返回局部引用，需长期使用要转全局引用 |
| `jobject NewObject(JNIEnv *env, jclass clazz, jmethodID methodID, ...)` | 创建Java对象，调用构造方法 | methodID为构造方法的ID，固定名为`<init>`，返回值签名为V |
| `jclass GetObjectClass(JNIEnv *env, jobject obj)` | 获取对象对应的Class对象 | 对应Java的`obj.getClass()` |
| `jboolean IsInstanceOf(JNIEnv *env, jobject obj, jclass clazz)` | 判断对象是否是类的实例 | 对应Java的`instanceof`关键字 |
| `jboolean IsSameObject(JNIEnv *env, jobject ref1, jobject ref2)` | 判断两个引用是否指向同一个Java对象 | 可用于判断弱引用是否被回收（与NULL比较） |

### 6.4 字段与方法访问
Native层可反向访问Java对象的字段、调用Java方法，分为**实例字段/方法**和**静态字段/方法**。

#### 6.4.1 字段访问
| 函数名 | 功能说明 |
|--------|----------|
| `jfieldID GetFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig)` | 获取实例字段的ID | name为字段名，sig为字段类型签名 |
| `<Type> Get<Type>Field(JNIEnv *env, jobject obj, jfieldID fieldID)` | 获取实例字段的值 | Type对应字段类型，如GetIntField、GetObjectField |
| `void Set<Type>Field(JNIEnv *env, jobject obj, jfieldID fieldID, <Type> value)` | 设置实例字段的值 |
| `jfieldID GetStaticFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig)` | 获取静态字段的ID |
| `<Type> GetStatic<Type>Field(JNIEnv *env, jclass clazz, jfieldID fieldID)` | 获取静态字段的值 |
| `void SetStatic<Type>Field(JNIEnv *env, jclass clazz, jfieldID fieldID, <Type> value)` | 设置静态字段的值 |

#### 6.4.2 方法调用
| 函数名 | 功能说明 |
|--------|----------|
| `jmethodID GetMethodID(JNIEnv *env, jclass clazz, const char *name, const char *sig)` | 获取实例方法的ID | 构造方法固定名为`<init>`，返回值签名V |
| `<Type> Call<Type>Method(JNIEnv *env, jobject obj, jmethodID methodID, ...)` | 调用实例方法 | Type对应返回值类型，如CallIntMethod、CallVoidMethod |
| `jmethodID GetStaticMethodID(JNIEnv *env, jclass clazz, const char *name, const char *sig)` | 获取静态方法的ID |
| `<Type> CallStatic<Type>Method(JNIEnv *env, jclass clazz, jmethodID methodID, ...)` | 调用静态方法 |

## 七、JNI引用管理
JNI将引用分为3类，用于管理Java对象的生命周期，防止内存泄漏和对象被GC意外回收。

### 7.1 局部引用（Local Reference）
- **生命周期**：Native方法调用期间有效，方法返回后自动被JVM释放，也可手动提前释放
- **创建方式**：绝大多数JNI函数返回的引用都是局部引用（如FindClass、NewObject等）
- **核心特性**：线程私有，**不能跨线程、跨Native方法传递使用**；占用局部引用表（Android默认512个），大量创建会导致溢出崩溃
- **手动管理API**：
  - `void DeleteLocalRef(JNIEnv *env, jobject ref)`：手动释放单个局部引用
  - `jint PushLocalFrame(JNIEnv *env, jint capacity)`：创建局部引用帧，预分配引用空间
  - `jobject PopLocalFrame(JNIEnv *env, jobject result)`：弹出当前帧，释放帧内所有局部引用，仅保留result返回
- **最佳实践**：循环内创建的局部引用用完立即释放；大量局部引用使用Push/PopLocalFrame批量管理

### 7.2 全局引用（Global Reference）
- **生命周期**：手动创建后，直到手动释放前一直有效，不会被GC回收
- **创建/释放**：`jobject NewGlobalRef(JNIEnv *env, jobject obj)` / `void DeleteGlobalRef(JNIEnv *env, jobject ref)`
- **核心特性**：进程全局有效，**可跨线程、跨方法传递使用**；不会自动释放，必须手动Delete，否则会造成永久内存泄漏
- **典型场景**：缓存jclass、Java回调对象等需要长期跨线程使用的对象

### 7.3 弱全局引用（Weak Global Reference）
- **生命周期**：手动创建后，直到手动释放前有效，但**不会阻止GC回收**，GC运行时若该对象无其他强引用，会被回收
- **创建/释放**：`jweak NewWeakGlobalRef(JNIEnv *env, jobject obj)` / `void DeleteWeakGlobalRef(JNIEnv *env, jweak ref)`
- **核心特性**：可跨线程使用，避免内存泄漏，适合缓存非必需的对象，防止类加载器泄漏
- **有效性判断**：`IsSameObject(env, weakRef, NULL)`，返回JNI_TRUE表示已被回收，不可再使用

## 八、JNI异常处理
JNI的异常机制与Java不同：**JNI函数抛出异常后，不会立即中断Native代码的执行，必须手动检查、处理、清除异常**，否则后续JNI调用会导致程序崩溃。

### 8.1 核心异常处理API
| 函数名 | 功能说明 |
|--------|----------|
| `jboolean ExceptionCheck(JNIEnv *env)` | 检查当前是否有挂起的异常，返回JNI_TRUE表示有异常 | 轻量高效，推荐优先使用 |
| `jthrowable ExceptionOccurred(JNIEnv *env)` | 获取当前挂起的异常对象，无异常返回NULL | 需要获取异常详情时使用 |
| `void ExceptionDescribe(JNIEnv *env)` | 打印异常堆栈信息，用于调试 |
| `void ExceptionClear(JNIEnv *env)` | 清除当前挂起的异常 | 异常处理后必须清除，否则后续JNI调用会崩溃 |
| `jint ThrowNew(JNIEnv *env, jclass clazz, const char *message)` | 向Java层抛出指定类型、指定信息的异常 | 最常用的抛异常方式 |

### 8.2 异常处理规范
1. 调用JNI函数后，必须及时检查异常，尤其是FindClass、GetMethodID等高频出错的函数
2. 存在挂起异常时，**只能调用异常处理相关的JNI函数**，禁止调用其他JNI函数，否则行为未定义
3. 异常处理完成后，必须调用`ExceptionClear`清除异常，才能继续执行其他逻辑
4. Native层抛出的异常，Java层可通过try-catch捕获

示例：
```cpp
void callJavaMethodSafe(JNIEnv *env, jobject obj) {
    jclass clazz = env->GetObjectClass(obj);
    jmethodID methodID = env->GetMethodID(clazz, "test", "()V");

    // 检查异常
    if (env->ExceptionCheck()) {
        env->ExceptionDescribe(); // 打印堆栈
        env->ExceptionClear(); // 清除异常
        return;
    }

    // 无异常才调用方法
    env->CallVoidMethod(obj, methodID);
}
```

## 九、JNI线程相关
Native线程（如pthread创建的线程）要访问Java层，必须先与JavaVM绑定，否则无法获取有效的JNIEnv，也无法安全访问Java对象。

### 9.1 核心规则
1. **JavaVM全局唯一**，可跨线程共享，建议在JNI_OnLoad中缓存全局指针
2. **JNIEnv线程私有**，不能跨线程传递使用，每个线程必须通过Attach获取专属的JNIEnv
3. **Attach与Detach必须成对调用**：Native线程退出前必须Detach，否则虚拟机无法释放线程资源，导致崩溃
4. Java主线程/Java创建的线程，无需手动Attach/Detach，JVM自动管理
5. 局部引用不能跨线程传递，跨线程共享Java对象必须使用全局引用

### 9.2 完整示例
```cpp
#include <pthread.h>
#include <jni.h>

// 全局缓存JavaVM
JavaVM *g_jvm = NULL;
// 全局引用，缓存Java回调对象
jobject g_callback_obj = NULL;

// JNI_OnLoad中缓存JavaVM
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
    g_jvm = vm;
    JNIEnv *env = NULL;
    if (vm->GetEnv((void**)&env, JNI_VERSION_1_6) != JNI_OK) {
        return JNI_ERR;
    }
    return JNI_VERSION_1_6;
}

// 线程执行函数
void* thread_func(void *arg) {
    JNIEnv *env = NULL;
    // 绑定线程到虚拟机
    if (g_jvm->AttachCurrentThread(&env, NULL) != JNI_OK) {
        return NULL;
    }

    // 执行JNI操作，调用Java回调
    if (g_callback_obj != NULL) {
        jclass clazz = env->GetObjectClass(g_callback_obj);
        jmethodID callbackID = env->GetMethodID(clazz, "onNativeCallback", "(I)V");
        if (callbackID != NULL && !env->ExceptionCheck()) {
            env->CallVoidMethod(g_callback_obj, callbackID, 100);
        }
    }

    // 解绑线程，必须调用
    g_jvm->DetachCurrentThread();
    return NULL;
}

// Java层调用，启动Native线程
JNIEXPORT void JNICALL Java_JniTest_startThread(JNIEnv *env, jobject thiz, jobject callback) {
    // 保存回调对象为全局引用
    if (g_callback_obj != NULL) {
        env->DeleteGlobalRef(g_callback_obj);
    }
    g_callback_obj = env->NewGlobalRef(callback);

    // 创建线程
    pthread_t tid;
    pthread_create(&tid, NULL, thread_func, NULL);
}
```

## 十、常见坑与最佳实践
1. **JNIEnv跨线程使用**：绝对禁止跨线程传递JNIEnv，必须通过JavaVM的AttachCurrentThread获取当前线程的JNIEnv
2. **内存泄漏**：GetStringUTFChars、Get<Type>ArrayElements必须与Release成对调用；全局引用必须手动Delete；循环内局部引用用完立即释放
3. **异常未处理**：JNI函数抛出异常后，未检查、未清除就继续调用其他JNI函数，导致崩溃
4. **签名错误**：方法/字段签名手写错误，推荐用`javap -s`自动生成签名
5. **类名错误**：FindClass的类名必须用`/`分隔包名，而非`.`
6. **引用滥用**：局部引用跨线程/跨方法使用，导致野指针崩溃；全局引用未释放导致永久内存泄漏
7. **临界区滥用**：GetPrimitiveArrayCritical临界区内调用其他JNI函数、阻塞，导致虚拟机死锁
8. **Native线程未Detach**：Attach后未Detach，线程退出时导致虚拟机崩溃
9. **缓存错误**：FindClass返回的是局部引用，长期缓存必须转为全局引用；jmethodID/jfieldID不是引用类型，无需转全局引用，只要类不被卸载就长期有效


