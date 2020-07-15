# static

static变量只被初始化一次，所以叫做静态

**静态数据成员**被编译器放在程序的一个global data members中，它是类的一个数据成员，但不影响类的大小。不管这个类产生了多少个实例，还是派生了多少新的类，静态数据成员只有一个实例。静态数据成员，一旦被声明，就已经存在。

C中介绍 **static关键词包含两个功能**

* 在代码块内部声明时，用于修改变量的**存储类型：**如果给代码块内部声明的变量，加上static关键字，可以把它的存储类型由自动变为静态
* 用于函数定义，或者用于代码块之外的声明时，用于修改**标识符链接属性**

### 函数中的静态变量

当变量声明为static时，空间**将在程序的生命周期内分配**。即使多次调用该函数，静态变量的空间也**只分配一次**，前一次调用中的变量值通过下一次函数调用传递。

**注意：static内部定义的值通过函数调用来传递。虽然是初始化代码，但是只初始化一次，每次调用函数时，都不会对变量计数进行初始化。**

```cpp
#include <iostream> 
#include <string> 
using namespace std; 

void demo() 
{ 
    // static variable 
    static int count = 0; 
    cout << count << " "; 

    // value is updated and 
    // will be carried to next 
    // function calls 
    count++; 
} 

int main() 
{ 
    for (int i=0; i<5; i++)  
        demo(); 
    return 0; 
} 
```

输出：

```text
0 1 2 3 4 
```

### **类**中的静态变量

static的变量只被初始化一次，因为它们在单独的静态存储中分配了空间，因此类中的静态变量**由对象共享。**

对于不同的对象，不能有相同静态变量的多个副本。也是因为这个原因，静态变量不能使用构造函数初始化。

**类中静态变量初始化方式：**应由用户使用类外的类名和范围解析运算符显式初始化

```cpp
class Apple 
{ 
public: 
    static int i; 

    Apple() 
    { 
        // Do nothing 
    }; 
}; 

int Apple::i = 1; 
```

### 静态对象

就像变量一样，对象也在声明为static时具有范围，直到程序的生命周期。

静态对象在main结束后调用析构函数。这是因为静态对象的范围是贯穿程序的生命周期。

```cpp
#include<iostream> 
using namespace std; 

class Apple 
{ 
    int i; 
    public: 
        Apple() 
        { 
            i = 0; 
            cout << "Inside Constructor\n"; 
        } 
        ~Apple() 
        { 
            cout << "Inside Destructor\n"; 
        } 
}; 

int main() 
{ 
    int x = 0; 
    if (x==0) 
    { 
        static Apple obj; 
    } 
    cout << "End of main\n"; 
} 
```

输出：

```text
Inside Constructor
End of main
Inside Destructor
```

### **类**中的静态函数

类中的静态数据成员或静态变量一样，静态成员函数也不依赖于类的对象。我们被允许使用对象和'.'来调用静态成员函数。但**建议使用类名和范围解析运算符调用静态成员。**

**静态成员函数无法访问类的非静态数据成员或成员函数。**

```cpp
#include<iostream> 
using namespace std; 

class Apple 
{ 
    public: 
        // static member function 
        static void printMsg() 
        {
            cout<<"Welcome to Apple!"; 
        }
}; 

// main function 
int main() 
{ 
    // invoking a static member function 
    Apple::printMsg(); 
} 
```

输出：

```text
Welcome to Apple!
```

