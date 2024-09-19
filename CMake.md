# CMake

CMakeList.txt文件内，指令类推荐用小写



## example

### 1. hello world

```cmake
# 设置最低cmake版本
cmake_minimum_required(VERSION 3.15)
# 设置项目名
project(hello_cmake)
# 设置build后生成的exe文件
add_executable(hello_cmake_bin main.cpp)
```

build

```shell
mkdir build
cd build
cmake ..
make 
./hello_cmake
```

### 

### 2. 添加依赖库

```cmake
cmake_minimum_required(VERSION 3.15)
project(hello_cmake)
# 添加依赖
add_library(hello_library STATIC
	src/Hello.cpp)
# 更改别名
add_library(hello::library ALIAS hello_library)
# 引入头文件
target_include_directories(hello::library
	PUBLIC
		${PROJECT_SOURCE_DIR}/include # PROJECT_SOURCE_DIR：cmakefiles所在的目录；
									  # PROJECT_BINARY_DIR：cmake编译后所在的目录，例如/build
)

add_executable(hello_cmake_bin 
	src/main.cpp)

# 链接依赖
target_link_libraries(hello_cmake_bin
	PRIVATE
		hello::library
)
```



### 3. install

```cmake
cmake_minimum_required(VERSION 3.15)
project(hello_install)
add_library(hello_library SHARED
	src/Hello.cpp)	
target_include_directories(hello_library
	PUBLIC
		${PROJECT_SOURCE_DIR}/include
)
add_executable(hello_install_bin
	src/main.cpp
)
target_link_libraries(hello_install_bin
	PRIVATE
		hello_library
)

# 将可执行文件安装到/usr/local/bin目录下，文件名：hello_install_bin
install(TARGETS hello_install
	DESTINATION bin)
# 将library库文件安装到/usr/local/lib目录下，文件名：hello_library.so
install(TARGETS hello_library
	LIBRARY DESTINATION lib)
# 头文件,注意用的是directory
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
	DESTINATION include)
# 注意用的是file，配置(config)文件，将cmake-examples.conf安装到/usr/local/etc目录下
install(FILES cmake-examples.conf
	DESTINATION etc)
```

```shell
mkdir build
cd build
cmake ..
make 

sudo make install
# 卸载
sudo xargs rm < install_manifest.txt
```



### 4. CPack

```cmake
cmake_minimum_required(VERSION 3.15)
project(hello_cpack)
add_library(library SHARED
	src/hello.cpp
)
target_include_directories(hello_cpack
	PUBLIC
		${PROJECT_SOURCE_DIR}/include
)
add_executable(hello_cpack_bin
	src/main.cpp
)
target_link_libraries(hello_cpack_bin
	PUBLIC
		library
)

install(TARGETS hello_cpack_bin
	DESTINATION bin
)
install(TARGETS library
	LIBRARY DESTINATION lib
)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
	DESTINATION include
)
install(FILE cmake-examples.conf
	DESTINATION etc
)

# 设置打包的文件类型
set(CPACK_GENERATOR "DEB")
# 设置打包人
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "yiga")
# 设置包版本号
set(CPACK_PACKAGE_VERSION "0.2.1")
# 打包启动！
include(Cpack)
```

```shell
mkdir build
cd build
cmake ..
make 

make help
make package
# 会将Debian软件包放在${PROJECT_BINARY_DIR}目录下
```



### 5. 利用头文件模板生成头文件

```cmake
cmake_minimum_required(VERSION 3.5)
project(hello_cg)

set(hello_cg_VERSION_MAJOR 0)
set(hello_cg_VERSION_MINOR 2)
set(hello_cg_VERSION_PATCH 1)
set(hello_cg_VERSION"${hello_cg_VERSION_MAJOR}.${hello_cg_VERSION_MINOR}.${hello_cg_VERSION_PATCH}")

# 根据ver.h.in头文件模板在PROJECT_BINARY_DIR目录下生成头文件
# 在cmake构建过程（即在make之前）已生成
configure_file(ver.h.in ${PROJECT_BINARY_DIR}/ver.h)
# @ONLY：只有使用@VAR@的方式才能引用被改变的量
configure_file(path.h.in ${PROJECT_BINARY_DIR}/path.h @ONLY)

add_executable(hello_cg_bin
	src/main.cpp
)
# 引用新生成的头文件目录
target_include_directories(hello_cg
	PUBLIC
		${PROJECT_BINARY_DIR}
)
```

`path.h.in`

```h
#ifndef __PATH_H__
#define __PATH_H__

// version variable that will be substituted by cmake
// This shows an example using the @ variable type
// 此处的@CMAKE_SOURCE_DIR@来自于camke文件
const char *path = "@CMAKE_SOURCE_DIR@";

#endif
```

`ver.h.in`

```h
#ifndef __VER_H__
#define __VER_H__

// version variable that will be substituted by cmake
// This shows an example using the $ variable type
// 此处的$hello_cg_VERSION来自于camke文件定义
const char* ver = "${hello_cg_VERSION}";

#endif
```

`path.h`

```h
#ifndef __PATH_H__
#define __PATH_H__

// version variable that will be substituted by cmake
// This shows an example using the @ variable type
const char *path = "/home/yiga/work/cmake-examples/03-code-generation/configure-files";

#endif
```

`ver.h`

```h
#ifndef __VER_H__
#define __VER_H__

// version variable that will be substituted by cmake
// This shows an example using the $ variable type
const char* ver = "0.2.1";

#endif
```

### 6. 子项目

```cmake
cmake_minimum_required(VERSION 3.15)
project(subprojects)
# 加入子项目
add_subdirectory(sublibrary)
add_subdirectory(sublibrary1)
add_subdirectory(sublibrary2)
```

`subbinary/CMakeLists.txt`

```cmake
project(subbinary)
add_executable(${PROJECT_NAME} main.cpp)

# Link the static library from subproject1 using its alias sub::lib1
# Link the header only library from subproject2 using its alias sub::lib2
# This will cause the include directories for that target to be added to this project
# 这里居然能直接识别sub::lib1？
target_link_libraries(${PROJECT_NAME}
    sub::lib1
    sub::lib2
)
```

`subbinary1/CMakeLists.txt`

```cmake
project (sublibrary1)
add_library(${PROJECT_NAME} src/sublib1.cpp)
add_library(sub::lib1 ALIAS ${PROJECT_NAME})
target_include_directories( ${PROJECT_NAME}
    PUBLIC ${PROJECT_SOURCE_DIR}/include
)

```

`subbinary2/CMakeLists.txt`

```cmake
project (sublibrary2)
add_library(${PROJECT_NAME} INTERFACE)
add_library(sub::lib2 ALIAS ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
    INTERFACE
        ${PROJECT_SOURCE_DIR}/include
)
```

### 7. 引入第三方包

```cmake
cmake_minimum_required(VERSION 3.5)
project (imported_targets)

# 在本地找名为Boost的包，要求版本号1.46.1 且一定要有filesystem 和 system部件
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)
add_executable(imported_targets main.cpp)

# 将第三方包链接bin文件
target_link_libraries( imported_targets
    PRIVATE
        Boost::filesystem
)

```



### 8. C++规范

设置C++11规范

```cmake
set(CMAKE_CXX_STANDARD 11)
```

使用`clang`编译

```cmake
mkdir build.clang
cd build.clang/
cmake .. -DCMAKE_C_COMPILER=clang-3.6 -DCMAKE_CXX_COMPILER=clang++-3.6
make VERBOSE=1
```

使用`ninja`编译

```bash
mkdir build.ninja
cd build.ninja/
cmake .. -G Ninja
cd build.ninja/
ninja -v
```

