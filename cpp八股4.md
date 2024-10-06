### 82. 

## 其他问题

### 1. 构造函数、析构函数的执行顺序

> 构造函数：父类构造函数、成员类对象构造函数、子类构造函数；
>
> 析构函数：子类析构函数、成员类对象析构函数、父类析构函数

1. 单继承情况下构造函数和析构函数的执行顺序
   - **构造函数执行顺序**：在单继承关系中，首先执行基类的构造函数，然后再执行派生类的构造函数。这是因为派生类是基于基类构建的，基类部分需要先初始化。
   - 例如，有一个基类`Base`和一个派生类`Derived`，代码如下：

```cpp
   class Base {
   public:
       Base() {
           std::cout << "Base constructor called." << std::endl;
       }
   };
   class Derived : public Base {
   public:
       Derived() {
           std::cout << "Derived constructor called." << std::endl;
       }
   };
   int main() {
       Derived d;
       return 0;
   }
```

- 在`main`函数中创建`Derived`类的对象`d`时，首先会调用`Base`类的构造函数，输出`Base constructor called.`，然后再调用`Derived`类的构造函数，输出`Derived constructor called.`。
- **析构函数执行顺序**：与构造函数的顺序相反。先执行派生类的析构函数，然后再执行基类的析构函数。因为在对象销毁时，派生类部分先被清理，最后才清理基类部分。
- 例如，为上述`Base`和`Derived`类添加析构函数：

```cpp
   class Base {
   public:
       Base() {
           std::cout << "Base constructor called." << std::endl;
       }
       ~Base() {
           std::cout << "Base destructor called." << std::endl;
       }
   };
   class Derived : public Base {
   public:
       Derived() {
           std::cout << "Derived constructor called." << std::endl;
       }
       ~Derived() {
           std::cout << "Derived destructor called." << std::endl;
       }
   };
   int main() {
       Derived d;
       return 0;
   }
```

- 当`main`函数结束，对象`d`销毁时，首先调用`Derived`类的析构函数，输出`Derived destructor called.`，然后调用`Base`类的析构函数，输出`Base destructor called.`。

1. 多继承情况下构造函数和析构函数的执行顺序
   - **构造函数执行顺序**：在多继承中，先执行基类构造函数，按照继承声明的顺序（从左到右）调用基类的构造函数，然后执行派生类的构造函数。
   - 例如，有三个类`Base1`、`Base2`和`Derived`，其中`Derived`继承自`Base1`和`Base2`：

```cpp
   class Base1 {
   public:
       Base1() {
           std::cout << "Base1 constructor called." << std::endl;
       }
   };
   class Base2 {
   public:
       Base2() {
           std::cout << "Base2 constructor called." << std::endl;
       }
   };
   class Derived : public Base1, public Base2 {
   public:
       Derived() {
           std::cout << "Derived constructor called." << std::endl;
       }
   };
   int main() {
       Derived d;
       return 0;
   }
```

- 当创建`Derived`类的对象`d`时，首先调用`Base1`的构造函数，输出`Base1 constructor called.`，然后调用`Base2`的构造函数，输出`Base2 constructor called.`，最后调用`Derived`类的构造函数，输出`Derived constructor called.`。
- **析构函数执行顺序**：与构造函数顺序相反。先执行派生类的析构函数，然后按照继承声明的相反顺序（从右到左）调用基类的析构函数。
- 例如，为上述类添加析构函数：

```cpp
   class Base1 {
   public:
       Base1() {
           std::cout << "Base1 constructor called." << std::endl;
       }
       ~Base1() {
           std::cout << "Base1 destructor called." << std::endl;
       }
   };
   class Base2 {
   public:
       Base2() {
           std::cout << "Base2 constructor called." << std::endl;
       }
       ~Base2() {
           std::cout << "Base2 destructor called." << std::endl;
       }
   };
   class Derived : public Base1, public Base2 {
   public:
       Derived() {
           std::cout << "Derived constructor called." << std::endl;
       }
       ~Derived() {
           std::cout << "Derived destructor called." << std::endl;
       }
   };
   int main() {
       Derived d;
       return 0;
   }
```

- 当`main`函数结束，对象`d`销毁时，首先调用`Derived`类的析构函数，输出`Derived destructor called.`，然后调用`Base2`的析构函数，输出`Base2 destructor called.`，最后调用`Base1`的析构函数，输出`Base1 destructor called.`。

1. 包含对象成员情况下构造函数和析构函数的执行顺序
   - **构造函数执行顺序**：当类包含对象成员时，先执行对象成员的构造函数（按照对象成员声明的顺序），然后执行类自身的构造函数。
   - 例如，有一个类`Outer`包含一个`Inner`类的对象成员：

```cpp
   class Inner {
   public:
       Inner() {
           std::cout << "Inner constructor called." << std::endl;
       }
   };
   class Outer {
   public:
       Outer() {
           std::cout << "Outer constructor called." << std::endl;
       }
       Inner inner;
   };
   int main() {
       Outer o;
       return 0;
   }
```

- 当创建`Outer`类的对象`o`时，首先调用`Inner`类的构造函数，输出`Inner constructor called.`，然后调用`Outer`类的构造函数，输出`Outer constructor called.`。
- **析构函数执行顺序**：与构造函数顺序相反。先执行类自身的析构函数，然后执行对象成员的析构函数（按照对象成员声明的相反顺序）。
- 例如，为上述类添加析构函数：

```cpp
   class Inner {
   public:
       Inner() {
           std::cout << "Inner constructor called." << std::endl;
       }
       ~Inner() {
           std::cout << "Inner destructor called." << std::endl;
       }
   };
   class Outer {
   public:
       Outer() {
           std::cout << "Outer constructor called." << std::endl;
       }
       ~Outer() {
           std::cout << "Outer destructor called." << std::endl;
       }
       Inner inner;
   };
   int main() {
       Outer o;
       return 0;
   }
```

- 当`main`函数结束，对象`o`销毁时，首先调用`Outer`类的析构函数，输出`Outer destructor called.`，然后调用`Inner`类的析构函数，输出`Inner destructor called.`。

### 2. 纯虚函数

- 纯虚函数是在基类中声明的虚函数，它在基类中没有定义具体的函数体，只是通过在函数声明的结尾加上` = 0`来标识。例如，在 C++ 中，`virtual void func() = 0;`就是一个纯虚函数的声明。
- 纯虚函数的存在是为了定义一个接口，**强制派生类去实现这个函数**。它体现了一种抽象的概念，即基类只是规定了派生类应该具有什么样的行为，但本身不提供具体的实现方式。
- 在 C++ 中，**只要一个类包含纯虚函数，它就是抽象类**。纯虚析构函数也不例外。当一个类有纯虚析构函数时，这个类不能被实例化，符合抽象类的定义。

```cpp
   class Vehicle {
   public:
       virtual void move() = 0;
   };
   class Car : public Vehicle {
   public:
       void move() override {
           cout << "The car is moving with engine." << endl;
       }
   };
   class Bicycle : public Vehicle {
   public:
       void move() override {
           cout << "The bicycle is moving with pedals." << endl;
       }
   };
```

### 3. 构造函数不定义为虚函数原因;父类析构函数定义为虚函数原因

> 构造函数若为虚函数，实例化对象时会实现多态进行动态绑定，**先调用了子类的构造函数，倒反天罡**

例如，当创建一个`Dog`类的对象时，编译器知道要调用`Dog`类的构造函数来初始化这个对象，这个过程不需要动态绑定（虚函数的主要特性）。

- **虚函数的语义不符**：虚函数主要用于实现多态，即通过基类指针或引用调用函数时，能够根据实际指向的对象类型来决定调用哪个函数。但在构造函数执行时，对象还没有完全构建好，没有所谓的 “对象实际类型” 和 “基类类型” 的动态切换场景。例如，在构造一个`Derived`类的对象时，它首先要构造基类部分，这个过程是**按照继承层次逐步构建**的，不是基于多态的动态调用场景。

> 父类指针指向子类对象向上转型实现多态时，析构函数非虚会导致调用的析构函数没有实现多态，而是父类的析构函数，先析构父类后没有析构子类，**内存泄漏，倒反天罡**

### 4. 类什么时候析构

1. 生命周期结束被销毁
2. delete

### 5. 构造函数或析构函数可以调用虚函数吗

> 构造函数（静态）里使用动态是不实际的，实现不了多态**，因为对象还没实例化，运行类型没确定**

1. 构造函数中调用虚函数
   - **从语法角度是可以的**：在构造函数中可以调用虚函数，但是这种调用可能不会产生预期的多态效果。
   - **行为分析**：当在构造函数中调用虚函数时，**虚函数的调用是基于对象的静态类型，而不是动态类型**。这是因为在构造函数执行期间，对象还没有完全构造完成，此时对象的动态类型信息还没有完全建立起来。
   - 例如，有一个基类`Base`和一个派生类`Derived`，代码如下：

```cpp
   class Base {
   public:
       Base() {
           print();
       }
       virtual void print() {
           std::cout << "Base::print" << std::endl;
       }
   };
   class Derived : public Base {
   public:
       Derived() {}
       void print() override {
           std::cout << "Derived::print" << std::endl;
       }
   };
   int main() {
       Derived d;
       return 0;
   }
```

- 在这个例子中，当创建`Derived`类的对象`d`时，首先会调用`Base`类的构造函数。在`Base`类的构造函数中调用`print`函数，虽然`print`是虚函数，但是这里会调用`Base`类的`print`函数，输出`Base::print`，而不是`Derived::print`。这是因为在`Base`类构造函数执行时，对象的`Derived`部分还没有被构造，对象被视为`Base`类型。

2. **析构函数中调用虚函数**

- **语法上可以调用**：和构造函数一样，析构函数中也可以调用虚函数，**前提是父类析构函数是虚函数**
- **行为注意事项**：在析构函数中调用虚函数同样可能不会产生完全的多态效果。当析构函数开始执行时，对象的派生部分已经开始被销毁，此时对象的动态类型信息也在逐渐消失。不过，如果析构函数是通过基类指针删除派生类对象**（并且虚函数机制正确设置）**，那么在析构函数中调用虚函数可以产生多态效果。
- 例如，以下是一个可能出现问题的示例：

```cpp
   class Base {
   public:
       ~Base() {
           print();
       }
       virtual void print() {
           std::cout << "Base::print" << std::endl;
       }
   };
   class Derived : public Base {
   public:
       ~Derived() {}
       void print() override {
           std::cout << "Derived::print" << std::endl;
       }
   };
   int main() {
       Base* b = new Derived();
       delete b;
       return 0;
   }
```

- 在这个例子中，当通过`Base`指针`b`删除`Derived`对象时，首先会调用`Base`类的析构函数。在`Base`类的析构函数中调用`print`函数，会输出`Base::print`，而不是`Derived::print`。这是因为在`Base`类析构函数执行时，对象的`Derived`部分已经开始被销毁，对象被视为`Base`类型。为了正确地实现多态销毁，应该将基类的析构函数声明为虚函数。

### 6. 构造函数的几种关键字

1. **default 关键字**
   - **用途**：在 C++ 11 中引入了`default`关键字用于显式指示编译器生成默认构造函数。默认构造函数是一种特殊的构造函数，它可以在没有参数的情况下初始化对象。
   - **使用场景**：当类中定义了其他构造函数（例如带参数的构造函数）后，编译器不会自动生成默认构造函数。如果此时还需要默认构造函数，可以使用`default`关键字来生成。例如：

```cpp
   class MyClass {
       int value;
   public:
       MyClass(int val) : value(val) {}
       MyClass() = default;
   };
```

- 在这个例子中，`MyClass`类首先定义了一个带参数的构造函数`MyClass(int val)`，按照 C++ 的规则，编译器此时不会自动生成默认构造函数。但通过`= default`，明确指示编译器生成默认构造函数，这样就可以像`MyClass obj;`这样创建对象而不需要传递参数。
- **注意事项**：使用`default`关键字生成的默认构造函数的行为与编译器自动生成的默认构造函数行为相同。它会对类中的成员进行默认初始化，例如对于基本类型成员会进行默认值初始化（如`int`类型成员初始化为 0）。

2. **delete 关键字**

- **用途**：`delete`关键字用于**禁止编译器生成某些函数，包括构造函数、析构函数、赋值运算符和其他一些函数**。这在一些特定场景下非常有用，比如当想要阻止某些不适当的操作时。
- **使用场景**：例如，当不希望类的对象被复制时，可以将复制构造函数和赋值运算符声明为`delete`。

```cpp
   class NonCopyable {
   public:
       NonCopyable() = default;
       NonCopyable(const NonCopyable&) = delete;
       NonCopyable& operator=(const NonCopyable&) = delete;
   };
```

- 在这个`NonCopyable`类中，通过`= delete`将复制构造函数和赋值运算符声明为删除状态。这样，如果尝试像`NonCopyable obj1; NonCopyable obj2 = obj1;`（复制构造）或者`NonCopyable obj1; obj1 = obj2;`（赋值）这样的操作，编译器会报错，禁止这些可能导致错误的复制行为。
- **注意事项**：一旦将某个函数声明为`delete`，就不能再使用被删除的函数形式进行操作。这是一种编译期的限制，有助于提高代码的安全性和逻辑性。

3. **= 0（纯虚函数）在构造函数中的误解澄清**

- 在构造函数中不能使用`= 0`来定义纯虚构造函数。因为构造函数的作用是创建对象，而纯虚函数所在的类是抽象类，抽象类是不能实例化的，所以纯虚构造函数没有意义。但是可以在抽象类中有纯虚函数，这些纯虚函数会强制派生类去实现相应的函数，用于定义接口和实现多态等操作，只是构造函数不能是纯虚的。

### 7. 构造函数、拷贝构造函数、赋值运算符区别

- 构造函数

对象不存在，不使用别的对象初始化

- 拷贝构造函数

**对象不存在**，使用别的对象初始化

- 赋值运算符

对象存在，用别的对象给它**赋值**

### 8. 拷贝构造函数、赋值运算符重载的区别

- 对象存不存在，前者原来不存在；后者一直存在
- 拷贝运算符**创建的新对象**是通过**复制已有同类型对象的数据成员**来初始化的；赋值运算符用于将一个对象的数据成员赋值给另一个**已存在**的同类型对象
- 拷贝初始化有些也用=运算符，看起来像=重载，但区别在对象存不存在

```cpp
Student s;
Student s1 = s;    // 调用拷贝构造函数
Student s2;
s2 = s;    // 赋值运算符操作

```

### 9. 虚拟继承

1. 定义
   - 虚拟继承是一种在 C++ 中用于解决多继承情况下菱形继承问题的机制。当一个派生类从多个基类继承，而这些基类又有一个共同的基类时，就可能会出现菱形继承结构。
   - 例如，有类 A，类 B 和类 C 都继承自 A，然后类 D 同时继承自 B 和 C。在这种情况下，如果没有虚拟继承，D 类中会包含两份 A 类的数据成员（一份来自 B，一份来自 C），这可能会导致数据冗余和二义性问题。
2. 虚拟继承的语法和实现方式
   - 在 C++ 中，使用`virtual`关键字来实现虚拟继承。例如，如果 B 和 C 虚拟继承自 A，语法如下：

收起



cpp



复制

```cpp
   class A {
       // A类的成员
   };
   class B : virtual public A {
       // B类的成员
   };
   class C : virtual public A {
       // C类的成员
   };
   class D : public B, public C {
       // D类的成员
   };
```

- 当使用虚拟继承时，编译器会创建一个共享的基类子对象。在上面的例子中，D 类对象中只会有一个 A 类的子对象，而不是两个。这是通过在对象内存布局中设置一个虚基类指针（如果有虚函数还会有虚函数表指针）来实现的。这个虚基类指针指向共享的基类对象部分。

1. 解决的问题 - 数据冗余和二义性
   - **数据冗余方面**：在非虚拟继承的菱形继承中，派生类会包含多个共同基类部分的数据副本。例如，假设 A 类有一个成员变量`int value;`，在非虚拟继承的情况下，D 类对象中会有两个`value`变量，分别来自 B 和 C 对 A 的继承。而通过虚拟继承，D 类对象中只有一个`value`变量，避免了数据冗余。
   - **二义性方面**：非虚拟继承还可能导致成员访问的二义性。例如，在 D 类中访问`A`类的成员时，如果没有虚拟继承，编译器不知道应该选择从 B 继承的 A 部分还是从 C 继承的 A 部分。而虚拟继承解决了这个问题，因为只有一个共享的基类子对象，访问其成员时不会产生二义性。
2. 虚拟继承的性能开销
   - 虚拟继承会带来一定的性能开销。因为需要额外的指针（虚基类指针）来维护共享基类对象的位置，并且在访问虚基类成员时，可能需要通过这个指针进行额外的偏移计算来找到正确的成员位置。
   - 不过，在解决菱形继承问题所带来的好处面前，这种性能开销在很多情况下是可以接受的，特别是在需要合理组织类层次结构和避免数据混乱的场景中。
3. 应用场景示例
   - 以图形绘制系统为例。假设有一个基本图形类`Shape`，有两个派生类`Rectangle`和`Circle`，它们都继承了`Shape`的基本属性（如颜色等）。**然后有一个复杂图形类`ComplexShape`，它可能同时包含`Rectangle`和`Circle`部分，此时`ComplexShape`可以虚拟继承`Shape`，以避免`ComplexShape`对象中出现两份`Shape`类的属性，确保数据的一致性和避免访问二义性。**

```cpp
#include <iostream>
using namespace std;

class A{}
class B : virtual public A{};
class C : virtual public A{};
class D : public B, public C{};

int main()
{
    cout << "sizeof(A)：" << sizeof A <<endl; // 1，空对象，只有一个占位
    cout << "sizeof(B)：" << sizeof B <<endl; // 4，一个bptr指针，省去占位,不需要对齐
    cout << "sizeof(C)：" << sizeof C <<endl; // 4，一个bptr指针，省去占位,不需要对齐
    cout << "sizeof(D)：" << sizeof D <<endl; // 8，两个bptr，省去占位,不需要对齐
}

```

![img](http://oss.interviewguide.cn/img/202205220025114.png)

![img](http://oss.interviewguide.cn/img/202205220022397.png)

### 10. 哪些函数可以是虚函数

**只有非静态成员函数、析构函数**可以是

**其他的静态成员函数、普通函数、构造函数、友元函数、内联函数都不行**

### 11. 模板类和模板函数的区别是什么

![image-20241006163911214](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20241006163911214.png)

### 12. 模板和实现可以不可以写在一个文件里面？为什么？

1. 在 C++ 中可以将模板和实现写在一个文件里
   - **原因在于模板的编译模型**：模板是一种在编译期生成代码的机制。当编译器遇到模板定义时，它不会立即为模板生成代码，而是在实例化模板（即使用模板创建具体类型的对象，如`MyTemplate<int> obj;`）时才生成代码。
   - **包含模型（inclusion model）**：如果把模板的声明和实现分开在不同的文件中，在链接阶段可能会出现问题。因为编译器在编译模板的声明文件（通常是头文件）时，没有足够的信息来生成具体的模板代码。当它在另一个文件（实现文件）中寻找模板的实现细节时，由于模板的编译特性，可能无法正确链接。而将模板和实现写在一个文件里，在包含这个文件的地方，编译器就能够获取完整的模板信息，在需要实例化模板时顺利生成代码。
   - **示例代码**：以下是一个简单的将模板类的声明和实现放在一个`.cpp`文件（虽然通常建议放在头文件，但这里为了说明）中的例子。

```cpp
   // MyTemplate.cpp
   template<typename T>
   class MyTemplate {
   public:
       MyTemplate(T val) : value(val) {}
       T getValue() const {
           return value;
       }
   private:
       T value;
   };
   int main() {
       MyTemplate<int> obj(5);
       return obj.getValue();
   }
```

- 在这个例子中，模板类`MyTemplate`的声明和实现都在一个文件中，当在`main`函数中实例化`MyTemplate<int>`时，编译器能够顺利地生成相应的代码。

1. 不分开的缺点及一些替代方法
   - **缺点**：把模板和实现放在一个文件中可能会使代码看起来不够整洁，尤其是对于大型项目和复杂的模板代码。并且如果多个源文件都需要使用这个模板，可能会导致代码的重复包含，增加编译时间和代码体积。
   - **替代方法 - 显式实例化**：可以使用显式实例化来将模板的声明和实现分开。在模板的实现文件中，对可能用到的模板类型进行显式实例化声明。例如，在一个头文件`MyTemplate.h`中声明模板：

```cpp
   template<typename T>
   class MyTemplate {
   public:
       MyTemplate(T val) : value(val) {}
       T getValue() const;
   private:
       T value;
   };
```

- 然后在实现文件`MyTemplate.cpp`中实现模板方法并进行显式实例化：

```cpp
   #include "MyTemplate.h"
   template<typename T>
   T MyTemplate<T>::getValue() const {
       return value;
   }
   // 显式实例化
   template class MyTemplate<int>;
```

- 这样，在其他文件中包含`MyTemplate.h`头文件并使用`MyTemplate<int>`时，编译器能够正确链接到已经显式实例化的模板代码。不过这种方法需要对可能用到的所有模板类型进行显式实例化，对于模板参数类型较多的情况可能会比较繁琐。

### 13. 将字符串”hello world“从开始到打印到屏幕的全过程

![image-20241006165323172](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20241006165323172.png)

### 14. C++的多继承有什么优点

1. 功能更强大的代码复用
   - 在 C++ 的多继承中，一个类可以从多个不同的基类继承属性和方法，这使得代码复用的程度更高。例如，假设有一个`Drawable`类，它提供了绘制对象的功能；还有一个`Serializable`类，用于实现对象的序列化。在 C++ 中，一个`ComplexShape`类可以同时继承自`Drawable`和`Serializable`，这样`ComplexShape`类的对象既能被绘制又能被序列化，直接复用了两个基类的功能。
   - 而在 Java 中，虽然有接口来实现类似的功能复用，但接口只能定义方法签名，没有像 C++ 多继承那样可以直接继承基类的实现部分。例如，Java 的接口不能包含成员变量（除了静态常量）和默认的方法实现（Java 8 之前），所以在某些情况下复用程度相对较低。
2. 灵活的类层次结构设计
   - C++ 的多继承允许创建更复杂和灵活的类层次结构。例如，在游戏开发中，可以有一个`Character`类，它继承自`PhysicalObject`（用于处理物理碰撞等物理属性）和`AIBehavior`（用于实现人工智能行为）。这种继承方式可以更好地模拟现实世界中对象的多种属性组合，通过多继承将不同方面的功能组合到一个类中，使得类的设计更贴近实际需求。
   - 相比之下，Java 主要通过单继承和接口实现来构建类层次结构。虽然这种方式也很灵活，但在一些需要同时继承多个具体实现类的场景下，C++ 的多继承更具优势。不过 Java 的接口机制也有其优点，如可以**实现多接口来定义多种行为规范，避免了 C++ 多继承可能带来的菱形继承等复杂问题**。

### 15. 为什么拷贝构造函数必须传引用不能传值

1. 避免无限递归调用
   - 如果拷贝构造函数的参数是按值传递的，那么在调用拷贝构造函数时，为了将实参传递给形参，就需要对实参进行拷贝。**而这个拷贝过程又会调用拷贝构造函数本身，这样就会陷入无限递归调用的状态。**
   - 例如，假设有一个类`MyClass`，如果拷贝构造函数的参数是按值传递的，像这样定义：

```cpp
   class MyClass {
   public:
       MyClass(const MyClass myObj) {
           // 拷贝操作
       }
   };
```

- 当尝试创建一个`MyClass`对象并使用另一个`MyClass`对象来初始化它时，例如`MyClass obj1; MyClass obj2 = obj1;`，在调用`MyClass`的拷贝构造函数（`MyClass(const MyClass myObj)`）时，为了将`obj1`传递给`myObj`，编译器会尝试调用拷贝构造函数来创建`myObj`，这个新的调用又会触发相同的操作，导致无限递归，最终导致栈溢出。

2. 引用传递的优势

- **效率高**：引用传递不需要像值传递那样创建一个新的对象副本。当传递一个大型对象时，值传递会带来较大的开销，因为需要复制整个对象的内容。**而引用传递只是传递对象的引用，相当于传递对象的地址，开销较小。**
- **语义合适**：拷贝构造函数的目的是创建一个新对象，这个新对象是另一个对象的副本。引用传递能够很好地实现这个目的，因为通过引用可以直接访问被引用的对象，从而进行成员的复制操作，以完成新对象的创建。例如，在正确定义的拷贝构造函数中：

```cpp
   class MyClass {
   public:
       MyClass(const MyClass& myObj) {
           // 进行成员的复制操作，例如：
           this->member1 = myObj.member1;
       }
       int member1;
   };
```

- 这里通过引用`myObj`可以方便地访问源对象的成员，并将其复制到新创建的对象中，实现了拷贝构造的功能。

### 16. 静态函数能定义为虚函数吗？

1. 没意义，又不实现多态
2. **虚函数调用关系：this->vptr->vtable->虚函数，静态函数没有this指针**

### 17. 虚函数代价

1. 带有虚函数的**类**，会产生**虚函数表**用来存储指向虚成员函数的指针
2. 带有虚函数的每一个**对象**，会有**一个指向虚表的指针**，增加对象的空间

### 18. 虚函数表、虚函数表指针创建的timing

1. **虚函数表的初始化时机**
   - **编译阶段确定虚函数表的基本结构**：在**编译阶段**，编译器会为包含虚函数的类构建虚函数表的基本框架。它会确定虚函数的地址并按照一定的顺序将其放入虚函数表中。对于基类，编译器会把基类的虚函数地址放入虚函数表；对于派生类，如果派生类重写了基类的虚函数，编译器会将派生类重写后的虚函数地址替换虚函数表中相应基类虚函数的位置。
   - **链接阶段完成虚函数表的填充（在一定程度上）**：在链接阶段，编译器和链接器一起工作来进一步完善虚函数表。特别是当涉及到多个编译单元（不同的`.cpp`文件）时，需要确保虚函数表中的函数地址能够正确地链接到实际的函数实现。例如，如果一个虚函数的实现位于另一个编译单元中，链接器会将虚函数表中的相应条目与正确的函数地址进行链接。
2. **虚函数表指针的初始化时机**
   - **对象构造时初始化虚函数表指针**：当**创建一个包含虚函数的类的对象时**，在对象的构造函数执行期间，会**初始化虚函数表指针。这个指针会被设置为指向该类对应的虚函数表**。
   - **具体过程**：在对象构造的早期阶段（通常是在执行基类构造函数之前，对于非虚继承情况），编译器会插入代码来初始化虚函数表指针。例如，在一个简单的单继承场景中，假设有基类`Base`（包含虚函数）和派生类`Derived`，当创建`Derived`类的对象时，首先会为这个对象分配内存空间，然后在执行`Derived`类的构造函数之前，会将对象中的虚函数表指针设置为指向`Derived`类的虚函数表。这个过程是由编译器自动完成的，程序员一般不需要显式地操作。
   - 对于继承情况的特殊性
     - **单继承**：在单继承场景下，派生类对象的虚函数表指针初始化相对简单。它直接指向派生类自己的虚函数表，这个虚函数表可能包含对基类虚函数的重写。例如，如果`Derived`类重写了`Base`类的某个虚函数`func`，那么`Derived`类对象的虚函数表中`func`对应的指针会指向`Derived::func`。
     - **多继承和虚继承**：在多继承和虚继承情况下，虚函数表指针的初始化会更复杂。在多继承中，可能会有多个虚函数表指针（取决于继承的层次和方式），每个指针负责管理从不同基类继承的虚函数关系。在虚继承中，虚函数表和虚函数表指针的初始化还需要考虑虚基类的因素，以确保虚基类的虚函数能够被正确地访问和重写，并且避免数据冗余和二义性等问题。