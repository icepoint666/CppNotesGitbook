# extern



为标识符指定为external链接属性，这样就可以访问其他任何位置定义的这个实体

**不会随着声明改变而随时改变，一次有效性：**static/extern用于源文件一个标识符的第一次声明，指定具有external属性，但是用于第二次以后再声明时并不会改变属性

除了函数内的变量，其他变量的默认作用域都是external，添加extern关键字在声明前也可以便于让读者理解意图

### extern const实现常量文件间共享

**const对象默认为文件局部变量**

注意：非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。

```cpp
extern const int ext=12;
```

所以一般，对于**const不管是声明还是定义都添加extern关键字**，就可以文件间共享，而且有一个需要初始化

```cpp
//文件1：
extern const int buf = 35;
//文件2：
extern const int buf;
```

### extern "C" C++调用C函数

 C++调用C函数的例子: 引用C的头文件时，需要加`extern "C"`

```cpp
//add.h
#ifndef ADD_H
#define ADD_H
int add(int x,int y);
#endif

//add.c
#include "add.h"
int add(int x,int y) {
    return x+y;
}

//add.cpp
#include <iostream>
using namespace std;
extern "C" {
    #include "add.h"
}
int main() {
    add(2,3);
    return 0;
}
```

编译的时候一定要注意，先通过gcc生成中间文件add.o。

```text
gcc -c add.c 
```

然后编译：

```text
g++ add.cpp add.o -o main
```

**经常用于c++中调用c库头文件**

### **extern "C" C调用C++函数**

 `extern "C"`在C中是语法错误，需要放在C++头文件中。

```cpp
// add.h
#ifndef ADD_H
#define ADD_H
extern "C" {
    int add(int x,int y);
}
#endif

// add.cpp
#include "add.h"

int add(int x,int y) {
    return x+y;
}

// add.c
extern int add(int x,int y);
int main() {
    add(2,3);
    return 0;
}
```

总结出使用方法，在C语言的头文件中，对其外部函数只能指定为extern类型，C语言中不支持extern "C"声明，在.c文件中包含了extern "C"时会出现编译语法错误。所以使用extern "C"全部都放在于cpp程序相关文件或其头文件中。

