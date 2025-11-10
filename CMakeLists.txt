cmake_minimum_required(VERSION 3.14)
project(Three-Address-Code-Generator-for-C LANGUAGES C)

# 查找 Flex 和 Bison
find_package(FLEX REQUIRED)
find_package(BISON REQUIRED)

# 启用调试（可选）
add_definitions(-DYYDEBUG=1)

BISON_TARGET(YParser Yacc.y ${CMAKE_BINARY_DIR}/y.tab.c
        DEFINES_FILE ${CMAKE_BINARY_DIR}/y.tab.h)
FLEX_TARGET(LexScanner myscanner.l ${CMAKE_BINARY_DIR}/lex.yy.c)
ADD_FLEX_BISON_DEPENDENCY(LexScanner YParser)


add_executable(myscanner
        myscanner.c
        ${FLEX_LexScanner_OUTPUTS}
        ${BISON_YParser_OUTPUTS}
)
if(APPLE)
    # 设置 flex 路径
    set(FLEX_HINT_PATHS /usr/local/opt/flex /opt/homebrew/opt/flex)

    # 查找 flex 的 include 目录
    find_path(FLEX_INCLUDE_DIR NAMES location.hh PATHS ${FLEX_HINT_PATHS} PATH_SUFFIXES include)
    find_library(FLEX_LIB NAMES fl PATHS ${FLEX_HINT_PATHS} PATH_SUFFIXES lib)

    if(FLEX_INCLUDE_DIR)
        include_directories(${FLEX_INCLUDE_DIR})
        message(STATUS "找到 Flex 头文件目录: ${FLEX_INCLUDE_DIR}")
    endif()

    if(FLEX_LIB)
        message(STATUS "找到 Flex 库: ${FLEX_LIB}")
    else()
        message(WARNING "未找到 Flex 库，尝试使用系统默认路径")
    endif()
    # 设置 bison 路径
    set(BISON_HINT_PATHS /usr/local/opt/bison /opt/homebrew/opt/bison)

    # 查找 bison 的 include 目录
    find_path(BISON_INCLUDE_DIR NAMES location.hh PATHS ${BISON_HINT_PATHS} PATH_SUFFIXES include)
    find_library(BISON_LIB NAMES y PATHS ${BISON_HINT_PATHS} PATH_SUFFIXES lib)

    if(BISON_INCLUDE_DIR)
        include_directories(${BISON_INCLUDE_DIR})
        message(STATUS "找到 Bison 头文件目录: ${BISON_INCLUDE_DIR}")
    endif()

    if(BISON_LIB)
        message(STATUS "找到 Bison 库: ${BISON_LIB}")
    else()
        message(WARNING "未找到 Bison 库，尝试使用系统默认路径")
    endif()

    # 设置编译器标志
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L/usr/local/opt/bison/lib")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L/usr/local/opt/flex/lib")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L/opt/homebrew/opt/flex/lib -L/opt/homebrew/opt/bison/lib")

elseif(UNIX AND NOT APPLE)
    message(STATUS "配置 Linux 环境")
    # Linux 下的配置
    # 通常 bison 会安装在标准路径，不需要特殊配置
    # 包含生成的头文件
    target_include_directories(myscanner PRIVATE ${CMAKE_BINARY_DIR})
endif()
# 解决 ubuntu-clang 找不到 liby.so 的终极方案
if(CMAKE_C_COMPILER_ID MATCHES "Clang" AND CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message(STATUS "配置 Clang link 目录")
    # 强制告诉 clang 去 gcc 的三元组目录找库
    link_directories(/usr/lib/x86_64-linux-gnu)
endif()
# 修改链接命令，使用找到的库变量而不是 -lfl -ly
if(FLEX_LIB AND BISON_LIB)
    target_link_libraries(myscanner ${FLEX_LIB} ${BISON_LIB})
else()
    message(STATUS "配置 link 目录")
    target_link_libraries(myscanner fl y)
endif()