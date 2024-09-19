

### 1. main函数执行之前和之后的 代码可能是什么

- main函数运行之前

  - 初始化全局`global`和静态变量`static`，`.data`(data segment，占用实际磁盘空间)段

    ```
         int global_var = 42;
         static int static_var = 100;
    ```

  - 自动初始化**未赋初值**的`global`和`static`对象，bool为FALSE，指针为NULL，数值型为0等；即`.bss`段(block started by symbol segment，不占用实际磁盘空间，因为只存储未初始化的数据占位符，当程序加载到内存时，`.bss`段会被初始化为0或空指针不需从磁盘复制数据，因此省磁盘空间)

  - 初始化**全局对象**

  - 设置栈指针

  - 将main函数的参数`argc`，`argv`传递给main函数，然后运行真正的main

  - `__attribute__((construtor))`是一个编译器特定的属性，用于指定一个函数在程序启动时（在`main`函数执行之前）自动被调用，就像一个构造函数一样。

    ```cpp
    #include <stdio.h>
    
    void init_function() __attribute__((constructor));
    
    void init_function() {
        printf("This function is called before main.\n");
    }
    
    int main() {
        printf("This is the main function.\n");
        return 0;
    }
    ```

- 运行之后

  - 全局对象析构函数在main函数之后执行

  - `__attribute__((destructor))`：是一个编译器特定的属性，用于指定一个函数在程序结束时（在`main`函数执行之后）自动被调用，就像一个析构函数一样。

  - ```c
    #include <stdio.h>
    
    void cleanup_function() __attribute__((destructor));
    
    void cleanup_function() {
        printf("This function is called after main.\n");
    }
    
    int main() {
        printf("This is the main function.\n");
        return 0;
    }
    ```

  - `atexit`是一个函数，用于注册一个在程序正常终止时被调用的函数。

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    
    void cleanup() {
        printf("Cleanup function called.\n");
    }
    
    int main() {
        atexit(cleanup);
        printf("Main function.\n");
        return 0;
    }
    ```

    ### 

### 2. 结构体内存对齐问题

- `alignof`：**返回**内存对齐的大小
- `alignas`：**指定**内存对齐的大小

```c
struct alignas(32) MyStruct {
  int a;
  double b;
};
std::cout << "alignof(MyStruct): " << alignof(MyStruct) << std::endl; // 32
std::cout << "sizeof(MyStruct): " << sizeof(MyStruct) << std::endl; // 32：alignas指定了对齐方式为 32 字节，为了满足对齐要求，结构体的大小需要是 32 字节的整数倍。
//int a占 4 字节，double b占 8 字节，总共是 12 字节，但是为了满足对齐要求，结构体的大小需要向上取整到 32 字节。

struct  MyStruct {
  int a;
  double b;
};
std::cout << "alignof(MyStruct): " << alignof(MyStruct) << std::endl; // 8
std::cout << "sizeof(MyStruct): " << sizeof(MyStruct) << std::endl; // 4字节的int要与最大的double8个字节对齐，16
```



### 3. 指针和引用区别

- 指针是**变量**，存储另一个变量的地址；引用是另一个变量的别名
- 指针在内存中为固定大小（**32位系统是4字节，64位系统8字节**）；**引用本身不占用额外空间**，是另一个变量的别名
- 指针可以**重新赋值**；引用一旦初始化就不能再绑定其他对象，**整个生命周期始终指向同一个对象**
- 指针可以是空指针；引用一定要绑定到一个有效变量，**不存在空引用**



### 4. 传递函数参数时，指针和引用区别

- **类对象**作为参数传递时使用引用
- **栈空间大小敏感（递归）**使用引用，因为**引用不用开辟新空间**
- 需返回**局部变量内存**使用指针，用完记得释放指针的内存不然会泄露



### 5. 栈和堆的区别

- 栈由系统**自动分配，操作系统在底层有分配专门的寄存器存放栈的地址，入栈出栈有专门的指令执行，速度快，不会有碎片**；堆由程序员申请和释放，**由C/C++函数库提供，获取堆内容需要两次访问第一次指针第二次指针保存的地址访问内存**，速度慢，会有碎片
- 栈空间默认4MB，从**高地址向低地址增长**；堆一般1-4GB，从低地址向高地址增长
- 栈存放函数的局部变量、函数参数、返回地址等；存放动态分配的对象（c++ new和delete；c malloc和free）、数据结构

###  6 区分以下指针类型

```cpp
int *p [10]; // 数组，元素是指针
int (*p) [10]; // 指针，指向数组
int *p (int); // 函数，返回int型指针
int (*p) (int); // 指针，指向int型参数int型返回类型的函数
```



### 7. news/delete 和 malloc/free区别

- new是**操作符，支持重载**；malloc是c/c++标准库函数

- new自动计算分配的空间大小，**返回的是具体类型指针**；malloc要手动计算，且**返回的是void型指针必须进行类型转换**

  ```cpp
  // 指向的int对象值为5，不是内存p大小为5
  int *p = new int(5);
  delete a;
  // 数组大小为5
  int *a = new int[5];
  delete [] a;
  // 返回的是void型指针必须进行类型转换
  int *a = (int*)malloc(sizeof(int));
  free(a);
  ```

- new/delete是操作符，与面向对象紧密结合，**适合非基本数据类型的对象，自动调用对象的构造函数和析构函数**进行对象进行创建和销毁；malloc/free库函数只分配和销毁内存，**不进行对象构造和析构**

- `free`回收的内存不会立即还给系统，而是被**`ptmalloc`双链表**保存起来用户下一次申请内存就从这些内存寻找合适的返回，避免频繁调用；`patmalloc`也会对小块内存进行合并，避免过多的内存碎片





### 8. new/delete是如何实现的

- **new先调用operator new进行分配内存**，后调用对象构造函数进行对象构造
- **delete先调用对象析构函数**，后调用operator delete释放内存



### 9. 宏定义与函数、typedef区别

```cpp
#define PI 3.14159
#define SQUARE(x) ((x)*(x))

int square(int x) { return x * x; } // 需要使用square(5)
typedef int myIntType; // 定义了一个新的类型名myIntType，它实际上是int类型的别名
```



- 宏定义没有类型检查，在**预处理阶段**即编译前完成，只是简单的文本替换；函数有严格的**类型检查**（参数类型、返回类型），**编译时**进行检查；`typedef`也是**编译**的一部分，会检查数据类型
- 宏效率高，相当于直接插入代码不需要调用；函数需要**一定开销**
- 宏在**整个文件可见**；函数有明确定义域；typedef也有明确定义域
- 宏**不是语句，不加分号**；调用函数、声明函数都要加分号；typedef也是语句，要加分号



### 10. 变量声明和定义区别

- 变量声明向编译器告知变量的存在，**但不分配内存空间或进行初始化**。它只是告诉编译器变量的**名称、类型和可能的作用域**。

  ```cpp
  extern int variable; 
  // 这里的extern关键字表示这是一个外部变量声明，告诉编译器这个变量在其他地方定义
  ```

- 变量定义不仅向编译器告知变量的存在，**还为变量分配内存空间**，并可以进行初始化。**一个变量在程序中可以有多个声明；但是只能有一个定义；**

### 11 strlen、sizeof、size()区别

- `sizeof`是一个**运算符**，用于在**编译时**确定给定类型或变量所占用的**字节数**；strlen是字符处理的库函数；对于**标准容器类**（如`std::vector`、`std::string`等），`size`是一个成员函数，用于在**运行时**确定**容器中元素的数量**。
- `sizeof`的结果在**编译时**就确定了，不依赖于程序的运行状态，返回值是一个`size_t`类型的无符号整数，表示**字节数**。对数组名**返回的是整个数组的大小而不是数组首元素的指针大小**；strlen只接受以'\0'结尾的字符数组
- sizeof返回值是无符号整数类型，表示字节数；strlen，返回值是`size_t`类型，通常是无符号整数类型，表示字符串的长度（字符个数）。
- `sizeof`：
  - 常用于确定数据类型的大小，以便进行内存分配、缓冲区大小计算等操作。
  - 在处理底层数据结构或进行与内存布局相关的操作时很有用。
- `size`：
  - 主要用于处理容器类对象，了解容器中当前存储的元素数量。
  - 在遍历容器、判断容器是否为空等操作中经常使用。

```cpp
int myArray[10];
std::vector<int> myVector;
// 编译时已确认
std::cout << "Size of myArray: " << sizeof(myArray) << std::endl; // 40
std::cout << "Size of int: " << sizeof(int) << std::endl; // 4

myVector.push_back(1);
myVector.push_back(2);
myVector.push_back(3);

std::cout << "Size of myVector: " << myVector.size() << std::endl; // 3，元素个数
```

`example2`

```c
void UdpServer::SendError() {
  std::string error;
  sendto(server_fd_, error.c_str(), error.size(), 0,
         (const struct sockaddr *)&client_addr_, sizeof(client_addr_));
}
```

1. 如果将 `error.size()` 替换为 `sizeof(error)`，这通常是不正确的做法，会产生错误的结果。

   - `sizeof(error)` 返回的是 `std::string` 类对象本身的大小，而不是字符串内容的长度。**`std::string` 对象通常包含一些额外的成员变量用于管理字符串**，如指向动态分配内存的指针、字符串长度、缓冲区大小等。这个大小通常远大于实际字符串内容的长度。

   - 而 `error.size()` 返回的是字符串中字符的数量，不包括结尾的空字符，它准确地反映了字符串内容的长度，这是在向网络发送数据时所需要的正确长度。

2. 因为 `sendto` 函数需要**知道目标地址结构体的大小**，以便正确地处理和解析地址信息。**不同类型的地址结构体（如 `sockaddr_in`、`sockaddr_in6` 等）可能有不同的大小**，通过传递正确的大小，函数可以确保在处理地址时不会访问超出结构体范围的内存，从而保证程序的安全性和正确性。

3. `error.c_str()`返回一个**指向以空字符结尾的字符数组的指针（const char*）**。这个指针指向的内存区域包含了与`error`字符串内容相同的字符序列，但这个指针所指向的内存是临时的，不应该被修改，并且**其有效性与`error`对象的生命周期相关。**

   - 用途：通常用于将`std::string`对象传递给需要以传统 C 风格字符串（以空字符结尾的字符数组）作为参数的函数。例如，一些 C 语言的函数或者**某些 C++ 函数接口要求传入 C 风格字符串**，这时可以使用`c_str()`来满足这种需求。

### 12. 常量指针和指针常量区别

- 常量指针，**指向的对象是常量**，用于函数传参以确保函数内部不能修改传入的对象。例如：`void printValue(const int* value);`，在这个函数中，不能通过`value`指针修改所指向的整数。
- 指针常量，**指针本体是常量**

### 13. a和&a的区别

- a值是数组首元素的地址；&a值是整个数组的地址，例如，`&a + 1` 会跳过整个数组的大小，指向数组后面的下一个位置，但这个位置通常在程序中没有实际意义。

```c
  int a[5] = {1, 2, 3, 4, 5};
  printf("a = %p\n", a);            // 输出数组首元素地址
  printf("&a = %p\n", &a);          // 输出整个数组地址
  printf("a + 1 = %p\n", a + 1);    // 指向数组第二个元素地址
  printf("&a + 1 = %p\n", &a + 1);  // 跳过整个数组的地址
```

![image-20240902140752292](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240902140752292.png)

### 14. C++和python区别

- c++是**编译语言，编译时将代码转为机器指令效率高**，先编译后在**特定平台**允许；python是**解释型语言**，**解释器逐行分析源代码，将其转换为中间代码或直接转换为机器指令解释器执行转换后的代码，输出结果**，因此有**解释器**就可以很**方便跨平台**

- python用缩进区分不同代码块；c++用花括号

  

### 15. C++和C区别

- C没有**字符串**类型，没有**引用**类型，不可以**函数重载**
- iostream取代了stdio；try/catch/throw取代了setjmp和longjmp
- C需要**手动管理内存**malloc free；C++有手动和自动new delete std::unique_ptr智能指针等
- C是**面向过程**的编程语言，以**过程为中心**，将数据和对数据的操作分离，注重算法和数据结构的实现；C++**面向对象、面向泛型、面向过程的多范式编程语言**，例如，可以使用 C++ 的类来封装数据和操作，提高代码的可维护性和可扩展性。同时，C++ 也支持模板和泛型编程，允许编写通用的代码，提高代码的复用性。

