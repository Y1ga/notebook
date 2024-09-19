# protobuf

## 定义

通信协议一般用分层模型，**不同模型功能定义及颗粒度不同**，例如：TCP/IP协议是一个**四层**协议，OSI模型是**7层**协议。OSI7层模型中展现层(Presentation Layer)功能是把**应用层对象(class)转换成一段连续的二进制串即序列化**，或者反序列化。**TCP/IP协议应用层对应OSI七层协议模型的应用层+展示层+会话层，所以序列化协议属于TCP/IP协议应用层的一部分。** 









## 序列化



### 为什么需要序列化和反序列化

序列化的目的是**将数据转换为一种通用的结构**，这样**其他系统或程序可以轻松地解析和使用这些数据**。这种通用的结构**通常是文本格式，如JSON、XML或YAML，但也可以是二进制格式，如Protocol Buffers**或MessagePack。

1. **数据可持久化**：

   - 序列化：通过将内存中的对象或数据结构转换为可存储的结构，如：二进制、json、xml等，这样数据就可以被保存在文件系统、数据库或其他持久存储介质中。

   - 反序列化：从存储介质中读取序列化的数据，并还原为内存中的对象或数据结构，允许数据在应用程序关闭后得以保留和恢复

2. **网络通信**：
   - 发送方将数据**序列化为可传输的格式**，数据能够在不同的计算机之间通过网络传递。接收方通过反序列化将接收到的数据还原成原始对象和数据结构，以便在本地使用。

3. **跨语言**： 



### 序列化协议特性

#### 1. 通用性

第一、技术层面，序列化协议是否支持**跨平台、跨语言**。如果不支持，在技术层面上的通用性就大大降低了。

第二、**流行程度**，序列化和反序列化需要多方参与，很少人使用的协议往往意味着昂贵的学习成本；另一方面，流行度低的协议，往往缺乏稳定而成熟的跨语言、跨平台的公共包。

#### 2. 强健性、鲁棒性

**成熟度**不够，一个协议从制定到实施，到最后成熟往往是一个漫长的阶段。协议的强健性依赖于大量而全面的测试，对于致力于提供高质量服务的系统，采用处于测试阶段的序列化协议会带来很高的风险。

#### 3. 可调试性、可读性

序列化和反序列化的数据正确性和业务正确性的**调试往往需要很长的时间**，良好的调试机制会大大提高开发效率。序列化后的**二进制串往往不具备人眼可读性**，为了验证序列化结果的正确性，写入方不得同时撰写反序列化程序，或提供一个查询平台–这比较费时；另一方面，如果读取方未能成功实现反序列化，这将给问题查找带来了很大的挑战–难以定位是由于自身的反序列化程序的bug所导致还是由于写入方序列化后的错误数据所导致。

#### 4.  性能

第一、**空间开销（Verbosity）**， 序列化需要在原有的数据上加上描述字段，以为反序列化解析之用。如果序列化过程引入的额外开销过高，可能会导致过大的网络，磁盘等各方面的压力。对于海量分布式存储系统，数据量往往以TB为单位，巨大的的额外空间开销意味着高昂的成本。

第二、**时间开销（Complexity）**，复杂的序列化协议会导致较长的解析时间，这可能会使得序列化和反序列化阶段成为整个系统的瓶颈。

#### 5. 可拓展性/兼容性

移动互联时代，业务系统需求的更新周期变得更快，新的需求不断涌现，而老的系统还是需要继续维护。如果序列化协议具有良好的可扩展性，支持自动增加新的业务字段，而不影响老的服务，这将大大提供系统的灵活度。

#### 6. 安全性/访问限制

在序列化选型的过程中，安全性的考虑往往发生在跨局域网访问的场景。当通讯发生在公司之间或者跨机房的时候，出于安全的考虑，对于**跨局域网的访问往往被限制为基于HTTP/HTTPS的80和443端口**。如果使用的序列化协议没有兼容而成熟的HTTP传输层框架支持，可能会导致以下三种结果之一：

第一、因为访问限制而降低服务可用性。 第二、被迫重新实现安全协议而导致实施成本大大提高。 第三、开放更多的防火墙端口和协议访问，而牺牲安全性。

### 数据结构、对象与二进制串

- 序列化：将**数据结构(struct)或对象(class)转换成二进制串(byte[])**的过程
- 反序列化：将在序列化过程中生成的**二进制串转换成数据结构或者对象**的过程

### 序列化底层原理

1. **IDL(Interface Description Language)**文件：为了建立一个与语言和平台无关的约定，需要用**与具体开发语言、开放平台无关的约定**进行描述，这种语言称为**接口描述语言**，采用IDL撰写的协议约定称为IDL文件，例如：protobuf对应的idl文件叫.proto

   ![image-20240808110541375](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808110541375.png)

2. IDL Complier：对应IDL语言的的编译器，将IDL文件转换成各语言对应的**动态库**，例如：protobuf对应的编译器叫**protobuf_complier，简称protoc**

3. Stub/Skeleton Lib：**负责序列化和反序列化的具体工作代码**

   记住**传输过程使用的是序列化的数据**即可

   - stub：是一段部署在分布式系统客户端的代码，一方面**接收应用层的参数，并对其序列化**后通过底层协议栈发送到服务端，另一方面**接收服务端序列化后的结果数据，反序列化**后交给客户端应用层
   - skeleton：部署在服务端，功能与stub相反，从**传输层接收序列化参数，反序列化**后交给服务端应用层，并**将应用层的执行结果序列化后传送给客户端stub**

4. Client/Server：指的是**应用层程序代码**，应用层面对的是IDL生成的**特定语言的class或struct**，例如c++对应编译后的.pb.h

   ```Plaintext
   protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR  path/to/file.proto
   # 指定特定语言
   --cpp_out生成 C++ 代码存储在DST_DIR
   --java_out生成 Java 代码存储在DST_DIR
   ```

5. 底层协议栈和互联网：序列化之后的数据**通过底层的传输层、网络层、链路层以及物理层协议转换成数字信号**在互联网中传递。

![image-20240808121114132](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808121114132.png)



![image-20240808122245604](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808122245604.png)

### 常见序列化

#### XML&SOAP

- XML(eXtensible Markup Language，可拓展标记语言)
- SOAP(Simple Object Access Protocol，简单对象访问协议)

##### XML

1. XML是一种常用的序列化和反序列化协议，跨机器、跨语言。设计之初用于对互联网文档进行标记，因此具有人和机器兼具可读性。
2. 但作为描述语言用于序列化对象显得冗余复杂，因此一般用于配置文件中，例如maven里的pom.xml

##### SOAP

1. 基于XML为序列化和反序列化的结构化消息传递协议，在互联网影响太大以至于我们给基于SOAP的解决方案一个特定的名称–Web service，常见的使用方式是**XML+HTTP**。SOAP协议的IDL是**WSDL(Web Service Description Language, web描述语言)**

#### 区别

- xml简单好调试，适合于**传输量小和实时性低（秒级）**例如公司之间的通信
- xml**空间和时间开销大**，soap虽然是simple但绝对不简单，wsdl文件不直观。

#### JSON(Javascript Object Notation)

起源于JavaScript，采用"attribute - value"的方式描述对象，web典型应用是json+http，适合跨防火墙访问

优点：

1. 保持了xml的人眼可读(Human-readable)优点
2. 序列化数据更简洁
3. 与XML相比，协议比较简单，解析速度比较快
4. JSON太像语言里面的类，因此进行序列化时不需要IDL，是一种天然的序列化协议

应用场景：

1. 公司之间传输数据量小，实时性要求低
2. json有很强的前后兼容性，对于**接口经常发生变化**，并对可调性要求高的场景
3. JSON对序列化的**内存和磁盘开销大**，由于在一些语言的序列化和反序列化需要使用**反射机制**因此实时性较低**（秒级）**

#### protobuf

**空间小，高性能**

1. 有标准的IDL和标准的IDLC
2. 序列化数据非常简洁，是**XML的1/3到1/10**
3. 解析速度快，比XML**快20-100倍**
4. 动态库好用，**反序列化只需要一行代码** 
5. 是**纯粹的展示层协议**，可以和各种传输协议一起使用
6. 由于产生于Google，因此只支持Java、C++、python三种语言

## protbuf基础知识

### 数据类型

![image-20240808113106590](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808113106590.png)

### proto文件最终生成了什么

当运行 [protocol buffer compiler](https://protobuf.dev/programming-guides/proto3/#generating) 编译test.proto时，编译器会以您选择的语言生成代码，您需要使用文件中描述的消息类型，包括获取和设置字段值、将消息序列化为输出流，并从输入流解析消息。

- 对于**C++**，编译器会根据每个 `.proto` 生成一个`.h`和`.cc`文件，其中包含文件中描述的每种消息类型的类。
- 对于**Java**，编译器会生成一个`.java`文件，其中包含每个消息类型的类，以及`Builder`用于创建消息类实例的特殊类。

![image-20240808163024668](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808163024668.png)

### protobuf_generate

**cmake 指令**，把proto文件生成`pb.h`、`pb.cc`

- `APPEND_PATH`— 一个标志，导致所有原型模式文件的基本路径被添加到`IMPORT_DIRS`.
- `LANGUAGE`— 单个值：cpp 或 python。确定正在生成什么类型的源文件。
- `OUT_VAR`— CMake 变量的名称，该变量将填充生成的源文件的路径。
- `EXPORT_MACRO`— 应用于所有生成的 Protobuf 消息类和`extern`变量的宏的名称。
- `PROTOC_OUT_DIR`—生成的源文件的输出目录，默认为`CMAKE_CURRENT_BINARY_DIR`.
- `PLUGIN`— 可选的插件可执行文件。例如，这可能是通往 的路径`grpc_cpp_plugin`。
- `PLUGIN_OPTIONS`— 为插件提供的附加选项，例如`generate_mock_code=true`gRPC cpp 插件。
- `TARGET`— 生成的文件将作为源添加到提供的目标。
- `PROTOS`— 原型模式文件列表。如果省略，则将使用每个以`proto`of结尾的源文件。`TARGET`
- `IMPORT_DIRS`— 模式文件的公共父目录。例如，如果架构文件是`proto/helloworld/helloworld.proto`且导入目录是`proto/`，则生成的文件是`${PROTOC_OUT_DIR}/helloworld/helloworld.pb.cc`.
- `GENERATE_EXTENSIONS`— 如果`LANGUAGE`省略，则必须将其设置为`protoc`生成的扩展。
- `PROTOC_OPTIONS`— 转发到`protoc`调用的其他参数。

### 使用步骤

1. 定义IDL文件即.proto文件：test.proto

   ```protobuf
   syntax = "proto3";
   # package：生成java类的层级
   option java_package = "com.test.protobuf";
   # 若是c++则是 直接package
   package monitor.proto
   
   message CpuLoad {
       float load_avg_1 = 1;
       float load_avg_3 = 2;
       float load_avg_15 = 3;
     }
     
   message NetInfo {
       string name = 1;
       float send_rate = 2;
     }
     
    message MonitorInfo{
     CpuLoad cpu_load = 1; #std::string
     repeated NetInfo net_info = 2; # 此处的repeated相当于C++中的std::vector
   }
   ```

   ![image-20240808163024668](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808163024668.png)

2. 使用protoc编译生成动态库

   cd到/proto所在的目录

   ```sh
   protoc -I =./ --java_out=./ ./test.proto
   # -I：待compile的proto文件所在的目录
   # --java_out：编译生成的java文件存放的目录
   # ./test.proto：待编译的proto文件
   # 这里的三个路径要么全绝对要么全相对，不要混用
   
   # 编译生成.cc和.h的c++文件
   protoc -I =./ --cpp_out=./ ./test.proto
   ```

3. 使用protobuf的API来读写消息，常用API:

   ```cpp
   // 序列化
   bool SerializeToString(string* output) const; // 此处的string是二进制即序列化后的数据，选用string是因为它是个好用的容器
   
   // 反序列化
   bool ParseFromString(const string& data);
   
   // 将消息序列化为数组
   bool SerializeToArray(void * data, int size) const;
   
   // 将数组反序列化
   bool ParseFromArray(const void * data, int size);
   
   // 将消息序列化到C++的ostream中
   bool SerializeToOstream(ostream * output) const;
   
   // 将C++ isteam的数据反序列化
   bool ParseFromStream(istream * input);
   ```

   ```protobuf
   syntax = "proto3";
   package monitor.proto;
   
   # message相当于class的意思
   message CpuLoad {
       float load_avg_1 = 1;
       float load_avg_3 = 2;
       float load_avg_15 = 3;
     }
     
   message NetInfo {
       string name = 1;
       float send_rate = 2;
     }
     
   message MonitorInfo {
     std::string happly = 1;
     CpuLoad cpu_load = 2; #std::string
     repeated NetInfo net_info = 3; # std::vector
   }
   
   
   monitor::proto::MonitorInfo monitor_info;
   monitor_info.set_happly("1111");
   
   ::monitor::proto::CpuLoad* cpu_load_msg
   cpu_load_msg->set_load_avg_3(1.4);
   cpu_load_msg->set_load_avg_15(1.8); 
   
   ::monitor::proto::NetInfo*  net_info_msg1  = monitor_info.add_net_info();#适用于多个类型
   net_info_msg1->set_name("super-1");
   net_info_msg1->set_send_rate(12.5);
   
   auto net_info_msg2  = monitor_info.add_net_info();
   net_info_msg2->set_name("super-2");
   net_info_msg2->set_send_rate(8.5);
   
    // 对消息对象MonitorInfo序列化到string容器
   std::string serializedStr;
   monitor_info.SerializeToString(&serializedStr);
   std::cout<<"serialization result:"<<serializedStr<<std::endl; //序列化后的字符串内容是二进制内容，非可打印字符，预计输出乱码
   
   
   //反序列化
   monitor::proto::MonitorInfo monitor_info;
   monitor_info.ParseFromString(serializedStr)；
       
   std::cout << monitor_info.happly()<<std::endl;
   
   
   message CpuLoad {
       float load_avg_1 = 1;
       float load_avg_3 = 2;
       float load_avg_15 = 3;
     }
     
   auto cpu_load_parse =  monitor_info.cpu_load();
   std::cout << cpu_load_parse.load_avg_1()<< cpu_load_parse.load_avg_3()<<std::endl;
   
   
   for (int i = 0; i < monitor_info.net_info_size(); i++) {
           std::cout <<monitor_info.net_info(i).name();
           std::cout << monitor_info.net_info(i).send_rate();
   }
   ```

   

​	

