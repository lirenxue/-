cmake_minimum_required(VERSION 3.10)
project(v3_memory_pool)

# 设置C++标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 编译选项
add_compile_options(-Wall -O2)

# 查找pthread库
find_package(Threads REQUIRED)

# 设置目录
set(SRC_DIR ${CMAKE_SOURCE_DIR}/src)
set(INC_DIR ${CMAKE_SOURCE_DIR}/include)
set(TEST_DIR ${CMAKE_SOURCE_DIR}/tests)

# 源文件
file(GLOB SOURCES "${SRC_DIR}/*.cpp")

# 添加头文件目录
include_directories(${INC_DIR})

# 创建单元测试可执行文件
add_executable(v3_unit_test 
    ${SOURCES}
    ${TEST_DIR}/UnitTest.cpp
)

# 创建性能测试可执行文件
add_executable(v3_perf_test
    ${SOURCES}
    ${TEST_DIR}/PerformanceTest.cpp
)

# 链接pthread库
target_link_libraries(v3_unit_test PRIVATE Threads::Threads)
target_link_libraries(v3_perf_test PRIVATE Threads::Threads)

# 添加测试命令
add_custom_target(test
    COMMAND ./v3_unit_test
    DEPENDS v3_unit_test
)

add_custom_target(perf
    COMMAND ./v3_perf_test
    DEPENDS v3_perf_test
)