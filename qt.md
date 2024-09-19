# QT

## 基础知识

### QApplication

QApplication是管理GUI程序的控制流和主要设置

对于QT写的任何一个GUI应用，不管应用有没有窗口或者有多少个窗口，有且只有一个`QApplication`对象

#### QApplication::QApplication(int &argc, char **argv)

- 使用在argv中包含的argc个命令行参数，初始化窗口系统以及应用对象
- 有一个全局的指针app指向应用对象(`QApplication`)，且只能创建一个`QApplication`对象
- QApplication对象必须在绘制设备(设备包括窗口、像素映射、位图等)之前创建

#### int QApplication::exec()

进入主循环，直到exit()被调用

### QWidget

QWidget类是所有窗口类的父类（控件类也属于窗口类），并且QWidget类的父类是QObject，意味着所有的窗口类对象只要制定了父对象，都可以实现内存资源的自动回收

### QAbstractTableModel

`QAbstractTableModel`继承自`QAbstractItemModel`，主要为QTableView提供相关接口，可以子类化该抽象类实现相关接口

### QTableView

- 表格视图控件`QTableView`，需要和`QStandardItemModel`配套使用，这套框架基于MVC设计
- M(Model)：`QStandardItemModel`数据模型，不能单独显示出来
- V(View)：`QTableView`视图，用于显示数据模型
- C(Controller)：视图在Qt中被弱化，与View合并到一起

### QStackedLayout

- 继承自QLayout，提供了多页面切换的布局，一次只能看到一个界面
- 可用于创建类似于QTabWidget提供的用户界面

### QPushButton

按钮，常用控件之一，有QPushButton(普通按钮)、QRadioButton(单选按钮)、QToolButton(工具栏按钮)

### QFont

调整Qt部件的大小、字体、字间距、下划线等属性

### QHBoxLayout

可以在水平方向或者垂直方向上排列控件，由QHBoxLayout、QVboxLayout继承

- QHBoxLayout：水平布局，在水平方向上排列控件，即：左右排列。 
- QVBoxLayout：垂直布局，在垂直方向上排列控件，即：上下排列。

### QLabel

文本控件，显示一串文本

### Q_OBJECT

只有加入了Q_OBJECT，才能使用QT中的signal和slot机制，而且Q_OBJECT要放在类的最前面

1. 支持信号与槽机制：`Qt` 的信号与槽机制是其强大的特性之一。使用 `Q_OBJECT` 宏可以让类能够使用信号和槽，实现对象之间的通信。例如，当某个事件发生时发送信号，其他对象可以通过连接到这个信号来响应。
2. 实现元对象系统：`Qt` 的元对象系统提供了一些高级特性，如动态属性、反射等。通过 `Q_OBJECT` ，类能够被元对象系统所识别和处理，从而支持这些特性。
3. 处理对象的内存管理和生命周期：在某些情况下，`Qt` 的元对象系统会对带有 `Q_OBJECT` 的类的对象进行特殊的内存管理和生命周期控制。

### QVarient

`QVariant` 是 Qt 中的一个类，用于存储不同类型的值。

- 它可以存储多种常见的数据类型，如整数、浮点数、字符串、布尔值、数组、列表，甚至自定义的数据类型。

- 主要作用是在不同组件或模块之间传递数据，特别是当数据类型不明确或可能变化的情况下。它提供了一种灵活且类型安全的方式来处理多种数据类型。
