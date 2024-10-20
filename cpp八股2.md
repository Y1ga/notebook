### 42. 值传递、指针传递、引用传递的区别、使用场景和效率

1. 值传递
   - 区别
     - 在值传递中，函数接收的是**实参的一个副本**。这意味着函数内部对形参的任何修改都不会影响到实参。例如，当传递一个基本数据类型（如`int`、`double`等）时，会把这个数据的值复制一份给函数的形参。
   - 使用场景
     - 当不需要修改原始数据，只是使用数据进行计算或者其他不改变原始数据的操作时，适合使用值传递。例如，一个计算两个数乘积的函数：

```cpp
int multiply(int a, int b) {
    return a * b;
}
```

在这里，函数只是使用`a`和`b`进行计算，不需要修改传入的参数的值，所以值传递是合适的。

- 效率
  - 对于基本数据类型，值传递的效率通常较高，因为复制一个基本数据类型的值（如`int`、`double`等）的开销相对较小。但**是，当传递大型的结构体或者对象时，由于需要复制整个数据结构，可能会导致较大的开销，特别是在频繁调用函数的情况下**。

2. 指针传递

- 区别
  - 指针传递是把实参的地址传递给函数的形参。函数内部通过这个地址来访问和修改实参所指向的数据。这意味着在函数内部对形参所指向的数据进行修改会影响到实参。例如，`int* p`是一个指向`int`类型的指针，通过`p`可以访问和修改它所指向的`int`值。
- 使用场景
  - 当需要在**函数内部修改原始数据**，并且可能需要在多个函数之间共享和修改这些数据时，指针传递是很有用的。例如，一个函数用于交换两个整数的值：

```cpp
void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

在这个例子中，通过传递指针，可以在函数内部直接修改实参所指向的整数的值。

- 效率
  - 传递指针本身的开销相对较小，**因为指针的大小通常是固定的（在 32 位系统中通常是 4 字节，在 64 位系统中通常是 8 字节）**。但是，需要注意指针的安全性，因为错误地使用指针可能导致程序崩溃或者出现难以调试的错误。

3. 引用传递

- 区别
  - 引用传递实际上是给实参起了一个别名，函数内部对形参的操作就是对实参的操作。引用在语法上看起来像普通的变量使用，但实际上它和被引用的变量是同一个实体。例如，`int& r = a`，`r`就是`a`的引用，对`r`的操作就是对`a`的操作。
- 使用场景
  - 引用传递和指针传递类似，适用于需要修改原始数据的情况。但是，引用传递在语法上更加简洁，对于一些复杂的数据类型，使用引用可以避免指针操作的复杂性。例如，一个函数用于修改一个对象的属性：

```cpp
class MyClass {
public:
    int value;
};
void modifyObject(MyClass& obj) {
    obj.value = 10;
}
```

在这里，通过引用传递对象`obj`，可以方便地在函数内部修改对象的属性。

- 效率
  - 引用传递在效率上和指针传递类似，**因为引用在底层实现上通常也是通过指针来实现的**。对于基本数据类型和大型数据结构，引用传递的效率和指针传递差不多，**不过引用传递在语法上更安全、更直观，减少了指针操作可能带来的错误**。

### 43. 静态变量什么时候初始化

1. 静态局部变量
   - 对于**静态局部变量，初始化是在程序执行第一次到达其定义处时进行的**。在整个程序的生命周期内，它只会被初始化一次。
   - 例如：

```cpp
void func() {
    static int count = 0;
    // 第一次调用func函数时，count被初始化为0
    count++;
    std::cout << "Count: " << count << std::endl;
}
```

当`func`函数第一次被调用时，`count`会被初始化为 0。之后每次调用`func`函数，`count`不会再次初始化，而是会保留之前的值并继续使用。

2. 静态成员变量（类内）

- 静态成员变量必须在类外进行初始化，初始化的时间是在程序开始执行，**进入`main`函数之前**。这是因为静态成员变量属于整个类，而不是某个对象，它的生命周期从程序开始到程序结束。

例如：

```cpp
class MyClass {
public:
    static int shared_variable;
};
// 在类外初始化静态成员变量
int MyClass::shared_variable = 0;
```

这个初始化过程在程序启动阶段完成，这样在任何对象被创建或者任何函数调用之前，静态成员变量就已经准备好了。并且无论创建多少个类的对象，这个静态成员变量只有一份副本，在整个程序运行过程中都存在。

3. 静态全局变量

- 静态全局变量的初始化也是在程序开始执行，**进入`main`函数之前完成**。它的作用域仅限于定义它的文件，和普通的全局变量不同，它不能被其他文件访问（除非通过一些特殊的机制，如函数指针等间接访问）。

例如：

```cpp
// file1.cpp
static int private_variable = 10;
```

这里的`private_variable`在程序启动阶段就会初始化，并且只在`file1.cpp`内部有效。这种初始化方式可以保证变量在文件内部的函数使用之前就已经准备好了合适的值。

### 44. 什么是成员列表初始化

1. 基本语法
   - 在构造函数的定义中，使用冒号（:）后跟成员变量的初始化列表来实现。其一般形式为：

```cpp
    class ClassName {
    public:
        ClassName(parameters) : member1(value1), member2(value2),... {
            // 构造函数的其他代码
        }
    private:
        Type1 member1;
        Type2 member2;
        //... 其他成员变量
    };
```

- 其中，`ClassName` 是类的名称，`parameters` 是构造函数的参数列表，`member1`、`member2` 等是类的成员变量，`value1`、`value2` 等是成员变量的初始值。

1. 使用场景和优势
   - **初始化 const 成员变量和引用类型成员变量**：**因为常量只能初始化不能赋值，引用必须在定义的时候初始化并且不能重新赋值，所以这两种类型的成员变量必须在初始化列表中进行初始化**。例如：

```cpp
    class MyClass {
    public:
        MyClass(int x) : constVar(x), refVar(constVar) {}
    private:
        const int constVar;
        int& refVar;
    };
```

- **初始化成员变量时需要调用其他类的构造函数**：**如果类的成员变量是另一个类的对象，且该对象所属的类没有无参构造函数，那么必须在初始化列表中调用该类的带参构造函数来初始化成员变量**。例如：

```cpp
    class OtherClass {
    public:
        OtherClass(int num) {}
    };

    class MyClass {
    public:
        MyClass() : otherObj(10) {}
    private:
        OtherClass otherObj;
    };
```

- **提高效率**：对于类类型的成员变量，使用初始化列表可以直接调用成员变量的构造函数进行初始化，**避免了先调用默认构造函数再赋值的过程**，提高了代码的执行效率。
  - **初始化顺序**：**成员变量的初始化顺序与它们在类中声明的顺序有关，而与初始化列表中的顺序无关**。例如：

```cpp
class MyClass {
public:
    MyClass(int x) : b(x), a(b) {}
private:
    int a;
    int b;
};
```

在这个例子中，`a` 的初始化依赖于 `b`，但按照成员变量的声明顺序，`a` 会先被初始化，此时 `b` 还未初始化，所以会导致不可预料的结果。正确的做法应该是先初始化 `b`，再初始化 `a`。

### 45. 从汇编层解释一下引用

1. 引用的基本概念回顾
   - 在 C++ 中，引用是一个别名，它提供了一种访问变量的替代方式。例如，`int a = 10; int& ra = a;`，这里`ra`就是`a`的引用，对`ra`的操作等价于对`a`的操作。
2. 汇编层面的解释
   - 变量的内存布局
     - 当定义一个变量（如`int a = 10;`）时，在内存中会为`a`分配一个存储单元（假设是 4 字节，在 32 位系统中用于存储`int`类型）。这个存储单元有一个内存地址，例如`0x1000`。
   - 引用的实现原理
     - 当创建一个引用（如`int& ra = a;`）时，从汇编角度看**，`ra`实际上并不占用新的独立的内存空间来存储`a`的值**。`ra`只是存储了`a`的内存地址（在这个例子中也是`0x1000`），并且**编译器会确保通过`ra`进行的操作都直接作用于`a`的内存地址所指向的存储单元**。
   - 引用的操作在汇编中的体现
     - 假设我们有以下代码：

```cpp
int a = 10;
int& ra = a;
ra++;
```

- 在汇编层面，

  ```
  ra++
  ```

  这个操作可能会被翻译成类似于下面的指令（这里是简化的示意，实际的汇编指令因处理器架构和编译器而异）：

  - 首先，将`ra`（也就是`a`的内存地址）加载到一个寄存器中，假设是`eax`寄存器，指令可能类似于`mov eax, [0x1000]`（这里`[0x1000]`表示内存地址为`0x1000`的存储单元）。
  - 然后，对寄存器中的值进行加 1 操作，指令类似于`add eax, 1`。
  - 最后，将寄存器中的值写回到`a`的内存地址所对应的存储单元，指令类似于`mov [0x1000], eax`。

- 与指针的对比（从汇编角度）

  - 指针也可以用来访问变量，但指针本身是一个独立的变量，它在内存中有自己的存储单元。例如，`int* p = &a;`，`p`有自己的内存地址（假设是`0x2000`），并且这个存储单元中存储的是`a`的内存地址（`0x1000`）。
  - 当通过指针进行操作（如`(*p)++`）时，汇编指令可能会多一些间接访问的步骤。**首先要从`p`的内存地址（`0x2000`）中取出`a`的内存地址（`0x1000`）**，然后再按照上述引用操作的步骤进行对`a`的操作。所以从汇编角度看，**引用在操作变量时相对指针更加直接，因为它本身就是变量的别名，不需要额外的间接访问步骤来获取变量的内存地址。**

### 46. delete p delete [] p allocator

1. 内存释放和析构函数调用

   - 当`p`是通过`new`操作符创建的单个对象的指针时，使用`delete p`来释放该对象所占用的内存。它首先会调用对象的析构函数（如果对象所属的类有析构函数），用于清理对象在生命周期内占用的资源，比如释放对象内部动态分配的其他内存等。然后，`delete`会将对象所占用的内存返回给自由存储区（free store），使得这部分内存可以被后续的`new`操作重新分配。

   - **示例**：

```cpp
class MyClass {
public:
    int* data;
    MyClass() {
        data = new int;
    }
    ~MyClass() {
        delete data;
    }
};
MyClass* p = new MyClass;
// 使用对象p
delete p; 
```

在这个例子中，`delete p`会先调用`p`所指向的`MyClass`对象的析构函数，析构函数中又会释放`MyClass`对象内部的`data`指针所占用的内存，最后释放`MyClass`对象本身占用的内存。

2. delete[] p

**动态数组内存释放和元素析构函数调用**

- 当`p`是通过`new[]`操作符创建的对象数组的指针时，需要使用`delete [] p`来正确地释放内存。它会对数组中的**每个元素依次调用析构函数**（如果元素所属的类有析构函数），然后将整个数组所占用的内存返回给自由存储区。这是因为**`new[]`分配的是连续的内存空间来存储数组元素，并且记录了数组的大小等信息（这个信息是编译器和运行时环境内部维护的）**，`delete []`能够正确地利用这些信息来完成内存释放和元素析构的操作。

- **示例**：

```cpp
class MyClassArray {
public:
    int value;
    ~MyClassArray() {}
};
MyClassArray* pArray = new MyClassArray[5];
// 使用对象数组pArray
delete [] pArray;
```

在这里，`delete [] pArray`会对`pArray`所指向的 5 个`MyClassArray`元素逐个调用析构函数（在这个例子中析构函数为空操作），然后释放整个数组所占用的内存。

3. `allocator`的作用

- 自定义内存分配和释放策略
  - `allocator`是 C++ 标准库中的一个工具，它提供了一种更灵活的内存分配和释放机制，与`new`和`delete`有所不同。它允许将**对象的内存分配和对象的构造分离开来**，同样也可以**将对象的析构和内存释放分离开来**。这在一些需要高效管理内存或者需要特殊内存分配策略的场景中非常有用。
- 内存分配
  - `allocator`可以通过`allocate`方法来分配内存。例如，`std::allocator<int> alloc; int* p = alloc.allocate(10);`会分配能容纳 10 个`int`类型元素的内存空间，但此时这些内存空间中的元素尚未构造，它们只是一块原始的内存区域。
- 对象构造
  - 可以使用`allocator`的`construct`方法在已分配的内存上构造对象。例如，`alloc.construct(p, 5);`会在`p`所指向的内存位置构造一个`int`值为`5`的对象。
- 对象析构
  - 当需要销毁对象时，可以使用`allocator`的`destroy`方法。例如，`alloc.destroy(p);`会调用`p`所指向的对象的析构函数（如果有）来销毁对象。
- 内存释放
  - 最后，使用`allocator`的`deallocate`方法来释放内存。例如，`alloc.deallocate(p, 10);`会释放之前分配的能容纳 10 个`int`类型元素的内存空间。不过要注意，在使用`deallocate`之前，必须确保已经通过`destroy`方法销毁了所有在这块内存上构造的对象。

### 47. new和delete的实现原理

1. new 的实现原理
   - 基本数据类型的内存分配
     - 当使用`new`操作符来分配基本数据类型（如`int`、`double`等）的内存时，编译器会将`new`表达式转换为对相应的内存分配函数的调用。**在底层是malloc，这些函数通常会调用操作系统提供的内存分配机制（如`brk`、`mmap`等系统调用）来从堆（heap）或者自由存储区（free store）获取足够大小的内存块。**这个大小是由编译器根据数据类型自动计算的，例如`new int`会分配`sizeof(int)`大小的内存。
   - 对象的内存分配和构造函数调用
     - **对于类对象，`new`首先会分配足够的内存来容纳该对象**，这个内存大小包括对象的数据成员、可能的虚函数表指针（如果是有虚函数的类）等所占用的空间。分配内存的过程和基本数据类型类似，也是通过底层的内存分配机制。然后，`new`会调用对象的构造函数来初始化对象。这个调用是由编译器在生成的代码中插入相应的指令来实现的，以确保对象在被使用之前处于一个合适的初始化状态。
2. delete 的实现原理及内存大小的确定
   - 对象的析构函数调用和内存释放
     - 当使用`delete`来释放一个对象时，编译器会首先检查指针是否为`NULL`，**如果是`NULL`，则不进行任何操作**。如果指针非`NULL`，`delete`会先调用对象的析构函数（如果对象所属的类有析构函数）。这个调用过程是通过编译器在生成的代码中查找对象的类型信息，找到对应的析构函数地址，然后跳转到该地址执行析构函数代码来实现的。
     - 在析构函数执行完毕后，`delete`需要释放对象所占用的内存。对于单个对象，编译器在分配内存时会记录一些关于内存块的信息，这些信息可能包括内存块的大小等。一种常见的实现方式是在对象的内存块头部或者其他位置存储额外的管理信息。例如，在某些编译器的实现中，**内存块头部可能会存储内存块的大小、是否是数组等标记信息**。当`delete`操作符执行时，它可以通过读取这些存储在内存块头部的信息来确定要释放的内存大小，然后将内存返回给操作系统或者内存池等管理机制。
   - 数组对象的特殊处理（`delete []`）
     - 对于通过`new[]`分配的数组，`delete []`的实现会更加复杂一些。在分配数组内存时，编译器除了分配足够的内存来存储数组元素外，还会**存储一些额外的信息，比如数组的大小**。当执行`delete []`时，它首先会根据存储的数组大小信息，**对数组中的每个元素逐个调用析构函数**（如果元素所属的类有析构函数）。然后，**通过读取存储的内存块大小信息，将整个数组所占用的内存释放回内存管理系统**。这些存储数组大小等信息的位置和方式因编译器和实现而异，有些可能在数组头部，有些可能通过其他复杂的内存管理结构来记录。

### 49. malloc和free的区别

1. malloc 的底层原理
   - 内存分配策略
     - **空闲链表管理**：malloc 通常通过**维护一个空闲链表（free list）**来管理堆内存。堆内存被看作是由**一系列大小不同的内存块组成**。当程序第一次调用 malloc 时，操作系统会为进程分配一大块连续的内存作为堆空间。这些内存块可以处于**已分配状态**（被程序中的变量或数据结构占用）或空闲状态。空闲链表中的每个节点代表一个空闲的内存块，节点中记录了该内存块的大小、起始地址等信息。
     - **内存块的分配方式**：当 malloc 需要分配一定大小的内存时，它会**遍历空闲链表，寻找一个足够大的空闲内存块**。如果找到的空闲内存块大小正好等于所需要的大小，就直接将这个内存块从空闲链表中移除，并返回给调用者。如果找到的空闲内存块比所需的大，就会将这个大的内存块分割成两部分：一部分是刚好满足需求的内存块，将其返回给调用者；**另一部分是剩余的空闲内存块，会更新其大小等信息并重新插入空闲链表**。
   - 内存对齐考虑
     - 为了提高内存访问效率，malloc 分配的内存地址通常会按照一定的对齐规则进行对齐。例如，对于大多数系统，要求分配的内存地址是某个字节数（如 8 字节或 16 字节）的倍数。这是因为处理器在访问内存时，以字（word）为单位进行读取效率更高。当 malloc 计算所需内存大小时，会根据数据类型和系统的对齐要求，向上调整请求的内存大小。比如，**如果请求分配一个`int`类型（假设`sizeof(int)=4`）的内存，并且系统要求内存地址是 8 字节对齐，那么 malloc 可能会实际分配 8 字节的内存，以满足对齐要求**。
   - 系统调用与内存扩展
     - 如果**空闲链表中没有足够大的空闲内存块来满足分配请求**，malloc 可能会通过系统调用（如`brk`或`mmap`）向操作系统请求更多的内存。`brk`是一种比较简单的方式，它通过移动进程的堆顶指针（program break）来扩展堆内存空间。`mmap`则更灵活，它可以将**文件或者匿名内存区域映射到进程的虚拟地址空间**，并且可以用于分配大的内存块。当通过系统调用获得新的内存后，这些新的内存块会被添加到空闲链表中，然后再进行内存分配的操作。
2. **free 的底层原理**
   - 内存块回收与链表更新
     - 当调用 free 函数释放内存时，它会**将所释放的内存块重新插入空闲链表**。首先，free 会检查要释放的内存块的边界信息（这些边界信息在 malloc 分配内存时可能已经记录在内存块头部或者其他位置），以确保内存块的完整性。然后，它会根据内存块的大小和起始地址，将这个内存块插入到空闲链表中的合适位置。插入的过程可能涉及到对空闲链表的重新排序或者合并操作。
   - 内存块合并
     - 如果要释放的内存块与相邻的空闲内存块在物理地址上是连续的，free 会将它们合并成一个更大的空闲内存块。这是通过检查相邻内存块的状态（是否空闲）来实现的。例如，**如果释放的内存块 A 的末尾地址与空闲内存块 B 的起始地址是连续的，并且 B 是空闲的，那么就会将 A 和 B 合并成一个新的空闲内存块，其大小为 A 和 B 的大小之和**。这种合并操作可以减少空闲链表中的节点数量，提高内存管理的效率，**避免内存碎片化**。

### 50. malloc、realloc、calloc的区别

1. `calloc`函数
   - 功能与基本用法
     - `calloc`的功能也是在堆中分配内存，其函数原型为`void* calloc(size_t num, size_t size)`。它与`malloc`的主要区别在于，`calloc`**会将分配的内存空间初始化为全 0**。例如，`int* p = (int*)calloc(5, sizeof(int));`会分配能存储 5 个`int`类型数据的内存空间，并且这块内存中的所有`int`值都会被初始化为 0。
   - 用途与优势
     - 当需要分配内存用于**存储一些数据结构（如数组）**，并且希望这些数据**一开始就被初始化为 0** 时，`calloc`就非常有用。这可以**避免在使用内存之前手动逐个初始化数据元素的麻烦，同时也能保证数据的初始一致性。**
2. `realloc`函数
   - 功能与基本用法
     - `realloc`**用于重新分配之前通过`malloc`或`calloc`分配的内存空间大小**。其函数原型为`void* realloc(void* ptr, size_t size)`，其中`ptr`是之前分配的内存块的指针，`size`是重新分配后的大小。如果`ptr`为`NULL`，`realloc`的功能等价于`malloc`。**例如，假设已经通过`malloc`分配了一个能存储 3 个`int`类型数据的内存块`int* p = (int*)malloc(3 * sizeof(int));`，如果想要将其扩展为能存储 5 个`int`类型数据的内存块，可以使用`p = (int*)realloc(p, 5 * sizeof(int));`。**
   - 内存调整方式
     - 当重新分配内存时，`realloc`会**尽量在原内存块的基础上进行扩展或者收缩**。如果原内存块后面有足够的空闲空间来满足新的大小要求，`realloc`会直接在原内存块上进行扩展，并且返回原指针（这种情况下内存中的数据会保持不变）。**如果原内存块后面没有足够的空间，`realloc`可能会在内存的其他位置找到一块足够大的空间，将原内存块中的数据复制到新的内存块中，然后释放原内存块，并返回新的内存块的指针**。**如果无法找到合适的新内存块，`realloc`会返回`NULL`，并且原内存块不会被释放。在这种情况下，之前指向原内存块的指针仍然有效，但不能再通过这个指针访问内存了，因为内存已经被标记为可能会被重新分配。所以在使用`realloc`后，一定要检查返回值是否为`NULL`。**



### 51. 类成员的初始化方式？构造函数的执行顺序？为什么用成员初始化列表会快一些

1. 类成员的初始化方式
   - 默认初始化
     - 如果类成员是基本数据类型（如`int`、`double`等），在没有进行任何显式初始化的情况下，编译器会进行默认初始化。**对于基本数据类型，其默认初始化的值是不确定的**。例如：

```cpp
class MyClass {
public:
    int num;
};
MyClass obj;
// obj.num的值是不确定的
```

- 就地初始化（C++11 及以上）
  - 在 C++11 及以后的版本中，可以在**类定义中直接对成员变量进行初始化。这种方式简单直接，适用于成员变量有明确初始值的情况**。例如：

```cpp
class MyClass {
public:
    int num = 10;
};
MyClass obj;
// obj.num的值为10
```

- 构造函数初始化
  - 通过构造函数来初始化成员变量是最常见的方式。可以在构造函数的函数体中对成员变量进行赋值操作。例如：

```cpp
class MyClass {
public:
    int num;
    MyClass() {
        num = 20;
    }
};
MyClass obj;
// obj.num的值为20
```

- 成员初始化列表初始化
  - 这是在构造函数定义中使用冒号（:）开头的初始化列表来初始化成员变量。它主要用于初始化**常量成员变量、引用成员变量以及需要调用其他类构造函数的成员变量**。例如：

```cpp
class MyClass {
public:
    const int const_num;
    int& ref_num;
    MyClass(int n) : const_num(n), ref_num(const_num) {}
};
```

1. 构造函数的执行顺序

   - 基类构造函数先执行
     - 如果一个类继承自其他类，那么在创建派生类对象时，首先会调用基类的构造函数。先执行基类构造函数的初始化部分，包括基类成员变量的初始化等操作。这是因为**派生类对象是基于基类对象构建的，需要先确保基类对象处于一个合适的初始化状态**。
   - 成员对象构造函数按声明顺序执行
     - 对于类中的成员对象，其构**造函数的执行顺序是按照成员在类中声明的顺序，而不是在初始化列表中的顺序**。在基类构造函数执行完毕后，会依次调用类中的成员对象的构造函数来初始化这些成员对象。
   - 派生类构造函数最后执行
     - 在基类构造函数和成员对象构造函数都执行完毕后，**最后执行派生类自身的构造函数**，包括在派生类构造函数体中的代码以及对剩余成员变量（如果有）的初始化操作。

2. 为什么成员初始化列表会快一些

   - 减少一次默认构造和赋值操作

     - 当使用构造函数体内部赋值的方式初始化成员变量时，对于类类型的成员变量，编译器会先调用默认构造函数来创建对象，然后再执行赋值操作。

       如果在构造函数体中进行赋值初始化，如

       ```cpp
       MyClass::MyClass() { obj = OtherClass(10); }
       ```

       1. **首先会调用`OtherClass`的默认构造函数创建`obj`**
       2. **然后再调用`OtherClass`的赋值运算符（`operator=`）将一个新创建的`OtherClass(10)`赋值给`obj`**。

       这样既要使用**默认构造函数创建**obj又要使用**拷贝初始化**创建一个新的临时变量赋给obj相当麻烦

       而使用成员初始化列表，如

       ```
       MyClass::MyClass() : obj(10) {}
       ```

       会直接调用`OtherClass`的合适的构造函数（这里是带参数的构造函数）即（**直接初始化**）来初始化`obj`，**避免了一次默认构造和赋值操作**，从而提高了效率。

   - 对于常量和引用成员必须使用初始化列表

     - **常量成员变量和引用成员变量以及没有默认构造函数的成员变量**在初始化后不能被重新赋值。在构造函数体中进行赋值操作对于它们是不合法的。所以必须使用成员初始化列表来对常量和引用成员变量进行初始化，这也使得在初始化这些特殊类型成员变量时，代码更加符合语法规则并且高效。





### 52. C++ 新加了string，与char* 有什么区别？是如何实现的、

> string是个类，将char*封装了起来，既方便又安全

1. **区别回顾**
   - 存储管理
     - `char*`是一个指针，指向字符数组，存储管理由程序员负责，**容易出现内存泄漏、缓冲区溢出等问题**。而`std::string`是一个**类**，**内部自动管理存储字符序列的内存，根据字符串长度动态分配和释放内存**，更安全。
   - 操作便捷性
     - 操作`char*`依赖 C 语言的字符串函数，如`strcpy`、`strcat`等，操作较繁琐。`std::string`有丰富的成员函数和运算符重载，如`+`用于拼接、`==`用于比较，操作更方便。
   - 生命周期管理
     - 动态分配的`char*`指向的字符串需要手动释放内存，`std::string`对象的内存自动释放。
2. **`std::string`的实现原理**
   - 数据成员
     - `std::string`通常包含一个指针成员，用于指向存储字符序列的内存空间，还可能包含一个表示字符串长度的成员（有些实现还会有表示容量的成员，用于优化内存分配）。例如，在一些简单的实现中，`string`类可能有类似以下的数据成员：

```cpp
class string {
private:
    char* data;
    size_t length;
    // 可能还有容量相关成员，如size_t capacity;
};
```

## 内存管理

### 1. 类的对象存储空间

1. 非静态成员变量存储
   - 基本数据类型成员变量
     - 对于类中的基本数据类型成员变量（如`int`、`double`、`char`等），它们在对象的存储空间内**按照声明的顺序**依次存储。例如，有一个类`MyClass`：

```cpp
class MyClass {
public:
    int num1;
    double num2;
    char ch;
};
```

- 当创建`MyClass`的一个对象时，`num1`、`num2`和`ch`会在对象的存储空间中依次排列。对象存储的起始位置是`num1`的存储位置，`num1`占用`sizeof(int)`字节，`num2`紧跟其后，占用`sizeof(double)`字节，`ch`再紧跟`num2`，占用`sizeof(char)`字节。
- 类对象成员变量
  - 如果类中有其他类的对象作为成员变量，那么存储情况会复杂一些。当包含的类对象没有虚函数等特殊情况时，存储方式和基本数据类型类似，按照声明顺序存储这些类对象。每个类对象在存储时会包含其自身的成员变量（按照其自身的存储规则）。例如：

```cpp
class OtherClass {
public:
    int value;
};
class MyClass {
public:
    int num;
    OtherClass obj;
};
```

- 在`MyClass`对象的存储空间中，首先是`num`，然后是`obj`，`obj`会包含`OtherClass`对象自己的成员变量`value`。
- 数组类型成员变量
  - 对于数组类型的成员变量，它会在对象存储中占用连续的空间，空间大小为数组元素类型大小乘以数组元素个数。例如，一个类中有一个`int`数组作为成员变量：

```cpp
class MyClass {
public:
    int arr[5];
};
```

- `arr`会在对象存储中占用`5 * sizeof(int)`字节的连续空间。

2. **静态成员变量存储**

- 静态成员变量不属于任何一个具体的对象，它是类的所有对象共享的变量。静态成员变量存储在全局数据区（与全局变量存储位置类似），而不是对象的存储空间内。无论创建多少个类的对象，静态成员变量只有一份副本。例如：

```cpp
class MyClass {
public:
    static int static_num;
    int num;
};
int MyClass::static_num = 0;
```

- 当创建`MyClass`的多个对象时，`static_num`**并不在这些对象的存储空间内，而是在全局数据区单独存储**，所有`MyClass`对象都可以访问这个静态成员变量。

3. **虚函数相关存储（如果有虚函数）**

> 虚函数表在全局/静态存储区，虚函数在代码区，虚函数指针在对象存储空间

- 虚函数表指针（vptr）存储
  - **如果类中有虚函数，编译器会在对象的存储空间中添加一个虚函数表指针（vptr）。这个指针通常位于对象存储空间的开头（在一些编译器实现中）。**例如，一个有虚函数的类`MyClass`：

```cpp
class MyClass {
public:
    virtual void func() {}
    int num;
};
```

- 当创建`MyClass`的对象时，**对象存储的开头部分会有一个虚函数表指针，用于指向虚函数表（vtable）**，之后才是`num`的存储空间。虚函数表中**存储了类的虚函数的入口地址，通过虚函数表指针和虚函数表，实现动态绑定（多态）**。
- 虚函数表（vtable）存储
  - **虚函数表是一个存储虚函数入口地址的表格，它存储在全局数据区或者只读数据区（具体取决于编译器和系统）。每个有虚函数的类都有自己的虚函数表，表中的每一项对应一个虚函数的入口地址。例如，对于上述`MyClass`类，其虚函数表中有一项存储了`func`函数的入口地址。**

### 2. C++的内存分区

代码区、**全局/静态存储区、常量区**、栈、堆

![img](http://oss.interviewguide.cn/img/202205220021689.png)

1. 栈区（Stack）
   - 定义与特点
     - 栈是一种先进后出（FILO）的数据结构，在 C++ 中，栈区主要用于存储局部变量和函数参数。当一个函数被调用时，函数的参数和局部变量会在栈上分配空间。这些变量的生命周期与函数的执行周期相关，当函数执行结束后，栈上分配给这些变量的空间会自动释放。
   - 存储内容示例
     - 例如，有如下函数：

```cpp
void func(int param) {
    int local_variable = 10;
    // 函数执行过程中，param和local_variable存储在栈区
}
```

- 当`func`函数被调用时，`param`和`local_variable`会在栈上分配空间，函数执行完毕后，这些空间会自动回收，无需手动释放内存。栈区的内存分配和释放是由编译器自动管理的，这使得栈区的使用相对简单和安全。不过，栈区的大小是有限的，通常由编译器或者操作系统预先设定，如果函数的递归调用层数过深或者局部变量占用空间过大，可能会导致栈溢出。

2. 堆区（Heap）

- 定义与特点
  - 堆区用于动态分配内存，程序员可以通过`malloc`、`calloc`、`realloc`（C 语言）或者`new`、`delete`（C++）等操作符和函数在堆上分配和释放内存。堆区的内存分配是比较灵活的，不像栈区那样受函数调用和返回的限制。不过，这也意味着堆区的内存管理需要程序员更加小心，因为如果忘记释放内存，会导致内存泄漏；而如果在释放内存后仍然使用指向该内存的指针，会导致悬空指针问题，从而可能引起程序崩溃或者错误的行为。
- 存储内容示例
  - 例如，在 C++ 中：

```cpp
int* ptr = new int;
*ptr = 20;
// 此时，int类型的内存是在堆区分配的，需要使用delete来释放
delete ptr;
```

- 首先通过`new`操作符在堆区分配了能存储一个`int`类型数据的内存空间，将`20`存储到这个空间后，最后通过`delete`操作符释放了这块内存。在 C 语言中，可以使用`malloc`和`free`函数实现类似的功能。

3. 全局 / 静态存储区（Global/Static）

- 定义与特点
  - 全局变量和静态变量（包括**静态局部变量**和静态全局变量）存储在这个区域。全局变量在整个程序的生命周期内都存在，它们在程序开始执行时就被分配内存，并且在程序结束时才会释放内存。静态局部变量虽然是在函数内部定义的，但是它们的生命周期也贯穿整个程序的执行过程，并且只会初始化一次。**这个区域的内存分配是在程序编译和链接阶段确定的**。
- 存储内容示例
  - 例如：

```cpp
// 全局变量
int global_variable = 30;
void func() {
    // 静态局部变量
    static int static_local_variable = 40;
    // 函数每次调用，static_local_variable的值会保留
}
```

- `global_variable`存储在全局 / 静态存储区，在程序启动时就分配了内存，并且可以被程序中的任何函数访问（在合适的作用域内）。`static_local_variable`虽然是在`func`函数内部定义的，但它也是存储在全局 / 静态存储区，并且在第一次调用`func`函数时初始化，之后每次调用`func`函数，它的值都会被保留。

4. 常量存储区（Constant）

- 定义与特点
  - 常量存储区用于存储常量数据，如字符串常量和**用`const`修饰的全局变量**（在某些实现中）。这个区域的数据是只读的，不能被修改。在程序的整个生命周期内，这些常量都存储在这个特定的区域。
- 存储内容示例
  - 例如：

```cpp
const char* str = "Hello, World!";
// "Hello, World!"这个字符串常量存储在常量存储区
```

- 这里的`str`是一个指针，它本身可以存储在栈区或者其他合适的区域，但它所指向的字符串常量`"Hello, World!"`存储在常量存储区，不能通过`str`来修改这个字符串的内容。如果试图修改常量存储区的数据，会导致程序出现未定义行为。

5. 代码区（Code）

- 定义与特点
  - 代码区存储程序的可执行代码，也就是函数的机器指令。**这个区域是只读的**，因为在程序运行过程中，代码通常是不应该被修改的。当程序被加载到内存中运行时，操作系统会将可执行文件中的代码段加载到代码区。
- 存储内容示例
  - 例如，当定义一个函数：

```cpp
int add(int a, int b) {
    return a + b;
}
```

- `add`函数的机器指令会存储在代码区，当`add`函数被调用时，CPU 会从代码区读取相应的指令来执行函数的操作。

### 3. 虚函数

虚函数表**（Virtual Table，简称 V-Table 或 vtbl）**是 C++ 中实现虚函数机制的关键数据结构。

1. 基本定义
   - 虚函数表是一块**连续的内存空间**，其中每个内存单元记录着一个函数指针的地址。**这些函数指针指向类中的虚函数。可以将虚函数表理解为一个函数地址的数组，数组中的每个元素对应一个虚函数的地址。**
2. 创建与关联
   - 编译器会为每个包含虚函数的类（或者继承自包含虚函数的类）创建一个虚函数表。该**类的所有对象都共享这一个虚函数表**。同时，**编译器会在类的对象中添加一个隐藏的指针，称为虚函数表指针（vptr），这个指针指向该类的虚函数表**。当创建类的对象时，vptr 会被自动设置为指向所属类的虚函数表，这使得对象能够通过 vptr 找到对应的虚函数表。
3. 作用与意义
   - **实现多态性**：多态是 C++ 的重要特性，即通过父类类型的指针或引用操作子类对象时，能够**根据对象的实际类型调用正确的函数版本**。当使用父类指针调用虚函数时，程序会根据指针所指向对象的 vptr 找到对应的虚函数表，再从虚函数表中找到要调用的虚函数的地址，从而实现正确的函数调用。这解决了继承和覆盖的问题，保证了程序能够真实反映实际的函数调用。
   - **方便管理虚函数**：对于有多个虚函数的类，虚函数表提供了一种统一的方式来管理和查找这些函数。虚函数按照其声明的顺序放置在虚函数表中，方便了编译器和程序在运行时快速定位和调用虚函数3。
4. 多继承下的变化
   - 在多继承的情况下，每个父类都有自己的虚函数表。子类的对象中会包含多个 vptr，分别指向不同父类的虚函数表。并且，子类的成员函数会按照声明顺序被放置在第一个父类的虚函数表的最后（这里的第一个父类是按照声明顺序来判断的）。如果子类覆盖了某个父类的虚函数，那么在该父类的虚函数表中，被覆盖的虚函数的地址会被替换为子类中重写的虚函数的地址。

- 注意事项

1. 在子类中重写虚函数的函数**没有显式使用virtual关键字，但仍然也是虚函数**
2. 行为的**灵活性**：重写虚函数允许子类根据自身的特点和需求来**定制**特定的行为。不同的子类可以对同一个虚函数有不同的实现，**使得相同的函数调用在不同的对象上产生不同的效果**，这正是多态性的核心概念
3. 如果子类没有重写父类的非析构虚函数，并且通过父类指针或引用调用该虚函数，那么会调用父类中定义的版本。

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
    // 但注意！override不能出现在类以外的地方
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

#### 纯虚函数

```cc
class Shape{
	public:
		// virtural  + " = 0" 表面其是一个纯虚函数，实现只能在子类实现
		virtual void hi() = 0;
}
```

#### override

h文件写了override，那么在源文件**一定要重写/实现**，否则直接报错

```sh
In file included from /work/test_monitor/src/monitor/cpu_softirq_monitor.cpp:2:0:
/work/test_monitor/include/monitor/cpu_softirq_monitor.h:14:47: error: expected class-name before '{' token
 class CpuSoftIrqMonitor : public MonitorInter {
                                               ^
/work/test_monitor/include/monitor/cpu_softirq_monitor.h:33:8: error: 'void monitor::CpuSoftIrqMonitor::Stop()' marked 'override', but does not override
   void Stop() override {}
        ^~~~
/work/test_monitor/src/monitor/cpu_softirq_monitor.cpp: In member function 'void monitor::CpuSoftIrqMonitor::UpdateOnce(monitor::proto::MonitorInfo*)':

```

### 4. C++多态

> 编译（静态）多态：函数重载+运算符重载 ； 运行多态：虚函数重写

1. 多态的定义

   - 在 C++ 中，多态（Polymorphism）是指一个名字（如函数名、运算符等）具有多种不同的语义。它允许**使用统一的接口来处理多种不同类型的对象**，并且在运行时根据对象的实际类型来决定调用哪种具体的实现。简单来说，多态使得程序能够以一种通用的方式处理不同类型的对象，而这些对象可以对相同的消息（函数调用）做出不同的响应。

2. 多态的实现方式

   - 静态多态（编译时多态）

     地址早绑定early binding，根据**指针或引用的声明类型**在编译阶段就确定**函数的调用地址**

     - **函数重载（Function Overloading）**：这是最常见的静态多态形式。在同一个作用域内，可以定义多个同名函数，只要它们的参数列表（参数个数、类型或顺序）不同。例如：

```cpp
int add(int a, int b) {
    return a + b;
}
double add(double a, double b) {
    return a + b;
}
```

当调用`add`函数时，编译器会根据传入的参数类型在编译时确定具体调用哪个版本的`add`函数。

- **运算符重载（Operator Overloading）**：**C++ 允许对运算符进行重载**，使得运算符可以用于自定义类型。例如，对于一个复数类`Complex`，可以重载`+`运算符来实现复数的加法：

```cpp
class Complex {
public:
    double real;
    double imag;
    Complex operator+(const Complex& other) {
        Complex result;
        result.real = real + other.real;
        result.imag = imag + other.imag;
        return result;
    }
};
```

当使用`Complex`类的对象进行加法运算（如`Complex c1, c2; Complex c3 = c1 + c2;`）时，编译器会根据重载的`+`运算符定义来执行复数加法操作。

- 动态多态（运行时多态）
  - 动态多态是通过**虚函数（Virtual Function）和继承（Inheritance）**来实现的。基类定义一个虚函数，然后派生类可以重写（Override）这个虚函数。函数**地址晚绑定(late binding)，只有在运行时才确定，编译时无需检查对象类型只需检查对象是否支持特征和方法**
  - 当通过基类指针或引用调用这个虚函数时，实际调用的是派生类中重写后的函数。例如：

```cpp
class Shape {
public:
    virtual double area() const {
        return 0.0;
    }
};
class Circle : public Shape {
public:
    Circle(double r) : radius(r) {}
    virtual double area() const override {
        return 3.14159 * radius * radius;
    }
private:
    double radius;
};
```

- 当有`Shape* shape = new Circle(5.0);`这样的代码，并且调用`shape->area()`时，在运行时会根据`shape`实际指向的对象（这里是`Circle`对象）来调用`Circle`类中的`area`函数，而不是`Shape`类中的`area`函数。

- 多态的优势

  - 代码的可维护性和可扩展性
    - 多态使得代码结构更加清晰和灵活。通过使用统一的接口，如基类的虚函数，在添加新的派生类时，只需要在派生类中重写相应的虚函数，而不需要修改调用这些函数的代码。例如，在图形处理程序中，**如果要添加一个新的图形类型（如矩形），只需要从`Shape`基类派生一个新的`Rectangle`类，并重写`area`函数**，而程序中处理图形的通用部分（如计算一组图形的总面积）不需要修改。

  - 提高代码的复用性
    - 无论是静态多态还是动态多态，都可以提高代码的复用性。在函数重载中，同一个函数名可以用于执行类似的操作，只是参数不同，这避免了为每个类似操作定义不同的函数名。在动态多态中，基类的代码可以被派生类复用，同时派生类可以根据自身的特点定制虚函数的实现。

### 4. 什么是内存池，如何实现

### 5. C++中的数据成员和成员函数内存分布情况

1. 没有虚函数的类
   - 数据成员
     - **非静态数据成员在类的对象内存空间中按照声明的顺序依次存储**。例如，对于一个简单的类`class MyClass { int a; double b; };`，当创建`MyClass`的对象时，对象的内存空间首先存储`a`，然后是`b`。`a`占用`sizeof(int)`字节，`b`紧跟`a`之后，占用`sizeof(double)`字节。
   - 成员函数
     - 普通的成员函数（非虚函数）并不存储在对象的内存空间中。成**员函数的代码存放在代码区（text segment）**，所有该类的对象共享这些成员函数的代码。当**通过对象调用成员函数时，编译器会将对象的地址（`this`指针）作为隐藏参数传递给成员函数**，成员函数通过`this`指针来访问对象的数据成员。例如，`MyClass`类的成员函数`void func() { a++; }`，在调用`func`函数时，实际上传递了对象的`this`指针，**在函数内部通过`this->a++`来访问和修改对象的数据成员`a`**。
2. 包含虚函数的类
   - 数据成员和虚函数表指针（vptr）
     - 除了非静态数据成员按照上述顺序存储在对象内存空间中，编译器还会在对象中添加一个虚函数表指针（vptr）。这个指针通常位于对象内存空间的开头（不同编译器可能有不同的实现方式）。例如，对于类`class MyClassWithVirtual { virtual void func(); int a; };`，当创建对象时，对象内存空间首先是 vptr，然后是`a`。vptr 指向一个虚函数表（vtable），虚函数表存储在只读数据区（一般情况下），其中包含了类的虚函数的入口地址。
   - 虚函数和虚函数表
     - **虚函数的代码同样存放在代码区**，和普通成员函数一样。但是虚函数的调用方式与普通成员函数不同。当通过对象的指针或引用调用虚函数时，**程序会根据对象的 vptr 找到对应的虚函数表，再从虚函数表中获取要调用的虚函数的地址，从而实现动态绑定**。例如，有`MyClassWithVirtual* ptr = new MyClassWithVirtual;`，当调用`ptr->func();`时，会先通过`ptr`指向对象的 vptr 找到虚函数表，再从虚函数表中找到`func`函数的地址并调用。
3. 静态数据成员和静态成员函数
   - 静态数据成员
     - **静态数据成员不属于任何一个具体的对象，它是类的所有对象共享的变量**。静态数据成员存储在全局 / 静态存储区（与全局变量存储位置类似），无论创建多少个类的对象，静态数据成员只有一份副本。例如，对于类`class MyClass { static int static_a; };`，`static_a`存储在全局 / 静态存储区，并且可以通过`MyClass::static_a`的方式访问。
   - 静态成员函数
     - **静态成员函数也不存储在对象的内存空间中。它和普通成员函数一样，代码存放在代码区**。**静态成员函数没有`this`指针**，**因为它不与具体的对象相关联，它主要用于操作类的静态数据成员或者执行与类相关的通用操作，而不能直接访问非静态数据成员（除非通过对象来访问）**。例如，`class MyClass { static void static_func() { /* 操作静态数据成员 */ } };`，可以通过`MyClass::static_func();`来调用静态成员函数。

### 6. this指针

1. **this 指针的定义和本质**

- **含义**：在 C++ 中，`this`指针是一个隐含的指针，它是一个常量指针，指向当前正在被操作的对象。当调用一个类的成员函数时，编译器会自动将调用该函数的对象的地址赋给`this`指针。例如，对于类`MyClass`的成员函数`void func();`，当通过对象`obj`调用`func`函数（即`obj.func();`）时，在`func`函数内部，`this`指针就指向`obj`。
- **存储位置和类型**：`this`指针本身是一个**（底层const）常量指针，它存储在栈帧中（对于非静态成员函数）**。其类型是`const *`，这意味着`this`指针不能被重新赋值指向其他对象，但可以通过`this`指针来修改它所指向对象的成员变量。例如，在`MyClass`的成员函数`void func() { (*this).member_variable = 10; }`中，通过`this`指针来访问和修改对象的成员变量。

2. **this 指针在成员函数中的作用**

- **访问成员变量**：成员函数通过`this`指针来访问对象的非静态成员变量。这是因为在类的多个对象共享成员函数的代码时，成员函数需要知道是对哪个对象的成员变量进行操作。例如，有一个类`MyClass`：

```cpp
class MyClass {
public:
    int value;
    void setValue(int v) {
        this->value = v;
    }
};
class MyClass {
public:
    int value;
    void setValue(int v) {
        // 编译器会自动将value = v;解释为this->value = v;，它能够正确地识别是对当前对象的成员变量进行赋值操作。
        value = v;
    }
};
class MyClass {
public:
    int value;
    void setValue(int value) {
    // 解决命名冲突：当成员变量和函数的参数或者局部变量同名时，需要使用this指针来明确是访问成员变量。例如
        this->value = value;
    }
};
```

- 在`setValue`函数中，`this->value`用于访问调用该函数的对象的`value`成员变量。如果没有`this`指针，成员函数就无法区分不同对象的成员变量。
- **实现对象间的操作**：`this`指针还可以**用于在成员函数中返回对象本身，从而实现链式调用**。例如，对于一个表示向量的类`Vector`：

```cpp
class Vector {
public:
    double x, y;
    Vector& add(const Vector& other) {
        this->x += other.x;
        this->y += other.y;
        return *this;
    }
};
```

- 这样就可以进行链式调用，如`Vector v1, v2; v1.add(v2).add(v2);`，`this`指针使得`add`函数能够返回对象本身，方便了这种操作方式。

3. **this 指针在继承中的情况**

- **单继承情况**：在单继承中，当在派生类的成员函数中访问继承自基类的成员变量或者重写基类的成员函数时，`this`指针的行为和在基类中的行为类似。**`this`指针仍然指向派生类的对象，并且能够正确地访问基类和派生类的成员。**例如：

```cpp
class Base {
public:
    int base_value;
    void base_func() {}
};
class Derived : public Base {
public:
    int derived_value;
    void derived_func() {
        this->base_value = 10;
        this->derived_value = 20;
        this->base_func();
    }
};
```

- **多继承情况**：在多继承的情况下，`this`指针的使用会稍微复杂一些。派生类对象可能包含多个基类子对象，`this`指针需要根据具体的继承结构和成员访问情况来正确地指向相应的部分。例如，在一个同时继承自两个基类`Base1`和`Base2`的派生类`Derived`中，当访问`Base1`的成员时，`this`指针会被调整到指向`Derived`对象中`Base1`子对象的部分；当访问`Base2`的成员时，`this`指针会被调整到指向`Derived`对象中`Base2`子对象的部分。不过，编译器会自动处理这些细节，确保`this`指针能够正确地访问各个基类和派生类的成员。

4. **静态成员函数与 this 指针**

- **静态成员函数是属于类而不是属于对象的，它不依赖于具体的对象而存在。因此，静态成员函数没有`this`指针**。在静态成员函数中不能直接访问非静态成员变量和非静态成员函数，因为没有`this`指针来指定要操作的对象。如果要在静态成员函数中访问非静态成员，需要通过对象来调用。例如：

```cpp
class MyClass {
public:
    static void static_func() {
        // 不能直接访问非静态成员变量和非静态成员函数
    }
    int non_static_value;
    void non_static_func() {}
};
```

- 可以通过以下方式在静态成员函数中访问非静态成员：

```cpp
class MyClass {
public:
    static void static_func(MyClass& obj) {
        obj.non_static_value = 10;
        obj.non_static_func();
    }
    int non_static_value;
    void non_static_func() {}
};
```

#### 易混问题

1. **与静态成员函数的混淆**
   - **错误使用尝试**：静态成员函数是属于类而不是对象的，它没有`this`指针。一个常见的错误是在静态成员函数中试图直接访问非静态成员变量或调用非静态成员函数，就好像有`this`指针一样。例如：

```cpp
class MyClass {
public:
    int nonStaticMember;
    static void staticFunction() {
        nonStaticMember = 10; // 错误，没有this指针来访问非静态成员
    }
};
```

- **正确使用方式**：如果要在静态成员函数中访问非静态成员，必须通过类的对象来进行访问。例如：

```cpp
class MyClass {
public:
    int nonStaticMember;
    static void staticFunction(MyClass& obj) {
        obj.nonStaticMember = 10; 
    }
};
```

2. **在函数参数与成员变量同名时的误解**

- **命名冲突导致的错误理解**：当成员函数的参数与成员变量同名时，容易忽略`this`指针的隐式使用。如果没有正确使用`this`指针，代码可能不会按照预期执行。例如：

```cpp
class MyClass {
public:
    int value;
    void setValue(int value) {
        value = value; // 这里只是将参数赋值给自身，而非成员变量
    }
};
```

- **正确的处理方式**：应该使用`this`指针来明确访问成员变量，如`this->value = value;`，这样才能将参数值正确地赋给成员变量。

3. **在继承和多态场景中的混淆**

- **多继承中的指针偏移误解**：在多继承的情况下，`this`指针在派生类对象中的位置和调整可能会引起混淆。派生类对象包含多个基类子对象，`this`指针需要正确地指向相应的部分来访问各个基类的成员。例如，对于一个继承自`Base1`和`Base2`的派生类`Derived`，程序员可能错误地认为`this`指针在访问基类成员时的行为和单继承一样简单。实际上，编译器会根据继承顺序和内存布局对`this`指针进行调整，以访问正确的基类部分。
- **虚函数和动态绑定中的`this`指针**：在动态多态（通过虚函数实现）的场景中，通过基类指针或引用调用虚函数时，`this`指针指向的是实际对象（派生类对象）。但是，初学者可能会误解`this`指针总是指向基类类型的对象。例如：

```cpp
class Shape {
public:
    virtual void draw() {
        // 基类的draw函数
    }
};
class Circle : public Shape {
public:
    void draw() override {
        // 派生类Circle的draw函数
    }
};
Shape* shape = new Circle;
shape->draw(); 
// 这里this指针在draw函数内部指向Circle对象，而不是Shape对象，容易误解
```

4. **在返回对象引用时的错误使用**

- **返回局部对象引用的问题**：当在成员函数中返回`*this`（对象本身的引用）用于链式调用时，如果**不小心返回了一个局部对象的引用，会导致程序出现错误**。例如：

```cpp
class MyClass {
public:
    MyClass& createObject() {
        MyClass temp;
        return temp; // 错误，返回了局部对象的引用，对象在函数结束后销毁
    }
};
```

- **正确的返回方式**：应该返回调用函数的对象本身的引用，确保对象在返回引用后仍然有效。例如：

```cpp
class MyClass {
public:
    MyClass& createObject() {
        // 执行一些操作来修改当前对象
        return *this;
    }
};
```

#### 难点

1. **this 指针的创建时间**
   - 在 C++ 中，`this`指针是在调用类的非静态成员函数时创建的。当通过对象调用成员函数时，编译器会自动生成代码来传递对象的地址作为`this`指针。例如，对于类`MyClass`和对象`obj`，当执行`obj.someFunction();`时，**编译器会在调用`someFunction`函数前，将`obj`的地址赋值给`this`指针，**然后在`someFunction`函数内部就可以通过`this`指针来访问`obj`的成员变量和调用其他成员函数。
2. **this 指针的存放位置**
   - `this`指针存放在栈帧中。当调用非静态成员函数时，函数的栈帧会被创建，`this`指针作为一个隐含的参数被压入栈帧。在函数执行过程中，通过栈帧中的`this`指针来访问对象的成员。例如，在一个简单的函数调用`obj.func();`中，`obj`的地址（即`this`指针）会和函数的其他参数（如果有）一起被压入栈中，用于函数内部对对象的操作。
3. **this 指针的传递方式**
   - 在 C++ 中，`this`指针是作为一个**隐含的参数传递给成员函数的**。在底层实现上，它类似于函数的首参数。例如，对于类`MyClass`的成员函数`void func(int param);`，在编译器生成的代码中，**实际上可能类似于`void func(MyClass* this, int param);`。**不过，在编写代码时，不需要显式地传递`this`指针，编译器会自动处理这个过程。这种传递方式使得成员函数能够知道是对哪个对象进行操作。
4. **this 指针访问类变量的方式**
   - 可以通过`this`指针使用`->`运算符或者`(*this).`的形式来访问类的成员变量。例如，对于类`MyClass`，如果有成员变量`value`，在成员函数中可以通过`this->value`或者`(*this).value`来访问。通常情况下**，`this->`的形式更常用**，因为它更简洁并且符合 C++ 的编程习惯。这种访问方式是基于`this`指针所指向的对象来定位成员变量在内存中的位置。
5. **关于直接使用对象的 this 指针**
   - this指针只有在成员函数才有定义，因此获得一个对象后也无法使用this指针
6. **关于类函数表**
   - C++ 编译器通常会为每个类创建一个虚函数表（vtable），但不是所有的函数。只有包含虚函数的类会有虚函数表。虚函数表是一个**函数指针数组**，其中每个元素对应一个虚函数的入口地址。当通过基类指针或引用调用虚函数时，会根据对象的虚函数表指针（vptr）找到对应的虚函数表，再从表中获取要调用的虚函数的地址，从而实现动态绑定。**对于非虚函数，编译器在编译时就可以确定函数的调用地址，不需要这样的函数表来辅助调用**。因为**非虚函数的调用是静态绑定的**，编译器根据对象的类型就知道应该调用哪个函数。
7. **this指针调用成员变量时，堆栈会发生什么**
   - **栈帧的基本情况**
     - 当通过`this`指针调用成员变量时，首先要了解栈帧的概念。**栈帧是在函数调用过程中，为函数的执行分配的一块栈空间**，用于存储函数的参数、局部变量、返回地址等信息。当调用一个类的非静态成员函数时，会为这个函数创建一个栈帧。
   - **this 指针的压栈过程**
     - 在调用非静态成员函数时，`this`指针会被作为一个隐含的参数压入栈帧。**它通常是第一个被压入栈帧的参数（**在底层实现上类似于函数的第一个参数）。例如，对于一个类`MyClass`的成员函数`void func(int param);`，在编译器生成的代码中，实际的函数调用可能类似于`func(MyClass* this, int param);`，`this`指针会和其他显式参数（这里是`param`）一起存储在栈帧中。
   - **访问成员变量时的操作**
     - 当通过`this`指针访问成员变量时，实际上是根据`this`指针在栈帧中的值（也就是对象的地址），结合成员变量在对象中的偏移量来获取成员变量的内存地址。例如，**如果`this`指针的值为`0x1000`（假设对象存储在内存地址`0x1000`处）**，成员变量`value`在对象中的偏移量为`4`字节（假设`int`类型的成员变量），**那么通过`this->value`访问成员变量时，计算得到的成员变量地址为`0x1000 + 4`。**这个过程不会引起栈帧的大小变化，只是根据`this`指针和偏移量进行内存地址的计算，以访问正确的成员变量。
     - 从性能角度看，这种访问方式相对高效，因为它只是简单的指针运算。只要`this`指针和成员变量的偏移量是已知的（编译器在编译时可以确定这些信息），就可以快速地定位成员变量。而且，由于栈帧中已经存储了`this`指针，不需要额外的复杂操作来获取对象的地址以访问成员变量。
8. **在成员函数中调用delete this会怎么样？对象还能使用吗？在类的析构函数调用delete this会怎么样**

### 7. 内存泄露的后果？如何监测？解决方法

> 泄露后果

- 性能下降
  - 随着程序的运行，内存泄漏会导致程序占用的内存不断增加。这会使系统的可用内存逐渐减少，从而影响系统的性能。例如，在一个长时间运行的服务器程序中，如果存在内存泄漏，可能会导致服务器响应速度变慢。**因为当系统内存不足时，操作系统会频繁地进行磁盘交换（将内存数据交换到磁盘的虚拟内存中），而磁盘 I/O 操作的速度远远低于内存访问速度，这就会大大降低程序的运行效率。**
- 系统崩溃
  - 如果内存泄漏持续发生，程序占用的内存可能会耗尽系统的所有可用内存。一旦内存耗尽，操作系统可能会因为无法为新的进程或程序分配足够的内存而导致**系统崩溃**。这种情况在一些对稳定性要求极高的系统中（如医疗设备系统、航空航天控制系统等）是绝对不允许的，因为系统崩溃可能会带来严重的后果。

![image-20241004214537982](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20241004214537982.png)

- 有new就有delete、有malloc就有free

  ![image-20241006100505648](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20241006100505648.png)

- 数组一定要用delete[]

- **基类的析构函数声明为虚函数**

  避免资源泄漏和错误行为

  - 考虑析构函数的情况。如果基类的析构函数不是虚函数，当**通过基类指针删除一个派生类对象时，只会调用基类的析构函数**。
  - 例如：

  ```cpp
     class Base {
     public:
         ~Base() {
             // 基类资源清理代码
         }
     };
     class Derived : public Base {
     public:
         ~Derived() {
             // 派生类资源清理代码
         }
     };
  ```

  ```cpp
     Base* base = new Derived();
     delete base;
  ```

  - 因为`Base`类的析构函数不是虚函数，所以这里只会调用`Base`类的析构函数，即**不会触发动态绑定**而不会调用`Derived`类的析构函数。这可能**导致派生类中分配的资源（如动态内存等）无法正确释放，造成资源泄漏**。

  - 但如果将基类的析构函数声明为虚函数：

    ```cpp
      class Base {
       public:
           virtual ~Base() {
               // 基类资源清理代码
           }
       };
    ```

    - 当通过基类指针删除派生类对象时，会**先调用派生类的析构函数，然后再调用基类的析构函数**，保证了资源的正确释放。

### 8. 成员函数调用delete this会出现什么问题？对象还可以使用吗？

1. **在成员函数中调用`delete this`**
   - **对象生命周期的改变**：当在成员函数中调用`delete this`时，对象本身的内存会被释放。这**意味着对象的析构函数会被调用（如果析构函数存在），并且对象所占用的内存会被归还给堆管理器**，之后对象就不再存在了。
   - **后续使用的风险**：在调用`delete this`之后，**对象已经被销毁**，任何对该对象成员变量或成员函数的访问都将导致未定义行为。因为对象所占用的内存可能已经被重新分配给其他数据使用，或者指针已经变成悬空指针。例如：

```cpp
class MyClass {
public:
    void selfDestruct() {
        delete this;
    }
    void doSomething() {
        // 假设在selfDestruct之后调用这个函数，会出现问题
        // 因为对象已经不存在了
    }
};
```

- **正确的使用场景（非常少见）**：这种操作在一些特殊的设计模式（如引用计数对象的自我销毁）中可能会用到。例如，在一个实现了引用计数的智能指针类中，当引用计数降为 0 时，可以在成员函数中调用`delete this`来释放对象自身的内存。但这种操作需要非常谨慎，并且要确保对象的使用者都清楚对象可能会被自我销毁的情况。

2. **在析构函数中调用`delete this`**

- **栈溢出风险（递归调用）**：在析构函数中调用`delete this`通常是一个非常危险的行为。因为析构函数本身就是在对象销毁时被调用的过程，当在析构函数中再次调用`delete this`时，delete本来就会调用析构函数，会导致**析构函数的递归调用。**这可能会**导致栈溢出，因为每次函数调用都会在栈上占用一定的空间，过多的递归调用会耗尽栈空间。**

### 9. 空类的大小是多少

1. 空类的大小规则

   - 在 C++ 中，空类（没有任何数据成员、虚函数等的类）的大小通常是 1 字节。这是因为 **C++ 标准规定，在内存中，每个对象都必须有一个独一无二的地址，即使这个类不包含任何数据成员**。

2. 原因解释

   - **对象的可区分性**：**如果空类的大小为 0 字节，那么当创建多个空类对象时，它们的地址将是相同的，这与 C++ 中对象必须具有唯一地址的原则相违背**。例如，假设有两个空类对象`obj1`和`obj2`，如果类的大小为 0，那么`&obj1`和`&obj2`将会得到相同的结果，这会导致逻辑上的混乱。
   - **内存对齐和指针操作**：**从内存对齐的角度来看，为了方便指针操作和内存管理，即使是空类也需要占用一定的空间。**当使用指针来指向类对象时，一个大小为 0 的对象会使得指针的算术运算和比较操作变得复杂和不符合常规逻辑。例如，在一些需要遍历对象数组的场景中，如果对象大小为 0，指针的自增操作将无法按照正常的方式来定位下一个对象。

   ![image-20241004215418455](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20241004215418455.png)

![image-20241004215410904](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20241004215410904.png)

### 10. 类对象的大小受哪些因素影响

> 非静态数据成员、虚函数指针、**内存对齐**、**继承自基类的数据成员**

1. 非静态数据成员
   - 基本数据类型成员
     - 类中每个基本数据类型（如`int`、`double`、`char`等）的非静态成员变量会占用一定的空间，其大小取决于数据类型本身。例如，`int`类型通常占用 4 字节（在 32 位系统中），`double`类型通常占用 8 字节。如果一个类`MyClass`有一个`int`成员变量`a`和一个`double`成员变量`b`，那么这些数据成员就会占用`4 + 8 = 12`字节（不考虑内存对齐的情况下）。
   - 自定义类型成员变量
     - 当类中有其他自定义类型（如另一个类对象）作为成员变量时，该成员变量的大小等于其所属类对象的大小。例如，有一个类`OtherClass`，其大小为`N`字节，若`MyClass`包含一个`OtherClass`的成员变量`obj`，那么`obj`就会占用`N`字节的空间（同样暂不考虑内存对齐）。
   - 数组类型成员变量
     - 对于数组类型的成员变量，其占用的空间为数组元素类型大小乘以数组元素个数。例如，一个类中有一个`int`数组`arr[5]`作为成员变量，`int`类型假设为 4 字节，那么这个数组就会占用`4 * 5 = 20`字节的空间。
2. **内存对齐要求**
   - 基本原理
     - 现代计算机的内存系统是按照字节对齐的方式来访问内存的，这是为了提高内存访问效率。编译器会根据处理器的要求和数据类型的大小，对类中的成员变量进行内存对齐。例如，在大多数系统中**，`int`类型（通常 4 字节）的存储地址要求是 4 的倍数，`double`类型（通常 8 字节）的存储地址要求是 8 的倍数。**
   - 对齐对大小的影响
     - 当一个类中有不同类型的成员变量时，编译器会在成员变量之间添加一些填充字节（padding）来满足内存对齐要求。**例如，一个类`MyClass`有一个`char`成员变量`c`（占用 1 字节）和一个`int`成员变量`a`（占用 4 字节）。由于`int`类型要求 4 字节对齐，所以编译器可能会在`c`和`a`之间添加 3 个填充字节，使得`a`的存储地址是 4 的倍数。**这样，`MyClass`对象的大小就不是简单的`1 + 4 = 5`字节，而是 8 字节。
   
   ![image-20241006110227859](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20241006110227859.png)
3. 虚函数的影响（如果有）
   - **虚函数表指针（vptr）**
     - 如果类中有虚函数，编译器会在类对象中添加一个虚函数表指针（vptr）。这个指针通常占用一个机器字长的空间，例如在 32 位系统中占用 4 字节，在 64 位系统中占用 8 字节。这个指针用于在运行时动态地查找虚函数的地址。所以，含有虚函数的类对象大小会比没有虚函数的类似类对象大一个指针的大小。
   - 多继承中的虚函数情况
     - 在多继承且涉及虚函数的情况下，每个基类的虚函数表指针（如果有）都会添加到派生类对象中。这可能会导致派生类对象的大小增加多个指针大小的空间。例如，**一个派生类继承自两个含有虚函数的基类，那么派生类对象可能会包含两个虚函数表指针，从而使对象大小比单继承情况或者没有虚函数继承情况更大**。
4. **静态成员变量（不影响对象大小）**
   - 静态成员变量不属于任何一个具体的对象，它是类的所有对象共享的变量。静态成员变量存储在全局数据区，而不是对象的存储空间内。所以，无论类中有多少个静态成员变量，它们都不会影响类对象的大小。例如，一个类`MyClass`有静态成员变量`static_a`和`static_b`，这些静态变量的存在不会改变`MyClass`对象的大小。
5. 编译器的优化和特殊设置
   - 空基类优化（EBO）
     - 一些编译器支持空基类优化（Empty Base Class Optimization）。如果一个类继承自一个空类（没有数据成员、虚函数等），在某些情况下，编译器可以优化派生类的布局，使得空基类不占用额外的空间。例如，在符合 EBO 条件的编译器中，派生类可以直接使用空基类的空间，而不是为其单独分配空间。
   - 编译选项和字节对齐设置
     - 不同的编译选项可能会影响类对象的大小。例如，通过设置字节对齐的编译选项，可以改变编译器对内存对齐的要求。一些编译器允许指定字节对齐的字节数，如 1 字节对齐、2 字节对齐等。改变对齐方式可能会改变类对象的大小，因为不同的对齐方式会导致不同的填充字节数量。