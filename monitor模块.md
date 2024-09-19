# monitor模块

## C++基础知识

### struct和class区别

1. struct：public；class：private
2. struct 默认是公有继承，即子类仍然是public；而 class 是私有继承，子类是private
3. struct：描述一个数据结构；class：对一个对象数据的封装

### C++虚拟内存

![image-20240811140857702](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240811140857702.png)

一个程序可由**代码段、数据段和BSS段**组成；**可执行程序**在运行时会出现：堆区和栈区

1. 代码段(Text Segment)

​	又称为文本段，由编译器编译代码后生成的**机器代码**组成

2. 数据段(Data Segment)

​	存储**已初始化的全局变量和静态变量**

```cpp
int globalVar = 10;  // 全局已初始化变量，存储在数据段
static int staticVar = 20;  // 静态已初始化变量，存储在数据段
```

3. BSS(Block Started by Symbol Segment)

​	存储**未初始化的全局变量和静态变量**

```cpp
int globalUninitVar;  // 全局未初始化变量，存储在 BSS 段
static int staticUninitVar;  // 静态未初始化变量，存储在 BSS 段
```

4. 栈

   - 由**编译器自动管理**，当函数被调用时为其分配栈空间，**执行完毕后自动释放栈空间，也是存放局部变量**的地方
   - 主要存储**局部变量、函数参数、返回值**，是一块**连续的空间**
   - **占据空间较小**，一般几兆字节；分配和释放速度快，因为是编译器自动处理
   - 从**高地址向低地址**生长

   ```cpp
   void someFunction() {
       int arr[1000];  // 在栈上分配
   }
   ```

5. 堆

   - 由**程序员手动管理**，使用new分配，delete释放
   - 主要存储**动态分配的对象、大型数据结构**
   - **占据空间大，达到计算机可用的虚拟内存大小**
   - 分配和释放**速度慢**，涉及系统调用和复杂的内存管理操作
   - 从低地址**向高地址方向**生长

   ```cpp
   int* arr = new int[1000];  // 在堆上分配
   delete[] arr;  // 手动释放
   ```

6. 最后还有一个**文件映射区**，位于堆和栈之间。

### C++抽象类（接口）

- C++没有官方定义的接口，用抽象类做接口
- 类中至少有一个函数声明为纯虚函数时，这个类就是抽象类

### 工厂模式

工厂模式包括了**面向接口编程**的思想

- **解耦对象的创建和使用**：使用者不需要关心对象具体是如何创建的，只关注如何使用创建好的对象。好比你去商店购买一台电视，你不需要了解电视的生产过程，只需要知道如何操作和使用它。
- **对象创建的封装**：将对象的创建过程封装在一个工厂类中，客户端无需直接创建对象，降低了对象创建的复杂性。例如，在一个汽车生产的场景中，客户端不需要知道汽车各个零部件的组装细节，只需要向汽车工厂请求一辆汽车即可。

#### 例子

1. 定义一个形状接口 `Shape`

```java
interface Shape {
    void draw();
}
```

2. 创建两个具体的形状类 `Circle` 和 `Rectangle` 实现 `Shape` 接口

```java
class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("绘制圆形");
    }
}

class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("绘制矩形");
    }
}
```

3. 创建工厂类 `ShapeFactory`

```java
class ShapeFactory {
    public Shape createShape(String type) {
        if (type.equalsIgnoreCase("circle")) {
            return new Circle();
        } else if (type.equalsIgnoreCase("rectangle")) {
            return new Rectangle();
        }
        return null;
    }
}
```

4. 在主函数中使用工厂模式创建对象并调用方法

```java
public class Main {
    public static void main(String[] args) {
        ShapeFactory factory = new ShapeFactory();
        Shape shape1 = factory.createShape("circle");
        Shape shape2 = factory.createShape("rectangle");

        shape1.draw();
        shape2.draw();
    }
}
```

通过工厂类 `ShapeFactory` 根据输入的类型创建不同的具体形状对象，使用者只需要与 `Shape` 接口交互，无需关心具体的实现类，提高了代码的灵活性和可扩展性



### 表

#### map

- **有序**
- 基于**红黑树**，value按照key的升序排列，与**key的大小顺序强相关**
- 时间复杂度**O(log n)**
- 数据量不大，对顺序有要求，例如学生的成绩

#### unordered_map

- 无序
- 基于**哈希表**实现，通过哈希函数将映射到特定的存储位置，数据存储位置与**key的哈希值直接相关**不依赖key值比较
- 时间复杂度**o(1) ，最坏情况o(n)**
- 适用于对顺序没有要求，更**注重CRUD操作**，例如网站的用户名-密码登陆校验



### lambda表达式

cpp在代码中的**任何位置定义的小型、无名函数**。

```cpp
[capture list] (param list) {function body};
[capture list] () -> param type {function body};
// -> int 明确指定了这个 lambda 表达式的返回类型是 int 。
auto lambda = []() -> int { return 42; };
```

传参方式：

1. 值捕获：获得的是变量的副本

2. 引用捕获：获得的是引用，修改会影响外部作用域原始变量

3. 混合捕获，[x ,&y]

4. 默认捕获：`[=]` 表示值捕获外部作用域中的所有自动变量。

   `[&]` 表示引用捕获外部作用域中的所有自动变量。

```cpp
// 值捕获
int factor = 2;
auto multiplyByFactor = [factor](int num) { 
    return num * factor; 
    // 尝试修改 factor 会导致编译错误，因为是值捕获，无法修改
    // factor = 3; 
};

// 引用捕获
auto lambda2 = [&b]() {
    std::cout << "Value of b inside lambda2 before modification: " << b << std::endl;
    b = 25;
    std::cout << "Value of b inside lambda2 after modification: " << b << std::endl;
};

multiplyByFactor();
lambda2();
```



### 智能指针

用于管理heap的动态内存管理

```cpp
#include <iostream>
#include <memory>

// 一个简单的类，用于演示智能指针的使用
class MyClass {
public:
    void printMessage() {
        std::cout << "Hello from MyClass!" << std::endl;
    }
};

int main() {
    // 使用 unique_ptr
    std::unique_ptr<MyClass> uniquePtr1 = std::make_unique<MyClass>();
    uniquePtr1->printMessage();

    // 使用 shared_ptr
    std::shared_ptr<MyClass> sharedPtr1 = std::make_shared<MyClass>();
    std::shared_ptr<MyClass> sharedPtr2 = sharedPtr1;

    sharedPtr1->printMessage();
    sharedPtr2->printMessage();

    // 使用 weak_ptr 来解决循环引用问题
    std::shared_ptr<MyClass> obj1 = std::make_shared<MyClass>();
    std::shared_ptr<MyClass> obj2 = std::make_shared<MyClass>();

    // 模拟循环引用
    obj1->otherSharedPtr = obj2;
    obj2->otherSharedPtr = obj1;

    std::weak_ptr<MyClass> weakPtr1 = obj1;
    std::weak_ptr<MyClass> weakPtr2 = obj2;

    // 检查 weak_ptr 是否有效并获取 shared_ptr
    if (std::shared_ptr<MyClass> lockedPtr1 = weakPtr1.lock()) {
        lockedPtr1->printMessage();
    }

    return 0;
}
```

### chrono库

#### 时间间隔 duration

duration表示一段时间间隔，用来记录时间长度，可以表示几秒、几分钟、几个小时的时间间隔

#### 时间点 time point

chrono 库中提供了一个表示时间点的类 time_point

#### 时钟clock

chrono 库中提供了获取当前的系统时间的时钟类，包含的时钟一共有三种：

- system_clock：系统的时钟，系统的时钟可以修改，甚至可以网络对时，因此使用系统时间计算时间差可能不准。
- steady_clock：是固定的时钟，相当于秒表。开始计时后，时间只会增长并且不能修改，适合用于记录程序耗时
- high_resolution_clock：和时钟类 steady_clock 是等价的（是它的别名）。

如果我们通过时钟不是为了获取当前的系统时间，而是进行程序耗时的时长，此时使用 syetem_clock 就不合适了，因为这个时间可以跟随系统的设置发生变化。在 C++11 中提供的时钟类 steady_clock 相当于秒表，只要启动就会进行时间的累加，并且不能被修改，非常适合于进行耗时的统计。

### C++多态

C++多态分为编译时多态（重载）和运行时多态（重写）；Java多态则是指对象多态（运行类型和编译类型可以不一致）和方法多态（重载和重写）

#### 编译时多态

- 在**编译阶段**编译器就能根据调用时提供的参数类型和数量决定调用哪个函数
- 地址早绑定early binding，，根据**指针或引用的声明类型**在编译阶段就确定**函数的调用地址**

```cpp
void print(int num) {
    std::cout << "Integer: " << num << std::endl;
}

void print(double num) {
    std::cout << "Double: " << num << std::endl;
}
```

#### 运行时多态

- 通过**重写虚函数**实现，直接重写成员非虚函数不算体现C++多态性
- 函数地址晚绑定(late binding)，只有在运行时才确定，编译时无需检查对象类型只需检查对象是否支持特征和方法

```cpp
class Base {
public:
    // 定义虚函数
    virtual void virtualMethod() {
        std::cout << "Base::virtualMethod()" << std::endl;
    }
};

class Derived : public Base {
public:
    // 重写虚函数
    void virtualMethod() override {
        std::cout << "Derived::virtualMethod()" << std::endl;
    }
};

int main() {
    // 编译阶段只知道ptr是指向Base类型的指针
    Base* ptr = new Derived();
    ptr->virtualMethod();  // 调用派生类的虚函数实现
    delete ptr;
    return 0;
}
```

### 虚函数

1. 虚函数机制：当一个函数被声明为虚函数时，C++ 的运行时系统会建立一个**虚函数表（Virtual Function Table，简称 VTable）**来管理这些虚函数的地址。当通过基类的指针或引用调用虚函数时，运行时系统会**根据实际对象的类型在虚函数表中查找并调用相应的重写版本**
2. 在子类中重写虚函数的函数**没有显式使用virtual关键字，但仍然也是虚函数**
3. 行为的**灵活性**：重写虚函数允许子类根据自身的特点和需求来**定制**特定的行为。不同的子类可以对同一个虚函数有不同的实现，**使得相同的函数调用在不同的对象上产生不同的效果**，这正是多态性的核心概念
4. 如果子类没有重写父类的非析构虚函数，并且通过父类指针或引用调用该虚函数，那么会调用父类中定义的版本。

```cpp
#include <iostream>

class Base {
public:
    virtual void virtualMethod() {
        std::cout << "Base::virtualMethod()" << std::endl;
    }
};

class Derived : public Base {
public:
    void virtualMethod() override {
        std::cout << "Derived::virtualMethod()" << std::endl;
    }
};

int main() {
    Base* ptr = new Derived();
    ptr->virtualMethod();  // 输出 "Derived::virtualMethod()"，体现多态性
    delete ptr;
    return 0;
}
```

### 析构函数

1. 资源清理：
   - 释放**动态分配的内存**，例如通过new操作符分配的内存
   - **关闭文件、网络连接、数据库连接等资源**
2. 对象销毁的收尾工作：
   - 执行必要的**清理工作**，如对象的状态重置、释放相关的锁
3. 避免内存泄漏
   - 确保在对象不再使用时，其所占用的资源得到正确释放，防止资源的持续占用导致内存泄漏

```cpp
class MyClass {
private:
    int* data;

public:
    MyClass(int size) {
        data = new int[size];
    }

    ~MyClass() {
        delete[] data;  // 在析构函数中释放动态分配的内存
    }
};

class FileHandler {
private:
    std::fstream file;

public:
    FileHandler(const std::string& filename) {
        file.open(filename);
    }
	// 如果子类没有显式实现析构函数则会隐式调用父类析构函数
    ~FileHandler() {
        if (file.is_open()) {
            file.close();  // 在析构函数中关闭文件
        }
    }
};
```

- 为什么Java没有析构函数：
  1. 自动垃圾回收机制：Java 有一个**自动的垃圾回收器**，它负责回收不再被使用的对象所占用的内存。程序员**不需要手动管理内存的释放**，因此也就不太需要像 C++ 那样的显式析构函数来执行特定的清理操作。
  2. 确定性的资源释放问题：在 Java 中，对于像文件、网络连接等资源的管理，通常是通过在**使用完后显式地调用相应的关闭方法来完成**，而不是依赖于析构函数。

### chrono 库

https://subingwen.cn/cpp/chrono/

#### 时间间隔 duration

duration表示一段时间间隔，用来记录时间长度，可以表示几秒、几分钟、几个小时的时间间隔

#### 时间点 time point

chrono 库中提供了一个表示时间点的类 time_point

#### 时钟clock

chrono 库中提供了获取当前的系统时间的时钟类，包含的时钟一共有三种：

- system_clock：系统的时钟，系统的时钟可以修改，甚至可以网络对时，因此使用系统时间计算时间差可能不准。
- steady_clock：是固定的时钟，相当于秒表。开始计时后，时间只会增长并且不能修改，适合用于记录程序耗时
- high_resolution_clock：和时钟类 steady_clock 是等价的（是它的别名）。

如果我们通过时钟不是为了获取当前的系统时间，而是进行程序耗时的时长，此时使用 syetem_clock 就不合适了，因为这个时间可以跟随系统的设置发生变化。在 C++11 中提供的时钟类 steady_clock 相当于秒表，只要启动就会进行时间的累加，并且不能被修改，非常适合于进行耗时的统计。



### Tips

#### explicit关键字

当一个构造函数被声明为 `explicit` 时，它只能用于显式的对象初始化，不能用于隐式的类型转换

```cpp
class MyClass {
public:
    int value;

    explicit MyClass(int v) : value(v) {}
};

MyClass obj(5);  // 显式初始化，合法
MyClass obj = 5;  // 非法，因为构造函数是 explicit
```

#### 头文件

为什么Java不用头文件

1. 编译模型差异：c++编译是按文件进行，编译每个源文件时需要提前知道其他文件中定义的类型、函数等的声明；Java**基于jvm**，**运行时动态加载和链接库**，因此不需要在编译时就知道所有的声明
2. 分离声明和实现：头文件负责存放类、函数、变量等的**声明**，源文件存放**实现**。提高可读性
3. 历史和设计理念：Java强调“**一次编写，到处运行**”的特性和更简洁的代码结构
4. 大型项目中：修改函数的实现，cpp只需**重写编译对应的源文件**，而包含该函数的其他源文件无需重新编译；Java则需要对**整个项目重写编译和打包**

#### 命名规范

##### 变量

- 采用小写字母，多个单词之间用下划线 `_` 连接，例如 `my_variable` 。同python
- 对于类的成员变量，可以添加前缀 `m_` ，如 `m_member_variable` 

##### 函数

- 采用小写字母，多个单词之间用下划线 `_` 连接，例如 `my_function` 。同python
- 对于类的成员函数，与普通函数命名规则相同，或者使用驼峰命名法，如 `myMemberFunction` 

##### 类

- 采用大驼峰命名法，即每个单词的首字母大写，例如 `MyClass`，同Java

##### 常量

- 全部使用大写字母，多个单词之间用下划线 `_` 连接，例如 `MAX_VALUE` 。

##### 命名空间

- 采用小写字母，多个单词之间用下划线 `_` 连接，例如 `my_namespace`

##### 指针和引用

- 如果变量是指针，在变量名前添加 `p_` ，如 `p_pointer` 。
- 如果变量是引用，在变量名前添加 `r_` ，如 `r_reference`

## Linux monitor

在GNU/Linux系统中，/proc是一个位于内存中的伪文件系统(in-memory)



### cpu load

/proc/loadavg中保存了CPU负载的平均值，三列分别表示1分钟、5分钟、15分钟的平均负载，反映了当前系统的繁忙情况

![image-20240808184620159](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808184620159.png)

```
lavg_1 (4.61) 1-分钟平均负载
lavg_5 (4.36) 5-分钟平均负载
lavg_15(4.15) 15-分钟平均负载
```

#### 平均负载

- 单位时间内，系统**处于可运行状态和不可中断状态**的平均进程数，也就是**平均活跃进程数**，或者**单位时间内等待运行的进程数量平均值**，与CPU使用率没有直接关系。它不仅考虑了正在使用 CPU 的进程，还包括**等待 CPU 资源（处于就绪状态）和等待 I/O 等其他资源**的进程。
- CPU使用率指的是**单位时间内CPU处在非空闲态的时间比**，反映了CPU的繁忙程度
- 高平均负载不一定意味着高 CPU 使用率，如果等待的进程主要是在等待 I/O 操作完成，CPU 使用率可能不高。

##### 举例

平均负载=2

- 对4个CPU系统：50%CPU空闲
- 2个CPU系统：100%CPU占用
- 1个CPU系统：**一半的进程竞争不到CPU**

平均负载**最理想的情况=CPU数量**，当平均负载>CPU个数时，说明出现了**过载红温警告警告**



- 如果1分钟平均负载>15分钟平均负载，说明最近1分钟负载在增加，可能是临时性的
- 如果1分钟平均负载<15分钟平均负载，说明最近1分钟负载在减小，过去15负载有很大的负载
- 如果1分钟平均负载接近或超过CPU个数，说明正在发生过载，要想办法优化
- 如果单CPU平均负载为1.73 0.60 7.98，说明过去1分钟有73%超载，过去15分钟有698%超载，整体趋势是系统负载在降低
- 实际生产中，平均负载>70%CPU时，就该分析负载高的问题

### cpustat

- `user(通常缩写为us)`，代表用户态CPU时间，**不包括nice时间，但包括guest时间**
- `nice(ni)`：低优先级用户态CPU时间，即进程的nice值被调整为1-19之间时的CPU时间，nice可以是-20到19，数值越大，优先级越低
- `system(sys)`：内核态CPU时间
- `idle(id)`：空闲时间，不包括等待I/O的时间(iowait)
- `iowait(wa)`：等待I/O的CPU时间
- `irq(hi)`：处理硬中断的CPU时间
- `softirq(si)`：处理软中断的CPU时间
- `steal(st)`：系统运作在虚拟机时，被其他虚拟机占用的CPU时间
- `guest(guest)`：虚拟化运行其他操作系统的时间，即运行虚拟机的时间
- `guest_nice(gnice)`：低优先级运行虚拟机的时间

#### cpu使用率

![image-20240808200344411](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808200344411.png)

=忙碌时间/CPU时间

间隔一段时间**(比如3秒)的2次值，做差后**，再计算出这段时间内的平均CPU使用率

![image-20240808200404137](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808200404137.png)

### cpu softirqs

- `TIMER`：定时中断
- `NET_TX`：网络发送
- `NET_RX`：网络接收
- `SCHED`：内核调度
- `RCU`：RCU锁中，网络接收变化最快

```shell
root@yiga-virtual-machine:/proc# cat /proc/softirqs
                    CPU0       CPU1       CPU2       CPU3       CPU4       CPU5       CPU6       CPU7       CPU8       CPU9       CPU10      CPU11      CPU12      CPU13      CPU14      CPU15      CPU16      CPU17      CPU18      CPU19      CPU20      CPU21      CPU22      CPU23      CPU24      CPU25      CPU26      CPU27      CPU28      CPU29      CPU30      CPU31      CPU32      CPU33      CPU34      CPU35      CPU36      CPU37      CPU38      CPU39      CPU40      CPU41      CPU42      CPU43      CPU44      CPU45      CPU46      CPU47      CPU48      CPU49      CPU50      CPU51      CPU52      CPU53      CPU54      CPU55      CPU56      CPU57      CPU58      CPU59      CPU60      CPU61      CPU62      CPU63      CPU64      CPU65      CPU66      CPU67      CPU68      CPU69      CPU70      CPU71      CPU72      CPU73      CPU74      CPU75      CPU76      CPU77      CPU78      CPU79      CPU80      CPU81      CPU82      CPU83      CPU84      CPU85      CPU86      CPU87      CPU88      CPU89      CPU90      CPU91      CPU92      CPU93      CPU94      CPU95      CPU96      CPU97      CPU98      CPU99      CPU100     CPU101     CPU102     CPU103     CPU104     CPU105     CPU106     CPU107     CPU108     CPU109     CPU110     CPU111     CPU112     CPU113     CPU114     CPU115     CPU116     CPU117     CPU118     CPU119     CPU120     CPU121     CPU122     CPU123     CPU124     CPU125     CPU126     CPU127     
          HI:          0          0          1     199512          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
       TIMER:     575593     590851     962137    1908052          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
      NET_TX:          5          1          2          7          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
      NET_RX:      37873      37159      32595     323769          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
       BLOCK:      76799      21475      55407      29366          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
    IRQ_POLL:          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
     TASKLET:        241          2         28     799154          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
       SCHED:    1494798    1360639    1713100    3154266          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
     HRTIMER:          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
         RCU:     880363     867429     995143    1138718          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
```

查看中断次数

- /proc/softirqs：记录**开机以来软中断**累计次数
- /proc/interrupts：记录**开机以硬中断**累计次数



- 中断是一种**异步的事件处理机制**，与单片机、java的中断机制相同，提高系统并发处理能力
- 为了解决中断处理程序执行过长和中断丢失的问题，Linux将中断处理过程分成了两个阶段
  - 上半部(top half)：用于**快速**处理中断，在中断禁止模式下运行，处理跟**硬件紧密相关或时间敏感的工作**
  - 下半部(bottom half)：用于**延迟**处理上半部未完成的工作，由于中断处理程序需要尽快完成，以减少对系统性能的影响，一些**耗时**的操作会被**推迟到中断下半部（Bottom Half）**处理。常见的中断下半部机制包括**软中断（SoftIRQ）、任务队列（Task Queue）和工作队列（Work Queue）**等。
  - 下半部以**内核线程**的方式执行，并且每个 CPU 都对应

#### 硬中断

 由**外部硬件设备（如键盘**、鼠标、网卡等）产生，通知 CPU 设备需要服务或发生了重要事件。

#### 软中断

由软件指令触发，例如系统调用或异常。

#### 举例

- `Top Half`（上半部）：

​	这部分的中断处理程序需要尽快执行并且要尽可能地简短，主要完成一些**关键和紧急**的任务，例如**保存中断现场、识别中断源**等。

​	例如，当网络数据包到达网卡产生中断时，上半部可能只是简单地将数据包**接收并放入一个缓冲区，然后标记数据包已到达**。

- `Bottom Half`（下半部）：

​	下半部在稍后执行，用于处理那些相对不太紧急或者耗时较长的任务。

​	比如，还是对于网络数据包的处理，下半部可能会对数据包进行**解析、处理协议、将数据传递给上层应用程序等更复杂和耗时的操作**。

​	另一个例子是磁盘 I/O 中断。上半部可能只是确认 I/O 操作完成，将相关信息**记录**下来。而下半部则负责**将数据从磁盘缓冲区传输到用户空间的缓冲区**，或者更新文件系统的相关数据结构。

​	通过将中断处理分为上半部和下半部，可以在保证及时响应中断的同时，又能有效地处理那些复杂和耗时的任务，提高系统的整体性能和响应能力。



#### RCU锁

内核使用的同步机制，解决在多处理器环境下，读操作>>写操作情况下的并发数据访问问题

1. 读端无锁：读操作无任何锁，可以自由并发进行
2. 写操作：先复制修改的数据然后修改副本，然后修改副本，完成后等待时机（grace period，大大的timing）来删除旧数据
3. 内存回收：在grace period期间，确保所有正在进行的读操作完成后才安全的释放旧数据

### meminfo

/proc/meminfo

1. `MemTotal`：系统总的物理内存量。
2. `MemFree`：未被使用的物理内存量。
3. `MemAvailable`：估算的可供新进程使用的物理内存量。
4. `Buffers`：用于块设备（如磁盘）I/O 缓冲区的内存量。
5. `Cached`：用于文件系统缓存的内存量。
6. `SwapCached`：被交换出去但仍在交换缓存中的内存量。
7. `Active`：活跃使用的内存量，通常是最近被使用过的内存。
8. `Inactive`：不活跃使用的内存量，近期未被使用。
9. `Active(anon)`：活跃的匿名内存量（不与文件关联的内存）。
10. `Inactive(anon)`：不活跃的匿名内存量。
11. `Active(file)`：活跃的与文件关联的内存量。
12. `Inactive(file)`：不活跃的与文件关联的内存量。
13. `Unevictable`：不能被回收的内存量。
14. `Mlocked`：被锁定在内存中的内存量。
15. `SwapTotal`：交换分区的总容量。
16. `SwapFree`：交换分区的空闲容量。
17. `Dirty`：等待写回磁盘的脏页内存量。
18. `Writeback`：正在写回磁盘的内存量。
19. `AnonPages`：匿名页的内存量（不与文件关联）。
20. `Mapped`：映射到进程地址空间的文件内存量。
21. `Shmem`：共享内存的使用量。
22. `Slab`：内核 slab 分配器使用的内存量。
23. `SReclaimable`：可回收的 slab 内存量。
24. `SUnreclaim`：不可回收的 slab 内存量。
25. `KernelStack`：内核栈使用的内存量。
26. `PageTables`：页表使用的内存量。
27. `NFS_Unstable`：NFS 不稳定页的内存量。
28. `Bounce`：用于 DMA 映射的内存量。
29. `WritebackTmp`：临时的写回内存量。
30. `CommitLimit`：内存分配的提交限制。
31. `Committed_AS`：已提交的内存量。
32. `VmallocTotal`：虚拟内存的总分配量。
33. `VmallocUsed`：已使用的虚拟内存量。
34. `VmallocChunk`：最大的连续可用虚拟内存块大小。

```shell
yiga@yiga-virtual-machine:~$ cat /proc/meminfo
MemTotal:        3963556 kB
MemFree:          202660 kB
MemAvailable:    2108796 kB
Buffers:          292316 kB
Cached:          1508820 kB
SwapCached:          520 kB
Active:          1377516 kB
Inactive:        1476136 kB
Active(anon):     855068 kB
Inactive(anon):   237160 kB
Active(file):     522448 kB
Inactive(file):  1238976 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       2191356 kB
SwapFree:        2190064 kB
Zswap:                 0 kB
Zswapped:              0 kB
Dirty:               852 kB
Writeback:             0 kB
AnonPages:       1052028 kB
Mapped:           481260 kB
Shmem:             39756 kB
KReclaimable:     433624 kB
Slab:             596084 kB
SReclaimable:     433624 kB
SUnreclaim:       162460 kB
KernelStack:       13216 kB
PageTables:        24524 kB
SecPageTables:         0 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     4173132 kB
Committed_AS:    5052008 kB
VmallocTotal:   34359738367 kB
VmallocUsed:       34760 kB
VmallocChunk:          0 kB
Percpu:           104448 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
FilePmdMapped:         0 kB
Unaccepted:            0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:      173888 kB
DirectMap2M:     2971648 kB
DirectMap1G:     3145728 kB
```

#### free

1.  `total` 表示系统总的内存大小。
2.  `used` 表示已使用的内存大小。
3.  `free` 表示**未使用**的内存大小。
4. `shared` 表示**共享**内存大小。
5. `buff/cache` 表示**缓冲区和缓存**使用的内存大小。
6. `available` 表示可用于**新进程分配**的可用内存大小。

##### 共享内存

共享内存是一种进程间通信（IPC）机制，允许多个进程访问同一块物理内存区域。比如，在一个多媒体处理系统中，多个进程可能需要同时访问和处理同一段视频数据，共享内存可以提供快速的数据访问，提高处理效率。

### net

/proc/net/dev 网络流入流出的统计信息，包括接收包的数量、发送包的数量，发送数据包时的错误和冲突情况等

![image-20240808203424147](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808203424147.png)

- `Receive` 部分的字段：
  - `bytes`：接收到的字节数。
  - `packets`：接收到的数据包数量。
  - `errs`：接收错误的数量。
  - `drop`：接收时丢弃的数据包数量。
  - `fifo`：由于 FIFO 缓冲区溢出而丢弃的数据包数量。
  - `frame`：帧对齐错误的数量。
  - `compressed`：接收到的压缩数据包数量。
  - `multicast`：接收到的多播数据包数量。
- `Transmit` 部分的字段：
  - 含义与 `Receive` 部分类似，但统计的是发送的相关数据。



### stress压测指令

1. `-c` ：指定要产生的 CPU 工作进程的数量。    - 例如：`stress -c 4` 表示启动 4 个 CPU 工作进程。 
2. `-i` ：指定要产生的 I/O 操作进程的数量。
3. `-m` ：指定要产生的内存分配进程的数量。 
4. `-d` ：指定要产生的磁盘写进程的数量。 ` 
5. `-n` ：指定要产生的网络操作进程的数量。
6. `-t` ：指定运行的时间，后面接时间值和时间单位，如 `stress -t 60s` 表示运行 60 秒。
7.  `-v` ：显示详细的信息，包括进程的创建和资源使用情况。 通过组合这些选项，可以模拟不同类型和程度的系统负载，以测试系统在压力下的性能和稳定性。 



















