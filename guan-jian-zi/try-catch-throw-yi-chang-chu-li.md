# try, catch, throw异常处理

### 异常处理

C**++引入的异常机制，导致一些深刻的改变**

* 使用未经处理的或原始的C指针变得很危险；
* 资源泄漏的可能性增加了；
* 写出具有你希望的行为的构造函数与析构函数变得更加困难。
* 特别小心防止程序执行时 突然崩溃。
* 运行速度降低了

需要建立异常安全（Exception-safe）的程序：程序能够在存在异常的情况下正常运行

**C语言风格的错误码为什么不好呢？**如果一个函数通过设置一个状态变量或返回错误代码来表示 一个异常状态，没有办法保证函数调用者将一定检测变量或测试错误代码。结果程序会从它 遇到的异常状态继续运行，异常没有被捕获，程序立即会终止执行

### 异常实现原理

* 当throw出现在一个try块内时，检查与该块相关联的catch语句，如果找到了匹配的catch，就使用该catch处理异常
* 如果这一步没找到匹配的catch且该try语句嵌套在其他try块中，则继续检查与外层try匹配的catch子句
* 如果还是找不到匹配的match，则退出当前的函数，在调用当前函数的外层函数中继续寻找
* **（上述的一系列过程叫做栈展开）**

### 异常处理基本步骤

**抛出异常**

```cpp
double division(int a, int b)
{
   if( b == 0 )
   {
      throw "Division by zero condition!";
   }
   return (a/b);
}
```

**捕获异常**

```cpp
try
{
   // 保护代码
}catch( ExceptionName e1 )
{
   // catch 块
}catch( ExceptionName e2 )
{
   // catch 块
}catch( ExceptionName eN )
{
   // catch 块
}
```

**示例：抛出一个类型为const char\*的异常**

**注意异常对象，在外面声明可以，在catch语句里面声明也可以！！**

```cpp
#include <iostream>
using namespace std;
 
double division(int a, int b)
{
   if( b == 0 )
   {
      throw "Division by zero condition!";
   }
   return (a/b);
}
 
int main ()
{
   int x = 50;
   int y = 0;
   double z = 0;
 
   try {
     z = division(x, y);
     cout << z << endl;
   }catch (const char* msg) {//异常对象在catch语句里面声明
     cerr << msg << endl;
   }
 
   return 0;
}
```

### C++标准的异常

![](../.gitbook/assets/exceptions_in_cpp.png)

### noexcept声明

说明这个函数里不会抛出异常，这样声明后可以提高代码的执行效率

### 自定义异常类，用到what方法

```cpp
#include <iostream>
#include <exception>
using namespace std;
 
struct MyException : public exception
{
  const char * what () const throw ()
  {
    return "C++ Exception";
  }
};
 
int main()
{
  try
  {
    throw MyException();
  }
  catch(MyException& e)
  {
    std::cout << "MyException caught" << std::endl;
    std::cout << e.what() << std::endl;
  }
  catch(std::exception& e)
  {
    //其他的错误
  }
}
```

