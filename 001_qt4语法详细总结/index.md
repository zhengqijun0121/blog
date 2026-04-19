# Qt4 语法详细总结


# Qt4 语法详细总结
Qt4 是 Qt 经典的长期支持版本（最终稳定版为 Qt4.8.7），基于**元对象系统**扩展标准C++，核心主打跨平台GUI开发，同时提供完善的网络、数据库、XML、线程等能力。以下是其核心语法、模块特性与使用规范的完整总结，同时标注与Qt5的核心差异，避免混淆。

---

## 一、Qt4 核心基石：元对象系统（Meta-Object System）
元对象系统是Qt对C++的核心扩展，实现了信号槽、属性系统、运行时类型信息等核心特性，**使用必须满足3个必要条件**：
1.  类必须直接或间接继承自 `QObject`
2.  类声明的私有区域必须添加 `Q_OBJECT` 宏
3.  源文件必须通过MOC（元对象编译器）预处理，再参与编译

### 1.1 核心宏语法
| 宏 | 作用与语法规范 |
| :--- | :--- |
| `Q_OBJECT` | 核心宏，必须放在类声明的最开头（私有区），启用元对象特性，无参数。添加后必须重新执行qmake，否则会编译报错 |
| `Q_PROPERTY(type name READ getFunc WRITE setFunc [NOTIFY signal] [DESIGNABLE bool] ...)` | 属性系统宏，为类声明可通过元对象访问的属性。<br>必填：`type`属性类型、`name`属性名、`READ`读函数、`WRITE`写函数<br>可选：`NOTIFY`属性变化时触发的信号、`DESIGNABLE`是否在设计师可见 |
| `Q_ENUM(EnumType)` | 注册枚举类型到元对象系统，可通过元对象获取枚举名、值，支持字符串与枚举值互转 |
| `Q_FLAGS(FlagsType)` | 注册位运算枚举（标志位），支持按位与/或操作 |

### 1.2 核心配套语法
- **父子对象机制**：`QObject` 构造函数默认带父对象参数 `QObject *parent = 0`，设置父对象后，子对象会被加入父对象的子列表，**父对象析构时会自动销毁所有子对象**，是Qt核心的内存管理方案。
  示例：`QPushButton *btn = new QPushButton("确定", this);` 无需手动delete btn
- **国际化tr()函数**：所有需要翻译的字符串必须用 `tr()` 包裹，由元对象系统处理国际化，语法：`tr("你好Qt4")`，支持多语言切换。
- **调试输出qDebug()**：Qt4标准调试输出，需包含头文件 `<QDebug>`，语法：`qDebug() << "数值:" << 100 << "字符串:" << tr("测试");`

---

## 二、Qt4 核心特性：信号与槽（Signals & Slots）
信号槽是Qt的核心通信机制，实现了对象间的解耦通信，**Qt4的语法与Qt5有本质区别，必须严格遵守规范**。

### 2.1 信号的声明语法
- 关键字：`signals:`（Qt4专属宏，不可加访问控制修饰符，默认展开为protected）
- 语法规则：
  1.  仅可声明，不可实现（MOC自动生成实现代码）
  2.  返回值固定为 `void`，可自定义任意个数、类型的参数
  3.  不可手动调用，仅可通过 `emit` 关键字发射
- 示例：
```cpp
class MyWidget : public QWidget
{
    Q_OBJECT
signals:
    // 自定义信号，无实现
    void valueChanged(int newValue);
    void userLogin(QString username, int userId);
};
```

### 2.2 槽的声明语法
- 关键字：`slots:`，必须搭配访问控制符，分为 `public slots` / `protected slots` / `private slots`
- 语法规则：
  1.  **Qt4强制要求**：槽函数必须声明在slots块中，普通成员函数无法直接作为槽使用
  2.  槽函数是普通的成员函数，可手动调用，也可被信号触发
  3.  槽函数的参数类型、顺序必须与连接的信号匹配，信号参数个数可多于槽（多余参数会被忽略）
- 示例：
```cpp
public slots:
    // 与信号valueChanged(int)匹配的槽
    void onValueChanged(int value) {
        qDebug() << "当前值:" << value;
    }
    // 无参槽，可连接无参信号clicked()
    void onBtnClicked() {
        QMessageBox::information(this, "提示", "按钮被点击");
    }
```

### 2.3 信号发射与连接核心语法
#### 1. 信号发射：emit关键字
`emit` 是Qt专属宏，展开为空，仅做语法提示，用于发射信号，语法：
```cpp
// 发射信号，传递参数
emit valueChanged(200);
emit userLogin("zhangsan", 1001);
```

#### 2. 连接函数connect（Qt4标准语法）
Qt4仅支持基于宏的连接方式，语法固定，参数错误会导致运行时连接失败（编译不报错）。
```cpp
// 函数原型
bool QObject::connect(
    const QObject *sender,      // 信号发送者
    const char *signal,          // 信号，必须用SIGNAL()宏包裹
    const QObject *receiver,    // 信号接收者
    const char *method,          // 槽函数，必须用SLOT()宏包裹
    Qt::ConnectionType type = Qt::AutoConnection  // 连接类型，可选
);
```

**核心语法规范（Qt4强制要求）**：
1.  信号必须用 `SIGNAL(函数签名)` 包裹，槽必须用 `SLOT(函数签名)` 包裹
2.  宏内的函数签名**仅可写参数类型，不可写变量名**，否则MOC无法匹配
3.  函数签名必须与声明完全一致，包括const修饰符
4.  返回值：连接成功返回true，失败返回false，可通过返回值判断连接状态

**正确示例**：
```cpp
QPushButton *btn = new QPushButton("点击", this);
// 正确：无参信号连接无参槽
connect(btn, SIGNAL(clicked()), this, SLOT(onBtnClicked()));

QSlider *slider = new QSlider(Qt::Horizontal, this);
// 正确：带参信号连接带参槽，仅写类型
connect(slider, SIGNAL(valueChanged(int)), this, SLOT(onValueChanged(int)));
```

**错误示例**：
```cpp
// 错误1：宏内写了变量名
connect(slider, SIGNAL(valueChanged(int value)), this, SLOT(onValueChanged(int v)));
// 错误2：槽函数未声明在slots块中
connect(btn, SIGNAL(clicked()), this, SLOT(normalFunc()));
// 错误3：参数类型不匹配
connect(slider, SIGNAL(valueChanged(int)), this, SLOT(onValueChanged(QString)));
```

#### 3. 连接类型枚举
| 枚举值 | 作用 |
| :--- | :--- |
| `Qt::AutoConnection`（默认） | 自动判断：发送者和接收者在同一线程用直连，跨线程用队列连接 |
| `Qt::DirectConnection` | 信号发射时，立即调用槽函数，槽函数在发送者线程执行 |
| `Qt::QueuedConnection` | 信号发射后，放入接收者线程的事件队列，等待事件循环执行，槽函数在接收者线程执行 |
| `Qt::BlockingQueuedConnection` | 阻塞队列连接，槽函数执行完成后，发送者线程才继续运行，**禁止同一线程使用，会死锁** |

#### 4. 断开连接disconnect语法
Qt4中断开连接同样使用宏语法，示例：
```cpp
// 断开指定发送者、信号、接收者、槽的连接
disconnect(btn, SIGNAL(clicked()), this, SLOT(onBtnClicked()));
// 断开btn所有信号的所有连接
disconnect(btn, 0, 0, 0);
// 断开btn的clicked信号的所有连接
disconnect(btn, SIGNAL(clicked()), 0, 0);
```

### 2.4 信号槽核心注意事项
1.  包含`Q_OBJECT`宏的类，必须写在头文件中，MOC仅处理头文件的类声明
2.  新增/修改信号槽后，必须重新执行qmake，否则会出现编译报错或连接失败
3.  信号槽的参数类型必须是Qt元对象系统支持的类型，自定义类型需用 `qRegisterMetaType<Type>()` 注册
4.  一个信号可连接多个槽，多个信号可连接同一个槽，信号之间也可互相连接

---

## 三、Qt4 项目构建：.pro 项目文件语法
Qt4默认使用qmake构建系统，通过`.pro`文件配置项目，核心语法如下：

### 3.1 基础配置项
| 配置项 | 语法与说明 |
| :--- | :--- |
| `TEMPLATE` | 项目模板，必填。<br>`app`：桌面应用程序（默认）；`lib`：静态/动态库；`dll`：动态库；`subdirs`：子目录项目 |
| `TARGET` | 生成的目标文件名，示例：`TARGET = MyQt4App` |
| `QT` | 引入Qt模块，**Qt4默认包含core和gui模块，无需手动添加**。<br>示例：`QT += network sql xml`（引入网络、数据库、XML模块） |
| `SOURCES` | 源文件列表，示例：`SOURCES += main.cpp mainwindow.cpp` |
| `HEADERS` | 头文件列表，示例：`HEADERS += mainwindow.h` |
| `FORMS` | Qt设计师界面文件(.ui)，示例：`FORMS += mainwindow.ui` |
| `RESOURCES` | Qt资源文件(.qrc)，示例：`RESOURCES += res.qrc` |

### 3.2 进阶配置项
| 配置项 | 语法与说明 |
| :--- | :--- |
| `CONFIG` | 通用配置，示例：<br>`CONFIG += c++11`（启用C++11，Qt4.8+支持）；<br>`CONFIG += static`（编译静态库）；<br>`CONFIG += console`（启用控制台输出） |
| `DEFINES` | 全局宏定义，示例：`DEFINES += MY_APP_VERSION=100` |
| `INCLUDEPATH` | 头文件搜索路径，示例：`INCLUDEPATH += ./3rdparty/include` |
| `LIBS` | 链接外部库，示例：`LIBS += -L./3rdparty/lib -lmysql` |
| `DESTDIR` | 目标文件输出路径，示例：`DESTDIR = ./bin` |

---

## 四、Qt4 核心模块与常用类语法
### 4.1 Core核心模块（非GUI基础能力）
#### 1. 跨平台基础数据类型
Qt4提供固定宽度的跨平台类型，避免不同系统的基础类型宽度差异：
| 类型 | 说明 |
| :--- | :--- |
| `qint8`/`quint8` | 8位有符号/无符号整型 |
| `qint16`/`quint16` | 16位有符号/无符号整型 |
| `qint32`/`quint32` | 32位有符号/无符号整型 |
| `qint64`/`quint64` | 64位有符号/无符号整型 |
| `qreal` | 浮点类型，默认是double，嵌入式平台可配置为float |

#### 2. 核心字符串与字节数组
- **QString**：Qt核心字符串类，基于Unicode，支持全平台中文处理，常用语法：
  ```cpp
  // 构造
  QString str1 = "Hello Qt4";
  QString str2 = QString::fromUtf8("你好Qt4"); // UTF-8编码转QString
  QString str3 = QString::fromLocal8Bit("本地编码中文"); // 系统本地编码（Windows GBK/Linux UTF-8）

  // 格式化（Qt4主推，替代sprintf）
  QString info = QString("姓名：%1 年龄：%2").arg("张三").arg(20);

  // 常用方法
  str1.append(" World"); // 追加
  str1.replace("Hello", "Hi"); // 替换
  QStringList list = str1.split(" "); // 分割
  str1.trimmed(); // 去除首尾空白
  ```
- **QByteArray**：二进制数据处理类，用于文件、网络数据传输，支持与`char*`互转。

#### 3. 容器类
Qt4容器基于**隐式共享（写时复制）**，内存效率高，兼容Java风格和STL风格迭代器，同时提供专属`foreach`遍历宏。
| 容器类型 | 常用类 | 核心语法 |
| :--- | :--- | :--- |
| 顺序容器 | `QList<T>`（通用列表，Qt4主推）、`QVector<T>`、`QLinkedList<T>`、`QStack<T>`、`QQueue<T>` | 初始化：`QList<QString> list; list << "A" << "B" << "C";`<br>增删：`list.append("D"); list.removeAt(0);`<br>取值：`list[0]`、`list.at(1)`（只读，效率更高） |
| 关联容器 | `QMap<K,T>`（有序键值对）、`QMultiMap<K,T>`、`QHash<K,T>`（无序哈希表，效率更高）、`QMultiHash<K,T>`、`QSet<T>` | 初始化：`QMap<int, QString> map; map[1] = "张三"; map[2] = "李四";`<br>取值：`map.value(1)`、`map[2]`<br>遍历：`foreach(int key, map.keys()) { qDebug() << map[key]; }` |

**Qt4专属foreach遍历宏**：简化容器遍历，无需迭代器，语法：
```cpp
// 遍历QList
foreach(QString item, list) {
    qDebug() << item;
}

// 遍历QMap
foreach(int key, map.keys()) {
    qDebug() << key << ":" << map[key];
}
```

#### 4. 事件系统
Qt4是事件驱动框架，所有事件继承自`QEvent`，事件处理有2种核心方式：
1.  **重写专属事件处理函数**：最常用方式，重写对应事件的虚函数，示例：
    ```cpp
    // 头文件声明
    protected:
        void mousePressEvent(QMouseEvent *event); // 鼠标按下事件
        void keyPressEvent(QKeyEvent *event);     // 键盘按下事件
        void paintEvent(QPaintEvent *event);      // 绘制事件
        void resizeEvent(QResizeEvent *event);    // 窗口大小变化事件

    // 源文件实现
    void MyWidget::mousePressEvent(QMouseEvent *event) {
        if(event->button() == Qt::LeftButton) {
            qDebug() << "鼠标左键按下，坐标：" << event->pos();
        }
        // 事件传递：accept()接收事件，不再向上传递；ignore()忽略事件，传递给父窗口
        event->accept();
    }
    ```
2.  **事件过滤器eventFilter**：监听其他对象的事件，无需继承子类，语法：
    ```cpp
    // 安装事件过滤器，给btn安装当前类的过滤器
    btn->installEventFilter(this);

    // 重写eventFilter函数
    bool MyWidget::eventFilter(QObject *obj, QEvent *event) {
        if(obj == btn && event->type() == QEvent::MouseButtonPress) {
            qDebug() << "按钮被点击，事件过滤器拦截";
            return true; // 返回true表示拦截事件，不再传递给原对象
        }
        // 其他事件交给父类处理
        return QWidget::eventFilter(obj, event);
    }
    ```

#### 5. 线程与定时器
- **QThread线程类**：Qt4线程核心类，有2种使用方式：
  1.  继承QThread，重写`run()`函数（Qt4主流用法），`run()`内的代码在子线程执行：
      ```cpp
      class MyThread : public QThread
      {
          Q_OBJECT
      protected:
          void run() {
              // 子线程执行的代码，耗时操作放在这里
              for(int i=0; i<10; i++) {
                  msleep(500); // 线程休眠
                  qDebug() << "子线程运行：" << i;
              }
          }
      };

      // 使用
      MyThread *thread = new MyThread(this);
      thread->start(); // 启动线程，自动调用run()
      ```
  2.  `moveToThread()`：将QObject子类移到线程中，通过信号槽触发执行，Qt4已支持。
  线程同步类：`QMutex`/`QMutexLocker`、`QSemaphore`、`QWaitCondition`。

- **QTimer定时器类**：Qt4主推的定时器，基于事件循环，语法：
  ```cpp
  QTimer *timer = new QTimer(this);
  // 连接超时信号
  connect(timer, SIGNAL(timeout()), this, SLOT(onTimeout()));
  timer->start(1000); // 启动定时器，1000ms触发一次
  timer->setSingleShot(true); // 设置为单次触发
  ```

#### 6. 文件与IO操作
核心类：`QFile`（文件操作）、`QTextStream`（文本流）、`QDataStream`（二进制数据流，跨平台序列化）、`QDir`（目录操作）、`QFileInfo`（文件信息）。
示例：
```cpp
// 文本文件读取
QFile file("test.txt");
if(file.open(QIODevice::ReadOnly | QIODevice::Text)) {
    QTextStream in(&file);
    in.setCodec("UTF-8"); // 设置编码
    while(!in.atEnd()) {
        QString line = in.readLine();
        qDebug() << line;
    }
    file.close();
}

// 文本文件写入
if(file.open(QIODevice::WriteOnly | QIODevice::Text | QIODevice::Truncate)) {
    QTextStream out(&file);
    out.setCodec("UTF-8");
    out << "你好Qt4" << endl;
    file.close();
}
```

### 4.2 GUI模块（Qt4核心可视化能力）
Qt4的GUI能力全部集成在gui模块中（Qt5才拆分为gui和widgets模块），所有可视化部件的基类是`QWidget`。

#### 1. 窗口基础语法
- **基础窗口QWidget**：
  ```cpp
  QWidget *w = new QWidget();
  w->setWindowTitle("Qt4窗口"); // 设置窗口标题
  w->resize(800, 600); // 设置窗口大小
  w->setFixedSize(800, 600); // 固定窗口大小
  w->move(100, 100); // 移动窗口位置
  w->show(); // 显示窗口
  ```
- **主窗口QMainWindow**：标准桌面主窗口，自带菜单栏、工具栏、状态栏、中央部件，语法：
  ```cpp
  QMainWindow *mw = new QMainWindow();
  mw->resize(1000, 600);
  // 必须设置中央部件
  QWidget *centralWidget = new QWidget();
  mw->setCentralWidget(centralWidget);
  // 状态栏
  mw->statusBar()->showMessage("就绪", 3000);
  // 菜单栏
  QMenu *fileMenu = mw->menuBar()->addMenu("文件");
  fileMenu->addAction("新建");
  fileMenu->addAction("退出");
  mw->show();
  ```
- **对话框QDialog**：
  ```cpp
  // 模态对话框：阻塞父窗口，exec()返回后才继续执行
  QDialog dialog(this);
  dialog.setWindowTitle("模态对话框");
  dialog.resize(300, 200);
  int ret = dialog.exec();
  if(ret == QDialog::Accepted) {
      qDebug() << "对话框确认";
  }

  // 非模态对话框：不阻塞父窗口
  QDialog *dialog2 = new QDialog(this);
  dialog2->setAttribute(Qt::WA_DeleteOnClose); // 关闭时自动销毁
  dialog2->show();
  ```

#### 2. 布局管理（Qt4核心自适应方案）
Qt4布局实现窗口自适应缩放，替代固定坐标，四大核心布局类：
| 布局类 | 作用 | 核心语法 |
| :--- | :--- | :--- |
| `QHBoxLayout` | 水平布局，控件从左到右排列 | `QHBoxLayout *layout = new QHBoxLayout();`<br>`layout->addWidget(btn1);`<br>`layout->addWidget(btn2);`<br>`w->setLayout(layout);` |
| `QVBoxLayout` | 垂直布局，控件从上到下排列 | 同水平布局，控件垂直排列 |
| `QGridLayout` | 网格布局，按行、列排列控件 | `QGridLayout *layout = new QGridLayout();`<br>`layout->addWidget(label, 0, 0); // 第0行第0列`<br>`layout->addWidget(edit, 0, 1); // 第0行第1列` |
| `QFormLayout` | 表单布局，标签+输入框的两列布局（Qt4.2+引入） | `QFormLayout *layout = new QFormLayout();`<br>`layout->addRow("姓名：", nameEdit);`<br>`layout->addRow("年龄：", ageEdit);` |

**常用布局方法**：
- `addWidget()`：添加控件，可设置拉伸系数、对齐方式
- `addLayout()`：嵌套布局，实现复杂界面
- `addStretch()`：添加拉伸项，填充空白区域
- `setSpacing()`：设置控件之间的间距
- `setContentsMargins()`：设置布局的内边距

示例：
```cpp
QWidget *w = new QWidget();
w->setWindowTitle("布局示例");
w->resize(400, 300);

// 垂直布局
QVBoxLayout *mainLayout = new QVBoxLayout(w);

// 表单布局
QFormLayout *formLayout = new QFormLayout();
formLayout->addRow("用户名：", new QLineEdit());
formLayout->addRow("密码：", new QLineEdit());
mainLayout->addLayout(formLayout);

// 水平按钮布局
QHBoxLayout *btnLayout = new QHBoxLayout();
btnLayout->addStretch(); // 左侧拉伸
btnLayout->addWidget(new QPushButton("登录"));
btnLayout->addWidget(new QPushButton("取消"));
mainLayout->addLayout(btnLayout);

mainLayout->setContentsMargins(20, 20, 20, 20);
mainLayout->setSpacing(15);
w->show();
```

#### 3. 常用可视化部件
| 部件类 | 作用 | 核心语法 |
| :--- | :--- | :--- |
| `QLabel` | 标签，显示文本、图片、动画 | `QLabel *label = new QLabel("文本标签");`<br>`label->setPixmap(QPixmap(":/images/icon.png")); // 显示图片`<br>`label->setAlignment(Qt::AlignCenter); // 居中对齐` |
| `QPushButton` | 普通按钮 | `QPushButton *btn = new QPushButton("按钮文本");`<br>`connect(btn, SIGNAL(clicked()), this, SLOT(onBtnClicked()));` |
| `QLineEdit` | 单行文本输入框 | `QLineEdit *edit = new QLineEdit();`<br>`edit->setPlaceholderText("请输入内容");`<br>`edit->setEchoMode(QLineEdit::Password); // 密码模式` |
| `QTextEdit` | 多行富文本编辑器 | `QTextEdit *edit = new QTextEdit();`<br>`edit->setHtml("<b>富文本内容</b>");`<br>`QString text = edit->toPlainText();` |
| `QCheckBox` | 复选框 | `QCheckBox *check = new QCheckBox("记住密码");`<br>`connect(check, SIGNAL(stateChanged(int)), this, SLOT(onCheckChanged(int)));` |
| `QRadioButton` | 单选按钮 | 需搭配`QButtonGroup`实现互斥 |
| `QComboBox` | 下拉选择框 | `QComboBox *combo = new QComboBox();`<br>`combo->addItem("选项1");`<br>`combo->addItems(QStringList() << "选项2" << "选项3");` |
| `QSlider` | 滑块 | `QSlider *slider = new QSlider(Qt::Horizontal);`<br>`slider->setRange(0, 100);`<br>`slider->setValue(50);` |
| `QProgressBar` | 进度条 | `QProgressBar *bar = new QProgressBar();`<br>`bar->setRange(0, 100);`<br>`bar->setValue(30);` |

#### 4. 2D绘图与图形视图
- **QPainter绘图**：Qt4核心2D绘图类，**必须在`paintEvent`函数中使用**，语法：
  ```cpp
  void MyWidget::paintEvent(QPaintEvent *event) {
      QPainter painter(this);
      // 设置画笔（线条）
      painter.setPen(QPen(Qt::red, 2, Qt::SolidLine));
      // 设置画刷（填充）
      painter.setBrush(QBrush(Qt::blue, Qt::DiagCrossPattern));
      // 绘制图形
      painter.drawRect(20, 20, 100, 100); // 矩形
      painter.drawEllipse(150, 20, 100, 100); // 圆形
      painter.drawLine(20, 150, 250, 150); // 直线
      // 绘制文本
      painter.drawText(20, 180, "Qt4 绘图示例");
      // 绘制图片
      painter.drawPixmap(20, 200, QPixmap(":/images/test.png"));
  }
  ```
- **QGraphicsView图形视图框架**：Qt4.2+引入，用于管理大量2D图形项，支持缩放、旋转、碰撞检测，核心三要素：`QGraphicsScene`（场景，存放图形项）、`QGraphicsView`（视图，显示场景）、`QGraphicsItem`（图形项）。

#### 5. 资源系统
Qt4资源系统将图片、配置文件等资源编译到可执行文件中，避免路径问题，语法规范：
1.  创建`.qrc`资源文件，格式如下：
    ```xml
    <RCC>
        <qresource prefix="/">
            <file>images/icon.png</file>
            <file>css/style.qss</file>
        </qresource>
    </RCC>
    ```
2.  在`.pro`文件中添加：`RESOURCES += res.qrc`
3.  代码中访问资源，路径格式为 `:/前缀/文件路径`，示例：
    ```cpp
    QPixmap(":/images/icon.png");
    QFile file(":/css/style.qss");
    ```

### 4.3 其他常用模块语法
#### 1. 网络模块QtNetwork
需在`.pro`中添加 `QT += network`，核心类：
- `QTcpSocket`/`QTcpServer`：TCP通信
- `QUdpSocket`：UDP通信
- `QHttp`/`QFtp`：HTTP/FTP通信（Qt4内置，Qt5移到QtExtras）

TCP客户端示例：
```cpp
QTcpSocket *socket = new QTcpSocket(this);
// 连接服务器
socket->connectToHost("127.0.0.1", 8080);
// 接收数据
connect(socket, SIGNAL(readyRead()), this, SLOT(onReadData()));
// 发送数据
socket->write("Hello Server");

// 槽函数实现
void MyWidget::onReadData() {
    QByteArray data = socket->readAll();
    qDebug() << "收到数据：" << data;
}
```

#### 2. 数据库模块QtSql
需在`.pro`中添加 `QT += sql`，核心类：
- `QSqlDatabase`：数据库连接管理
- `QSqlQuery`：SQL语句执行
- `QSqlTableModel`/`QSqlQueryModel`：Model/View数据库模型

示例：
```cpp
// 添加SQLite数据库连接
QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
db.setDatabaseName("test.db");
// 打开数据库
if(!db.open()) {
    qDebug() << "数据库打开失败";
    return;
}
// 执行SQL语句
QSqlQuery query;
// 创建表
query.exec("CREATE TABLE IF NOT EXISTS user(id INT PRIMARY KEY, name VARCHAR(20), age INT)");
// 插入数据
query.prepare("INSERT INTO user(id, name, age) VALUES (?, ?, ?)");
query.addBindValue(1);
query.addBindValue("张三");
query.addBindValue(20);
query.exec();
// 查询数据
query.exec("SELECT * FROM user");
while(query.next()) {
    int id = query.value(0).toInt();
    QString name = query.value(1).toString();
    qDebug() << id << name;
}
```

#### 3. XML模块QtXml
需在`.pro`中添加 `QT += xml`，提供DOM方式（`QDomDocument`）和流方式（`QXmlStreamReader`/`QXmlStreamWriter`，Qt4.3+引入）解析XML。

#### 4. 国际化
Qt4国际化核心语法：
1.  所有用户可见的字符串用`tr()`包裹
2.  `.pro`文件添加翻译文件：`TRANSLATIONS += myapp_zh_CN.ts`
3.  用`lupdate`工具提取字符串生成`.ts`文件，用Qt Linguist编辑翻译
4.  用`lrelease`工具生成`.qm`二进制翻译文件
5.  代码中加载翻译文件：
    ```cpp
    QApplication a(argc, argv);
    QTranslator translator;
    if(translator.load("myapp_zh_CN.qm")) {
        a.installTranslator(&translator);
    }
    ```

---

## 五、Qt4 中文乱码解决方案（专属语法）
Qt4中文乱码是高频问题，核心是编码不匹配，通用解决方案：
1.  代码文件编码统一为UTF-8
2.  在main函数开头设置全局编码：
    ```cpp
    #include <QTextCodec>
    int main(int argc, char *argv[])
    {
        QApplication a(argc, argv);
        // 设置全局编码
        QTextCodec *codec = QTextCodec::codecForName("UTF-8");
        QTextCodec::setCodecForTr(codec);       // 设置tr()函数编码
        QTextCodec::setCodecForCStrings(codec); // 设置字符串常量编码
        QTextCodec::setCodecForLocale(codec);   // 设置本地编码
        // 其他代码
    }
    ```
> 注意：这三个setCodecFor*函数在Qt5中已被废弃，是Qt4专属语法。

---

## 六、Qt4 与 Qt5 核心语法差异
| 特性 | Qt4 语法 | Qt5 语法 |
| :--- | :--- | :--- |
| 信号槽连接 | 仅支持`SIGNAL()`/`SLOT()`宏，槽必须声明在slots块 | 兼容宏语法，新增函数指针语法，普通成员函数可作为槽 |
| 模块划分 | core和gui默认包含，gui集成所有widgets | core和gui默认包含，widgets拆分为独立模块，需手动加`QT += widgets` |
| 中文编码 | 提供`setCodecForTr`/`setCodecForCStrings`设置全局编码 | 废弃上述函数，默认全UTF-8编码 |
| 容器遍历 | 主推`foreach`宏 | 推荐C++11范围for，`foreach`保留但不推荐 |
| 网络模块 | 内置`QHttp`/`QFtp`类 | 移除上述类，移到QtExtras模块 |
| 线程用法 | 继承QThread重写`run()`为主流 | 推荐`moveToThread()`方式，`run()`默认启动事件循环 |

---

## 七、Qt4 完整语法示例
### 7.1 头文件 mainwindow.h
```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QPushButton>
#include <QLineEdit>

class MainWindow : public QMainWindow
{
    Q_OBJECT
public:
    MainWindow(QWidget *parent = 0);
    ~MainWindow();

signals:
    void loginSuccess(QString username);

public slots:
    void onLoginBtnClicked();
    void onLoginSuccess(QString username);

private:
    QLineEdit *m_userEdit;
    QLineEdit *m_pwdEdit;
    QPushButton *m_loginBtn;
};

#endif // MAINWINDOW_H
```

### 7.2 源文件 mainwindow.cpp
```cpp
#include "mainwindow.h"
#include <QVBoxLayout>
#include <QFormLayout>
#include <QMessageBox>
#include <QDebug>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    this->setWindowTitle("Qt4 登录示例");
    this->resize(400, 300);

    // 中央部件
    QWidget *centralWidget = new QWidget(this);
    this->setCentralWidget(centralWidget);

    // 布局
    QVBoxLayout *mainLayout = new QVBoxLayout(centralWidget);
    QFormLayout *formLayout = new QFormLayout();

    // 控件创建
    m_userEdit = new QLineEdit();
    m_pwdEdit = new QLineEdit();
    m_pwdEdit->setEchoMode(QLineEdit::Password);
    m_loginBtn = new QPushButton("登录");

    // 添加到布局
    formLayout->addRow("用户名：", m_userEdit);
    formLayout->addRow("密码：", m_pwdEdit);
    mainLayout->addLayout(formLayout);
    mainLayout->addWidget(m_loginBtn, 0, Qt::AlignCenter);
    mainLayout->setContentsMargins(30, 30, 30, 30);
    mainLayout->setSpacing(20);

    // 信号槽连接
    connect(m_loginBtn, SIGNAL(clicked()), this, SLOT(onLoginBtnClicked()));
    connect(this, SIGNAL(loginSuccess(QString)), this, SLOT(onLoginSuccess(QString)));
}

MainWindow::~MainWindow()
{
}

void MainWindow::onLoginBtnClicked()
{
    QString username = m_userEdit->text().trimmed();
    QString password = m_pwdEdit->text().trimmed();

    if(username.isEmpty() || password.isEmpty()) {
        QMessageBox::warning(this, "警告", "用户名和密码不能为空");
        return;
    }

    // 简单校验
    if(username == "admin" && password == "123456") {
        emit loginSuccess(username);
    } else {
        QMessageBox::critical(this, "错误", "用户名或密码错误");
    }
}

void MainWindow::onLoginSuccess(QString username)
{
    QMessageBox::information(this, "成功", QString("欢迎你，%1").arg(username));
    qDebug() << "用户" << username << "登录成功";
}
```

### 7.3 主函数 main.cpp
```cpp
#include <QApplication>
#include "mainwindow.h"
#include <QTextCodec>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    // 设置全局编码，解决中文乱码
    QTextCodec *codec = QTextCodec::codecForName("UTF-8");
    QTextCodec::setCodecForTr(codec);
    QTextCodec::setCodecForCStrings(codec);
    QTextCodec::setCodecForLocale(codec);

    MainWindow w;
    w.show();
    return a.exec();
}
```

### 7.4 项目文件 Qt4Demo.pro
```cpp
TEMPLATE = app
TARGET = Qt4Demo
SOURCES += main.cpp mainwindow.cpp
HEADERS += mainwindow.h
QT += core gui
CONFIG += c++11
```

---

## 八、Qt4 语法核心避坑指南
1.  **Q_OBJECT宏必须加**：继承QObject的类，若使用信号槽、属性系统，必须加Q_OBJECT宏，新增/修改后必须重新qmake。
2.  **信号槽宏内禁止写变量名**：`SIGNAL()`和`SLOT()`宏内仅可写参数类型，不可写变量名，否则运行时连接失败。
3.  **槽函数必须声明在slots块中**：Qt4强制要求，普通成员函数无法作为槽使用。
4.  **QObject子类禁止拷贝**：QObject的拷贝构造和赋值运算符是私有的，不可值传递，必须用指针操作。
5.  **GUI操作必须在主线程**：所有QWidget相关的创建、修改、显示操作，必须在主线程执行，禁止在子线程操作GUI。
6.  **中文乱码统一编码**：代码文件编码、全局编码、文件读取编码必须统一，推荐全程使用UTF-8。


