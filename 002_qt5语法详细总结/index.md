# Qt5 语法详细总结


# Qt5 语法详细总结
Qt5 是跨平台 C++ GUI 开发框架，核心基于**元对象系统(MOC)** 扩展标准C++，核心特性为信号与槽机制，同时提供了完整的UI、事件、IO、网络、多线程等开发能力。以下是从基础到进阶的全语法体系总结。

## 一、核心根基：元对象系统（MOC）
元对象系统是Qt区别于标准C++的核心，为Qt提供了信号槽、属性系统、反射、类型安全等能力，由三大核心要素构成。

### 1.1 核心三要素
1.  **基类 `QObject`**：所有启用元对象系统的类的最终基类，提供父子对象内存自动管理、事件循环基础、信号槽通信载体等能力。
2.  **`Q_OBJECT` 宏**：必须放在类声明的私有区域（Qt5支持公有区域），启用元对象特性，MOC会解析该宏生成元对象代码。
3.  **元对象编译器(MOC)**：Qt的预处理工具，在C++编译前解析头文件中的`Q_OBJECT`宏，生成`moc_xxx.cpp`补充代码，实现标准C++不支持的元对象能力。

### 1.2 基础语法与规范
```cpp
// 头文件 mywidget.h（类声明必须放在头文件，MOC仅解析头文件）
#ifndef MYWIDGET_H
#define MYWIDGET_H

#include <QWidget>

// 必须继承QObject或其子类（QWidget是QObject的子类）
class MyWidget : public QWidget
{
    // 必须添加此宏，启用元对象系统
    Q_OBJECT

public:
    explicit MyWidget(QWidget *parent = nullptr);
    ~MyWidget();

// 信号声明区
signals:
    void mySignal(int value);

// 槽函数声明区
public slots:
    void mySlot(int value);
};

#endif // MYWIDGET_H
```

### 1.3 关键注意事项
- 类继承QObject时，**多继承的第一个父类必须是QObject**，否则元对象系统失效。
- 新增/删除`Q_OBJECT`宏后，必须重新执行`qmake`，否则会报`undefined reference to vtable`错误。
- 继承QObject的类禁止拷贝构造和赋值，QObject禁用了拷贝语义。

## 二、核心通信机制：信号与槽（Signals & Slots）
信号与槽是Qt的核心对象间通信机制，替代了传统的回调函数，类型安全、松耦合、支持跨线程、支持一对多/多对一连接。

### 2.1 核心定义
- **信号(Signals)**：对象状态变化时发出的通知，**只有声明，无需实现**，返回值必须为`void`，可携带自定义参数，默认访问权限为`protected`。
- **槽函数(Slots)**：接收信号的处理函数，是普通的C++成员函数，可被直接调用，也可被信号触发，返回值必须为`void`，参数需与对应信号匹配，支持`public/protected/private`访问权限。

### 2.2 两种连接语法
#### 1. Qt5 推荐：函数指针语法（编译期强类型检查）
**语法格式**
```cpp
QObject::connect(
    const QObject *sender,      // 信号发送者
    &SenderType::signal,         // 信号函数地址
    const QObject *receiver,    // 信号接收者
    &ReceiverType::slot,         // 槽函数地址
    Qt::ConnectionType type = Qt::AutoConnection  // 连接类型
);
```

**代码示例**
```cpp
MyWidget *w = new MyWidget;
QPushButton *btn = new QPushButton("点击", w);

// 基础连接：按钮点击信号 连接到 窗口槽函数
QObject::connect(btn, &QPushButton::clicked, w, &MyWidget::mySlot);

// 连接Lambda表达式（Qt5核心优势）
QObject::connect(btn, &QPushButton::clicked, w, [=](){
    qDebug() << "按钮被点击";
});

// 信号连接信号（信号转发）
QObject::connect(btn, &QPushButton::clicked, w, &MyWidget::mySignal);
```

#### 2. Qt4 兼容：宏语法（运行期检查，不推荐）
**语法格式**
```cpp
QObject::connect(sender, SIGNAL(signal(参数类型列表)), receiver, SLOT(slot(参数类型列表)));
```

**代码示例**
```cpp
// 仅做兼容，不推荐使用，编译期不检查参数匹配，易出现运行期错误
QObject::connect(btn, SIGNAL(clicked()), w, SLOT(mySlot()));
```

### 2.3 连接类型 `Qt::ConnectionType`
| 类型 | 作用 | 适用场景 |
|------|------|----------|
| `Qt::AutoConnection` | 默认值，自动判断：同线程用直连，跨线程用队列连接 | 绝大多数通用场景 |
| `Qt::DirectConnection` | 信号发出时立即调用槽函数，槽在发送者线程执行 | 同线程同步执行，需谨慎跨线程使用 |
| `Qt::QueuedConnection` | 信号放入接收者线程的事件队列，等待事件循环处理，槽在接收者线程执行 | 跨线程通信，线程安全 |
| `Qt::BlockingQueuedConnection` | 信号发出后发送者线程阻塞，直到槽执行完毕 | 跨线程同步操作，禁止同线程使用（会死锁） |

### 2.4 断开连接语法
```cpp
// 1. 断开发送者所有信号的所有连接
disconnect(sender, nullptr, nullptr, nullptr);

// 2. 断开发送者特定信号的所有连接
disconnect(sender, &Sender::signal, nullptr, nullptr);

// 3. 断开发送者和接收者的所有连接
disconnect(sender, nullptr, receiver, nullptr);

// 4. 断开特定的信号槽连接
disconnect(sender, &Sender::signal, receiver, &Receiver::slot);
```

### 2.5 核心特性与注意事项
- 一个信号可以连接多个槽，槽按连接顺序执行；多个信号可以连接同一个槽。
- 信号参数个数可多于槽，槽会忽略多余的参数，**参数类型必须完全匹配，顺序必须一致**。
- 自定义类型作为信号槽参数时，必须用`Q_DECLARE_METATYPE`注册元类型。
- 槽函数的访问权限不影响连接，即使是`private`槽，也可被外部信号触发。
- 发送者/接收者对象销毁时，连接会自动断开，无需手动处理。

## 三、项目构建与模块系统
Qt5 基于`qmake`构建项目，核心配置文件为`.pro`项目文件，同时对模块做了精细化拆分。

### 3.1 .pro 文件核心语法
```pro
# 1. 模块引入（Qt5默认包含core、gui模块，widgets需手动添加）
QT += core gui widgets  # 基础桌面项目必选
QT += network           # 新增网络模块
QT += sql               # 新增数据库模块
QT -= gui               # 移除默认模块（控制台程序）

# 2. 源文件与资源声明
SOURCES += main.cpp \
           mywidget.cpp
HEADERS += mywidget.h
FORMS += mywidget.ui    # Qt Designer UI文件
RESOURCES += res.qrc    # 资源文件

# 3. 目标配置
TARGET = MyQtApp        # 生成的可执行文件名
TEMPLATE = app          # 项目模板：app=应用程序，lib=库，subdirs=子项目

# 4. 编译配置
CONFIG += c++17         # 指定C++标准
CONFIG += debug         # debug模式，release为发布模式

# 5. 平台专属配置
win32 {
    DEFINES += WIN32_PLATFORM  # Windows平台宏定义
}
unix {
    DEFINES += UNIX_PLATFORM   # Linux/macOS平台宏定义
}
```

### 3.2 Qt5 核心模块说明
| 模块名 | 功能说明 | 引入方式 |
|--------|----------|----------|
| core | 核心非GUI功能：元对象、事件循环、容器、线程、文件IO、定时器 | 默认包含 |
| gui | GUI核心基础：图形、字体、窗口、事件、OpenGL | 默认包含 |
| widgets | 桌面UI控件：QWidget、QPushButton、QLabel等所有标准控件 | `QT += widgets` |
| network | 网络编程：TCP/UDP/HTTP/WebSocket | `QT += network` |
| sql | 数据库操作：MySQL、SQLite、ODBC等 | `QT += sql` |
| multimedia | 音视频播放、采集 | `QT += multimedia` |
| qml/quick | QML/Qt Quick 动态UI开发 | `QT += qml quick` |
| printsupport | 打印相关功能 | `QT += printsupport` |

### 3.3 Qt5 程序入口标准写法
```cpp
#include "mywidget.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    // QApplication是GUI应用的核心，管理事件循环、UI资源
    QApplication a(argc, argv);

    // 创建主窗口
    MyWidget w;
    // 显示窗口
    w.show();

    // 启动应用事件循环，阻塞直到程序退出
    return a.exec();
}
```

## 四、核心基础类与常用语法
### 4.1 字符串类 `QString`
Qt核心字符串类，基于Unicode编码，采用**隐式共享（写时复制）** 机制，比`std::string`功能更丰富，是Qt开发最常用的类。

```cpp
// 1. 初始化
QString str1 = "Hello Qt5";
QString str2("Hello World");
QString str3 = QString::number(100);  // 数字转字符串
// 格式化字符串（Qt5推荐，安全无溢出）
QString str4 = QString("姓名：%1，年龄：%2").arg("张三").arg(20);
// C风格格式化（兼容，不推荐）
QString str5 = QString::asprintf("年龄：%d", 20);

// 2. 常用操作
str1.append("!");               // 追加字符串
str1.prepend("Hi ");            // 前置字符串
str1.contains("Qt5");           // 判断是否包含子串，返回bool
str1.isEmpty();                 // 判断是否为空字符串（长度为0）
str1.isNull();                  // 判断是否为null（未初始化）
str1.trimmed();                 // 去除首尾空白字符
str1.replace("Qt5", "Qt 5.15");// 替换子串
str1.split(" ");                // 按分隔符拆分，返回QStringList
str1.toInt();                   // 转int，配套toDouble()/toLongLong()
str1.toUpper();                 // 转大写，toLower()转小写
```

### 4.2 容器类
Qt提供了兼容STL、隐式共享、只读线程安全的容器类，分为顺序容器和关联容器。

#### 顺序容器
```cpp
// 1. QList<T>：Qt5最常用顺序容器，动态数组，Qt5.15后与QVector合并
QList<int> list;
list << 1 << 2 << 3;               // 流式添加元素
list.append(4);                     // 尾部追加
list.prepend(0);                    // 头部插入
list.insert(2, 10);                 // 索引2位置插入10

// 元素访问
int val = list[0];
int first = list.first();
int last = list.last();

// 遍历
for(int i : list) { qDebug() << i; }
for(int i=0; i<list.size(); i++) { qDebug() << list[i]; }

// 其他操作
list.removeAt(0);                   // 移除指定索引元素
list.contains(2);                   // 判断是否包含元素
list.clear();                       // 清空容器

// 2. QStringList：QList<QString>别名，字符串专用容器
QStringList strList;
strList << "a" << "b" << "c";
strList.join(",");                  // 拼接为"a,b,c"
strList.filter("a");                // 筛选包含指定子串的元素
```

#### 关联容器
```cpp
// 1. QMap<Key, T>：有序键值对，红黑树实现，Key需重载<运算符
QMap<QString, int> map;
map["张三"] = 20;
map.insert("李四", 25);

// 元素访问
int age = map["张三"];
int age2 = map.value("李四", 0);    // 不存在则返回默认值0

// 遍历
QMap<QString, int>::iterator it;
for(it = map.begin(); it != map.end(); it++){
    qDebug() << it.key() << it.value();
}

// 2. QHash<Key, T>：无序哈希表，查找速度优于QMap，Key需重载==和qHash()
QHash<QString, int> hash;
hash["王五"] = 30;
// 操作API与QMap基本一致
```

### 4.3 智能指针类
Qt提供了完善的智能指针，配合父子对象机制，避免内存泄漏。
```cpp
// 1. QScopedPointer<T>：作用域独占指针，离开作用域自动释放，不可拷贝
QScopedPointer<QPushButton> btn(new QPushButton("测试"));
btn->setText("Hello");  // 正常使用指针，无需手动delete

// 2. QSharedPointer<T>：共享指针，引用计数，引用为0自动释放
QSharedPointer<QPushButton> btn1(new QPushButton("btn1"));
QSharedPointer<QPushButton> btn2 = btn1;  // 引用计数+1

// 3. QWeakPointer<T>：弱引用，配合QSharedPointer使用，解决循环引用
QWeakPointer<QPushButton> weakBtn = btn1;
```

### 4.4 调试输出 `qDebug()`
Qt专用调试输出，支持Qt各类原生类型，比`printf/cout`更便捷。
```cpp
#include <QDebug>

// 基础输出
qDebug() << "Hello Qt5" << 123 << QString("测试");

// 容器输出
QList<int> list << 1 << 2 << 3;
qDebug() << "列表内容：" << list;

// C风格格式化输出
qDebug("姓名：%s，年龄：%d", "张三", 20);
```

## 五、UI界面开发核心语法
Qt5 桌面UI开发基于`Qt Widgets`模块，核心基类为`QWidget`，所有控件和窗口均继承自该类。

### 5.1 QWidget 窗口基础语法
```cpp
// 创建独立窗口
QWidget *window = new QWidget;
window->setWindowTitle("Qt5 窗口");  // 设置窗口标题
window->resize(800, 600);            // 设置窗口大小
window->setFixedSize(800, 600);      // 固定窗口大小，禁止缩放
window->move(100, 100);               // 移动窗口到屏幕坐标(100,100)
window->show();                        // 显示窗口
// window->showMaximized();           // 最大化显示
// window->showFullScreen();          // 全屏显示
// window->hide();                    // 隐藏窗口

// 创建子控件，指定父窗口（父窗口销毁时自动释放子控件）
QPushButton *btn = new QPushButton("按钮", window);
btn->setGeometry(100, 100, 200, 50); // 设置控件位置和大小(x,y,宽,高)
```

### 5.2 布局管理 Layout
Qt推荐使用布局管理替代固定坐标，实现窗口缩放自适应，四大核心布局如下。

#### 核心布局类与语法
| 布局类 | 功能 |
|--------|------|
| `QHBoxLayout` | 水平布局，控件从左到右排列 |
| `QVBoxLayout` | 垂直布局，控件从上到下排列 |
| `QGridLayout` | 网格布局，按行列排列控件 |
| `QFormLayout` | 表单布局，标签+输入框两列布局 |

**代码示例**
```cpp
QWidget *window = new QWidget;
window->setWindowTitle("布局示例");
window->resize(400, 300);

// 1. 创建主垂直布局，绑定到窗口
QVBoxLayout *mainLayout = new QVBoxLayout(window);
// 设置布局边距和控件间距
mainLayout->setContentsMargins(20, 20, 20, 20);
mainLayout->setSpacing(10);

// 2. 创建水平布局，存放标签和输入框
QHBoxLayout *hLayout = new QHBoxLayout;
QLabel *label = new QLabel("用户名：");
QLineEdit *edit = new QLineEdit;
hLayout->addWidget(label);
hLayout->addWidget(edit);

// 3. 添加控件和子布局到主布局
QPushButton *btn = new QPushButton("登录");
mainLayout->addLayout(hLayout);
mainLayout->addWidget(btn);
mainLayout->addStretch();  // 添加伸缩项，将控件顶到上方

window->show();
```

### 5.3 常用标准控件核心语法
| 控件类 | 核心功能 | 核心信号 | 核心方法 |
|--------|----------|----------|----------|
| `QPushButton` | 按钮控件 | `clicked()` 点击、`pressed()` 按下 | `setText()` 设置文本、`setIcon()` 设置图标 |
| `QLabel` | 文本/图片显示标签 | 无默认常用信号 | `setText()` 设置文本、`setPixmap()` 设置图片、`setAlignment()` 设置对齐 |
| `QLineEdit` | 单行输入框 | `textChanged()` 文本变化、`editingFinished()` 编辑完成 | `text()` 获取文本、`setPlaceholderText()` 设置占位符、`setEchoMode()` 设置密码模式 |
| `QTextEdit` | 多行富文本输入框 | `textChanged()` 文本变化 | `toPlainText()` 获取纯文本、`append()` 追加文本 |
| `QCheckBox` | 复选框 | `stateChanged()` 状态变化 | `isChecked()` 判断是否选中、`setChecked()` 设置选中状态 |
| `QRadioButton` | 单选框 | `toggled()` 状态切换 | `isChecked()` 判断是否选中 |
| `QComboBox` | 下拉选择框 | `currentIndexChanged()` 选中项变化 | `addItem()` 添加选项、`currentText()` 获取当前选中文本 |
| `QSlider` | 滑块控件 | `valueChanged()` 值变化 | `setValue()` 设置值、`setRange()` 设置取值范围 |

### 5.4 标准对话框核心语法
Qt提供了开箱即用的标准对话框，均通过静态方法调用，无需手动创建实例。
```cpp
// 1. 消息对话框 QMessageBox
QMessageBox::information(window, "提示", "操作成功");  // 信息提示
QMessageBox::warning(window, "警告", "操作有误");      // 警告提示
QMessageBox::critical(window, "错误", "致命错误");     // 错误提示
// 询问确认框
int ret = QMessageBox::question(window, "确认", "是否退出？",
                                 QMessageBox::Yes | QMessageBox::No,
                                 QMessageBox::No);
if(ret == QMessageBox::Yes) { window->close(); }

// 2. 文件对话框 QFileDialog
// 打开单个文件
QString filePath = QFileDialog::getOpenFileName(window,
    "打开文件", "./", "文本文件(*.txt);;所有文件(*.*)");
// 保存文件
QString savePath = QFileDialog::getSaveFileName(window,
    "保存文件", "./", "文本文件(*.txt)");
// 选择文件夹
QString dirPath = QFileDialog::getExistingDirectory(window, "选择文件夹", "./");

// 3. 输入对话框 QInputDialog
bool ok;
// 文本输入
QString text = QInputDialog::getText(window, "输入", "请输入用户名：",
                                      QLineEdit::Normal, "默认值", &ok);
// 数字输入
int num = QInputDialog::getInt(window, "输入数字", "请输入年龄：",
                                20, 0, 100, 1, &ok);
```

## 六、事件系统与事件处理
Qt的所有交互（鼠标、键盘、绘制、窗口变化）均基于事件系统，事件由`QCoreApplication`的事件循环分发，提供了4种层级的事件处理方式。

### 6.1 重写专用事件处理函数（最常用）
QObject提供了一系列虚函数，对应不同的事件类型，子类重写即可处理对应事件，是最常用的事件处理方式。

#### 常用事件处理函数
```cpp
// 鼠标事件
void mousePressEvent(QMouseEvent *event) override;    // 鼠标按下
void mouseReleaseEvent(QMouseEvent *event) override;  // 鼠标释放
void mouseMoveEvent(QMouseEvent *event) override;     // 鼠标移动
void wheelEvent(QWheelEvent *event) override;         // 鼠标滚轮

// 键盘事件
void keyPressEvent(QKeyEvent *event) override;        // 按键按下
void keyReleaseEvent(QKeyEvent *event) override;      // 按键释放

// 绘制事件（必须在此函数内执行绘制操作）
void paintEvent(QPaintEvent *event) override;

// 窗口事件
void resizeEvent(QResizeEvent *event) override;       // 窗口大小变化
void closeEvent(QCloseEvent *event) override;         // 窗口关闭

// 焦点事件
void focusInEvent(QFocusEvent *event) override;       // 获得焦点
void focusOutEvent(QFocusEvent *event) override;     // 失去焦点
```

#### 代码示例
```cpp
// 1. 鼠标按下事件处理
void MyWidget::mousePressEvent(QMouseEvent *event)
{
    if(event->button() == Qt::LeftButton) {
        qDebug() << "左键按下，窗口坐标：" << event->pos();
        qDebug() << "屏幕全局坐标：" << event->globalPos();
    } else if(event->button() == Qt::RightButton) {
        qDebug() << "右键按下";
    }
    // 调用父类实现，保证事件继续传递
    QWidget::mousePressEvent(event);
}

// 2. 键盘按下事件处理
void MyWidget::keyPressEvent(QKeyEvent *event)
{
    // 单按键判断
    if(event->key() == Qt::Key_Escape) {
        this->close();
    }
    // 组合键判断
    if(event->modifiers() == Qt::ControlModifier && event->key() == Qt::Key_S) {
        qDebug() << "按下Ctrl+S，执行保存";
    }
    QWidget::keyPressEvent(event);
}

// 3. 绘制事件处理（QPainter必须在paintEvent内创建）
void MyWidget::paintEvent(QPaintEvent *event)
{
    QPainter painter(this);
    // 设置画笔
    QPen pen(Qt::red, 2, Qt::SolidLine);
    painter.setPen(pen);

    // 基础绘制
    painter.drawLine(0, 0, 100, 100);          // 绘制直线
    painter.drawRect(100, 100, 200, 150);     // 绘制矩形
    painter.drawEllipse(200, 200, 100, 100);  // 绘制圆形
    painter.drawText(300, 300, "Hello Qt5");   // 绘制文本
}
```

### 6.2 重写`event()`函数
`event()`是所有事件的入口函数，返回`true`表示事件已处理，不再向下分发，用于拦截事件或处理无专用处理函数的事件。
```cpp
bool MyWidget::event(QEvent *event)
{
    // 拦截鼠标按下事件
    if(event->type() == QEvent::MouseButtonPress) {
        qDebug() << "event()拦截到鼠标按下事件";
        // return true; // 返回true，事件不再传递给mousePressEvent
    }
    // 其他事件交给父类处理
    return QWidget::event(event);
}
```

### 6.3 事件过滤器 Event Filter
给目标对象安装事件过滤器，无需子类化即可拦截处理对象的事件，灵活处理多个对象的事件。
1.  过滤器类必须继承QObject，重写`eventFilter()`函数
2.  目标对象调用`installEventFilter()`安装过滤器
3.  `eventFilter()`返回`true`表示拦截事件，不再传递给目标对象

```cpp
// 头文件声明
class MyWidget : public QWidget
{
    Q_OBJECT
protected:
    bool eventFilter(QObject *watched, QEvent *event) override;
private:
    QPushButton *m_btn;
};

// 实现
MyWidget::MyWidget(QWidget *parent) : QWidget(parent)
{
    m_btn = new QPushButton("测试按钮", this);
    // 给按钮安装事件过滤器，this作为过滤器对象
    m_btn->installEventFilter(this);
}

bool MyWidget::eventFilter(QObject *watched, QEvent *event)
{
    // 判断目标对象
    if(watched == m_btn) {
        // 拦截鼠标按下事件
        if(event->type() == QEvent::MouseButtonPress) {
            qDebug() << "事件过滤器拦截到按钮按下";
            // return true; // 拦截事件，按钮不会收到按下事件
        }
    }
    // 其他事件交给父类处理
    return QWidget::eventFilter(watched, event);
}
```

### 6.4 事件传递顺序
`QApplication`全局事件过滤器 → 目标对象的事件过滤器 → 目标对象的`event()`函数 → 目标对象的专用事件处理函数

## 七、模型/视图（Model/View）架构语法
Qt5的MVC架构变体，将**数据(Model)** 和**显示(View)** 完全分离，通过**委托(Delegate)** 处理单元格渲染和编辑，适合大数据量、复杂列表/表格/树形结构的展示。

### 7.1 核心类
| 分类 | 核心类 | 说明 |
|------|--------|------|
| 模型基类 | `QAbstractItemModel` | 所有模型的抽象基类，需子类化实现核心接口 |
| 标准模型 | `QStringListModel` | 字符串列表专用模型 |
| 标准模型 | `QStandardItemModel` | 通用数据模型，支持表格/树形结构 |
| 标准模型 | `QSqlTableModel` | 数据库表专用模型 |
| 视图基类 | `QAbstractItemView` | 所有视图的抽象基类 |
| 标准视图 | `QListView` | 列表视图 |
| 标准视图 | `QTableView` | 表格视图 |
| 标准视图 | `QTreeView` | 树形视图 |
| 委托 | `QStyledItemDelegate` | 自定义单元格渲染和编辑的基类 |

### 7.2 代码示例
#### 1. QStringListModel + QListView
```cpp
// 1. 创建数据模型
QStringListModel *model = new QStringListModel;
QStringList list;
list << "苹果" << "香蕉" << "橙子" << "葡萄";
model->setStringList(list);

// 2. 创建视图，绑定模型
QListView *view = new QListView;
view->setModel(model);
view->resize(300, 200);
view->show();
```

#### 2. QStandardItemModel + QTableView
```cpp
// 1. 创建表格模型，4行3列
QStandardItemModel *model = new QStandardItemModel(4, 3);
// 设置表头
model->setHeaderData(0, Qt::Horizontal, "姓名");
model->setHeaderData(1, Qt::Horizontal, "年龄");
model->setHeaderData(2, Qt::Horizontal, "性别");

// 2. 填充数据
model->setItem(0, 0, new QStandardItem("张三"));
model->setItem(0, 1, new QStandardItem("20"));
model->setItem(0, 2, new QStandardItem("男"));
model->setItem(1, 0, new QStandardItem("李四"));
model->setItem(1, 1, new QStandardItem("25"));
model->setItem(1, 2, new QStandardItem("女"));

// 3. 创建表格视图，绑定模型
QTableView *view = new QTableView;
view->setModel(model);
view->resize(500, 300);
// 表格属性设置
view->horizontalHeader()->setStretchLastSection(true);  // 最后一列拉伸
view->setEditTriggers(QAbstractItemView::DoubleClicked); // 双击编辑
view->setSelectionBehavior(QAbstractItemView::SelectRows); // 选中整行
view->show();
```

## 八、多线程编程核心语法
Qt5多线程核心类为`QThread`，基于事件循环驱动，**核心原则：UI操作只能在主线程（GUI线程）执行，子线程绝对禁止操作UI控件**，线程间通信推荐使用信号槽。

Qt5官方推荐两种多线程实现方式：

### 8.1 方式一：继承QThread，重写run()函数
适合简单的循环耗时任务，`run()`是线程的入口函数，`run()`执行完毕线程结束，默认无事件循环，需手动调用`exec()`启动。

```cpp
// 线程类头文件 mythread.h
#ifndef MYTHREAD_H
#define MYTHREAD_H

#include <QThread>

class MyThread : public QThread
{
    Q_OBJECT
protected:
    void run() override; // 重写线程入口函数

signals:
    void progressChanged(int value); // 进度更新信号
    void threadFinished();           // 线程完成信号
};

#endif // MYTHREAD_H

// 线程类实现 mythread.cpp
#include "mythread.h"

void MyThread::run()
{
    // 子线程执行的耗时操作
    for(int i=0; i<=100; i++) {
        msleep(10); // 模拟耗时
        emit progressChanged(i); // 发送信号到主线程
    }
    emit threadFinished();
}

// 主线程调用
MyThread *thread = new MyThread;
// 线程结束自动销毁
connect(thread, &MyThread::finished, thread, &MyThread::deleteLater);
// 进度信号更新UI
connect(thread, &MyThread::progressChanged, this, [=](int value){
    ui->progressBar->setValue(value); // 主线程更新UI
});
// 启动线程（自动调用run()）
thread->start();
```

### 8.2 方式二：QObject::moveToThread()（官方推荐）
将业务逻辑封装到QObject子类，把对象移动到子线程，通过信号槽触发执行，线程拥有完整事件循环，支持定时器、网络、多业务对象共享线程，代码结构更清晰。

```cpp
// 业务类头文件 worker.h
#ifndef WORKER_H
#define WORKER_H

#include <QObject>

class Worker : public QObject
{
    Q_OBJECT
public slots:
    void doWork(); // 业务处理槽函数，在子线程执行

signals:
    void progressChanged(int value);
    void workFinished();
};

#endif // WORKER_H

// 业务类实现 worker.cpp
#include "worker.h"
#include <QThread>

void Worker::doWork()
{
    // 子线程执行的耗时操作
    for(int i=0; i<=100; i++) {
        QThread::msleep(10);
        emit progressChanged(i);
    }
    emit workFinished();
}

// 主线程调用
// 1. 创建子线程
QThread *thread = new QThread;
// 2. 创建业务对象，禁止指定父对象
Worker *worker = new Worker;
// 3. 将业务对象移动到子线程
worker->moveToThread(thread);

// 4. 信号槽连接
// 线程启动后执行业务函数
connect(thread, &QThread::started, worker, &Worker::doWork);
// 工作完成，退出线程事件循环
connect(worker, &Worker::workFinished, thread, &QThread::quit);
// 线程退出，自动销毁对象
connect(thread, &QThread::finished, worker, &Worker::deleteLater);
connect(thread, &QThread::finished, thread, &QThread::deleteLater);
// 进度更新UI
connect(worker, &Worker::progressChanged, this, [=](int value){
    ui->progressBar->setValue(value);
});

// 5. 启动线程
thread->start();
```

### 8.3 多线程核心注意事项
- 跨线程信号槽默认使用`Qt::QueuedConnection`，自动保证线程安全。
- 自定义类型用于跨线程信号槽时，必须用`qRegisterMetaType()`注册。
- QObject的父子关系必须在同一个线程，不能给不同线程的对象设置父子关系。
- 线程退出：先调用`quit()`退出事件循环，再调用`wait()`等待线程结束。
- `QTimer`必须在有事件循环的线程中使用，重写`run()`的线程需调用`exec()`启动事件循环。

## 九、网络编程核心语法
Qt5网络功能由`network`模块提供，核心分为TCP、UDP、HTTP/HTTPS三类。

### 9.1 TCP 编程（QTcpSocket/QTcpServer）
面向连接的可靠传输协议，分为服务端和客户端。
#### 服务端核心语法
```cpp
#include <QTcpServer>
#include <QTcpSocket>

class TcpServer : public QObject
{
    Q_OBJECT
public:
    explicit TcpServer(QObject *parent = nullptr) {
        m_server = new QTcpServer(this);
        // 新客户端连接信号
        connect(m_server, &QTcpServer::newConnection, this, &TcpServer::onNewConnection);
    }

    // 启动服务，监听端口
    bool startServer(quint16 port) {
        return m_server->listen(QHostAddress::Any, port);
    }

private slots:
    // 处理新客户端连接
    void onNewConnection() {
        QTcpSocket *socket = m_server->nextPendingConnection();
        m_clientList.append(socket);

        // 连接信号槽
        connect(socket, &QTcpSocket::readyRead, this, &TcpServer::onReadyRead);
        connect(socket, &QTcpSocket::disconnected, socket, &QTcpSocket::deleteLater);
    }

    // 接收客户端数据
    void onReadyRead() {
        QTcpSocket *socket = qobject_cast<QTcpSocket*>(sender());
        QByteArray data = socket->readAll();
        qDebug() << "收到客户端数据：" << data;
        // 回复客户端
        socket->write("服务器已收到：" + data);
    }

private:
    QTcpServer *m_server;
    QList<QTcpSocket*> m_clientList;
};
```

#### 客户端核心语法
```cpp
#include <QTcpSocket>

class TcpClient : public QObject
{
    Q_OBJECT
public:
    explicit TcpClient(QObject *parent = nullptr) {
        m_socket = new QTcpSocket(this);
        connect(m_socket, &QTcpSocket::connected, this, &TcpClient::onConnected);
        connect(m_socket, &QTcpSocket::readyRead, this, &TcpClient::onReadyRead);
        connect(m_socket, &QTcpSocket::errorOccurred, this, &TcpClient::onError);
    }

    // 连接服务器
    void connectToServer(const QString &ip, quint16 port) {
        m_socket->connectToHost(ip, port);
    }

    // 发送数据
    void sendData(const QByteArray &data) {
        if(m_socket->state() == QAbstractSocket::ConnectedState) {
            m_socket->write(data);
            m_socket->flush();
        }
    }

private slots:
    void onConnected() { qDebug() << "连接服务器成功"; }
    void onReadyRead() {
        QByteArray data = m_socket->readAll();
        qDebug() << "收到服务器数据：" << data;
    }
    void onError(QAbstractSocket::SocketError error) {
        qDebug() << "连接错误：" << m_socket->errorString();
    }

private:
    QTcpSocket *m_socket;
};
```

### 9.2 UDP 编程（QUdpSocket）
无连接的不可靠传输协议，速度快，适合广播、实时音视频等场景。
```cpp
#include <QUdpSocket>

// 1. UDP接收端
QUdpSocket *udpReceiver = new QUdpSocket(this);
udpReceiver->bind(QHostAddress::Any, 8888); // 绑定端口
// 接收数据
connect(udpReceiver, &QUdpSocket::readyRead, this, [=](){
    while(udpReceiver->hasPendingDatagrams()) {
        QByteArray datagram;
        datagram.resize(udpReceiver->pendingDatagramSize());
        QHostAddress senderAddr;
        quint16 senderPort;
        // 读取数据
        udpReceiver->readDatagram(datagram.data(), datagram.size(), &senderAddr, &senderPort);
        qDebug() << "收到数据：" << datagram << "来自：" << senderAddr.toString() << ":" << senderPort;
    }
});

// 2. UDP发送端
QUdpSocket *udpSender = new QUdpSocket(this);
QByteArray data = "Hello UDP";
// 发送数据到目标地址和端口，广播用QHostAddress::Broadcast
udpSender->writeDatagram(data, QHostAddress("127.0.0.1"), 8888);
```

### 9.3 HTTP/HTTPS 编程（QNetworkAccessManager）
Qt5统一使用`QNetworkAccessManager`处理HTTP/HTTPS请求，异步操作，基于事件循环，全局建议只创建一个实例。
```cpp
#include <QNetworkAccessManager>
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QJsonDocument>
#include <QJsonObject>

// 1. 创建全局管理器
QNetworkAccessManager *manager = new QNetworkAccessManager(this);

// 2. GET 请求
QNetworkRequest getRequest;
getRequest.setUrl(QUrl("https://www.qt.io"));
getRequest.setRawHeader("User-Agent", "Qt5 App");

QNetworkReply *getReply = manager->get(getRequest);
connect(getReply, &QNetworkReply::finished, this, [=](){
    if(getReply->error() == QNetworkReply::NoError) {
        QByteArray data = getReply->readAll();
        qDebug() << "响应数据：" << data;
    } else {
        qDebug() << "请求错误：" << getReply->errorString();
    }
    getReply->deleteLater();
});

// 3. POST JSON 请求
QJsonObject jsonObj;
jsonObj["username"] = "test";
jsonObj["password"] = "123456";
QByteArray jsonData = QJsonDocument(jsonObj).toJson();

QNetworkRequest postRequest;
postRequest.setUrl(QUrl("https://example.com/api/login"));
postRequest.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

QNetworkReply *postReply = manager->post(postRequest, jsonData);
connect(postReply, &QNetworkReply::finished, this, [=](){
    if(postReply->error() == QNetworkReply::NoError) {
        QByteArray data = postReply->readAll();
        qDebug() << "POST响应：" << data;
    }
    postReply->deleteLater();
});
```

## 十、Qt5 进阶语法特性
### 10.1 属性系统 `Q_PROPERTY`
基于元对象系统的属性机制，支持动态属性，可与Qt设计器、QML、脚本交互，语法如下：
```cpp
class MyWidget : public QWidget
{
    Q_OBJECT
    // 定义属性
    Q_PROPERTY(int age READ age WRITE setAge NOTIFY ageChanged)
    Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)

public:
    // 读函数
    int age() const { return m_age; }
    QString name() const { return m_name; }

    // 写函数
    void setAge(int age) {
        if(m_age != age) {
            m_age = age;
            emit ageChanged(m_age);
        }
    }
    void setName(const QString &name) {
        if(m_name != name) {
            m_name = name;
            emit nameChanged(m_name);
        }
    }

signals:
    // 属性变化通知信号
    void ageChanged(int age);
    void nameChanged(const QString &name);

private:
    int m_age = 0;
    QString m_name;
};

// 属性使用
MyWidget *w = new MyWidget;
w->setProperty("age", 20);        // 写属性
int age = w->property("age").toInt(); // 读属性
```

### 10.2 元类型注册
自定义类型要在元对象系统中使用（信号槽参数、QVariant存储），必须注册：
```cpp
// 自定义类型
struct MyData {
    int id;
    QString name;
};

// 1. 头文件中声明，必须在结构体定义之后
Q_DECLARE_METATYPE(MyData)

// 2. 使用前注册（main函数中），支持跨线程信号槽
qRegisterMetaType<MyData>("MyData");

// 使用
MyData data;
data.id = 1;
data.name = "test";
QVariant var = QVariant::fromValue(data); // 存入QVariant
MyData data2 = var.value<MyData>();       // 取出数据
```

### 10.3 枚举类型注册 `Q_ENUM`
自定义枚举需用`Q_ENUM`注册，支持字符串与枚举值互转，可用于元对象系统。
```cpp
class MyClass : public QObject
{
    Q_OBJECT
public:
    enum Color {
        Red,
        Green,
        Blue
    };
    // 注册枚举
    Q_ENUM(Color)
};

// 使用
MyClass::Color color = MyClass::Red;
// 枚举值转字符串
QString colorStr = QMetaEnum::fromType<MyClass::Color>().valueToKey(color);
// 字符串转枚举值
int colorVal = QMetaEnum::fromType<MyClass::Color>().keyToValue("Blue");
```

### 10.4 定时器 `QTimer`
基于事件循环的定时器，精度取决于操作系统，推荐使用信号槽方式。
```cpp
// 1. 单次定时器，1000ms后执行一次
QTimer::singleShot(1000, this, [=](){
    qDebug() << "单次定时器执行";
});

// 2. 循环定时器
QTimer *timer = new QTimer(this);
timer->setInterval(1000); // 间隔1000ms
connect(timer, &QTimer::timeout, this, [=](){
    qDebug() << "定时器超时";
});
timer->start(); // 启动定时器
// timer->stop(); // 停止定时器
```

### 10.5 国际化 `tr()` 函数
Qt国际化核心，所有需要翻译的字符串必须用`tr()`包裹，基于元对象系统。
```cpp
// 基础用法
QString text = tr("Hello World");
// 带上下文，翻译时区分场景
QString btnText = tr("登录", "按钮文本");
// 带参数
QString userText = tr("用户名：%1").arg(username);

// 非QObject子类使用
QString text = QObject::tr("Hello World");
```

## 十一、Qt5 与 Qt4 核心语法差异
1.  **模块拆分**：Qt4的`QtGui`模块包含Widgets，Qt5将Widgets拆分为独立的`QtWidgets`模块，必须手动添加`QT += widgets`。
2.  **信号槽语法**：Qt5新增基于函数指针的信号槽语法，编译期类型检查，支持Lambda；Qt4的`SIGNAL/SLOT`宏仅做兼容，不推荐使用。
3.  **多线程**：Qt5官方推荐`moveToThread`方式，修复了Qt4`QThread`的设计缺陷，`run()`默认调用`exec()`启动事件循环。
4.  **网络模块**：Qt5移除了Qt4的`QHttp/QFtp`类，统一使用`QNetworkAccessManager`处理HTTP请求。
5.  **C++标准**：Qt5要求编译器支持C++11及以上，Qt4默认不支持C++11。
6.  **容器类**：Qt5优化了`QList`，Qt5.15后`QList`与`QVector`合并，推荐使用`QList`替代`QVector`。
7.  **API更新**：Qt5废弃了Qt4大量过时API，例如`QString::fromAscii()`替换为`QString::fromLatin1()`，`QPtrList`替换为`QList`。

## 十二、常见开发坑与避坑指南
1.  **信号槽连接失败**：忘记加`Q_OBJECT`宏、类声明未放在头文件、参数不匹配，解决：添加宏后重新执行`qmake`。
2.  **程序崩溃**：子线程操作UI控件，解决：只能通过信号槽通知主线程操作UI。
3.  **内存泄漏**：QObject对象未设置父对象且未手动释放，解决：利用父子对象机制，父对象销毁时自动释放子对象。
4.  **事件不响应**：事件处理函数未调用父类的实现，导致事件传递中断，解决：重写事件函数后，末尾调用父类的对应函数。
5.  **中文乱码**：Qt5默认使用UTF-8编码，解决：源码文件保存为UTF-8编码，使用`QString::fromUtf8()`处理中文字符串。
6.  **跨线程信号槽失效**：自定义类型未注册元类型，解决：使用`qRegisterMetaType()`注册自定义类型。


