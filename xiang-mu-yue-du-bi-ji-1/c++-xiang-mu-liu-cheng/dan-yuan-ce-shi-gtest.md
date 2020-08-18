# 单元测试，gtest

单元测试详细：单元测试到底是什么？应该怎么做？ - 腾讯技术工程的回答 - 知乎 [https://www.zhihu.com/question/28729261/answer/1058317111](https://www.zhihu.com/question/28729261/answer/1058317111)

测试根据粒度分类可以分为单元测试，集成测试与系统测试

广义的单元测试，我们指这三部分的有机组合：

* code review
* 静态代码扫描
* 单元测试用例编写

开发同学写单测，测试同学具有写单测的能力。重点在于开发脚手架、分层测试/端到端测试

### GTest\(Google Test测试框架\)

gtest 是谷歌的 C++ 单元测试框架，安装步骤：

```bash
$ git clone https://github.com/google/googletest.git
$ cd googletest
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install
```

安装 gtest 之后，要怎样在 CMake 中使用 gtest 呢？让我们用一个简单的例子演示一下，首先编写项目的 CMakeLists.txt 文件：

```bash
cmake_minimum_required (VERSION 2.8)

project(code-for-blog)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -std=c++11 -Wall")
find_package(GTest REQUIRED)
find_package(Threads REQUIRED)

include_directories(${GTEST_INCLUDE_DIRS})

add_executable(MyTests test.cpp)
target_link_libraries(MyTests ${GTEST_BOTH_LIBRARIES})
target_link_libraries(MyTests ${CMAKE_THREAD_LIBS_INIT})

add_test(Test MyTests)
enable_testing()
```

接着编写一个简单的单元测试文件 test.cpp：

```cpp
#include <gtest/gtest.h>
#include <numeric>
#include <vector>
// 测试集为 MyTest，测试案例为 Sum
TEST(MyTest, Sum)
{
    std::vector<int> vec{1, 2, 3, 4, 5};
    int sum = std::accumulate(vec.begin(), vec.end(), 0);
    EXPECT_EQ(sum, 15);
}
int main(int argc, char *argv[])
{
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

 编译好项目之后，使用命令`make test`就可以执行单元测试了：

```cpp
$ make test
Running tests...
Test project /Users/senlin/my-test/build
    Start 1: Test
1/1 Test #1: Test .............................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
```

更多的gtest测试用法：[http://senlinzhan.github.io/2017/10/08/gtest/](http://senlinzhan.github.io/2017/10/08/gtest/)

