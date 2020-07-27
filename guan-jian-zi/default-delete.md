# =default, =delete

c++11新特性

### Defaulted 函数 =default

如果为了定义自己的构造函数，但是同时又不让`X x`编译报错，所以需要定义两个构造函数

（参考**默认构造函数章节**）

需要这样写：

```cpp
class X{ 
public: 
 X(){};  // 手动定义默认构造函数
 X(int i){ 
   a = i; 
 }     
private: 
 int a; 
}; 
 
X x;   // 正确，默认构造函数 X::X() 存在
```

这样的问题：

1. 多写一个函数，增加程序员工作量
2. 手动编写的特殊成员函数的代码执行效率比编译器自动生成的特殊成员函数低。

**=default是为了解决这个问题**

* 减轻程序员的编程工作量
* 获得编译器自动生成的默认特殊成员函数的高的代码执行效率

```cpp
class X{ 
public: 
 X()= default; 
 X(int i){ 
   a = i; 
 }     
private: 
 int a; 
}; 
 
X x;
```

编译器会自动生成默认构造函数 `X::X(){}`，该函数可以比用户自己定义的默认构造函数获得更高的代码效率。

###  =default**特性**

1.Defaulted 函数特性仅适用于类的特殊成员函数，且该**特殊成员函数**没有默认参数。例如：

\(特殊成员函数：默认构造函数、析构函数、拷贝构造函数以及拷贝赋值运算符）

```cpp
class X { 
public: 
 int f() = default;      // 错误 , 函数 f() 非类 X 的特殊成员函数
 X(int) = default;       // 错误 , 构造函数 X(int, int) 非 X 的特殊成员函数
 X(int = 1) = default;   // 错误 , 默认构造函数 X(int=1) 含有默认参数
};
```

2.**default函数在类内定义默认为inline**，也可以在类体外（out-of-line）定义

```cpp
class X{ 
public:  
  X() = default; //Inline defaulted 默认构造函数
  X(const X&); 
  X& operator = (const X&); 
  ~X() = default;  //Inline defaulted 析构函数
}; 
 
X::X(const X&) = default;  //Out-of-line defaulted 拷贝构造函数
X& X::operator = (const X&) = default;     //Out-of-line defaulted  
    // 拷贝赋值操作符
```

### =default**解决继承中析构的问题**

**问题：下面这个代码段**

```cpp
class X { 
private: 
 int x; 
}; 
 
class Y: public X { 
private: 
 int y; 
}; 
 
int main(){ 
 X* x = new Y; 
 delete x; 
}
```

 这段代码存在内存泄露的问题，当利用 `delete` 语句删除指向派生类对象的指针 `x` 时，系统调用的是基类的析构函数，而非派生类 `Y` 类的析构函数，因此，编译器无法析构派生类的 `int` 型成员变量 y

 一般情况下我们需要将基类的析构函数定义为虚函数，当利用 delete 语句删除指向派生类对象的基类指针时，系统会调用相应的派生类的析构函数（实现多态性），从而避免内存泄露。但是编译器隐式自动生成的析构函数都是非虚函数，这就需要由程序员手动的为基类 `X` 定义虚析构函数

```cpp
class X { 
public: 
 virtual ~X(){};     // 手动定义虚析构函数
private: 
 int x; 
}; 
 
class Y: public X { 
private: 
 int y; 
}; 
 
int main(){ 
 X* x = new Y; 
 delete x; 
 }
```

手动定义的析构函数的代码执行效率要低于编译器自动生成的析构函数。

为了解决上述问题，我们可以将基类的虚析构函数声明为 defaulted 函数，这样就可以显式的指定编译器为该函数自动生成函数体。例如：

```cpp
class X { 
public: 
 virtual ~X()= default; // 编译器自动生成 defaulted 函数定义体
private: 
 int x; 
}; 
 
class Y: public X { 
private: 
 int y; 
}; 
 
int main(){ 
 X* x = new Y; 
 delete x;
 }
```

### Deleted 函数 <a id="major2"></a>

假设我们不允许发生类对象之间的拷贝和赋值，可是又无法阻止编译器隐式自动生成默认的拷贝构造函数以及拷贝赋值操作符, 程序员只需在函数声明后加上“`=delete;`”，就可将该函数禁用。

```cpp
class X{            
     public: 
       X(); 
       X(const X&) = delete;  // 声明拷贝构造函数为 deleted 函数
       X& operator = (const X &) = delete; // 声明拷贝赋值操作符为 deleted 函数
     }; 
 
 int main(){ 
  X x1; 
  X x2=x1;   // 错误，拷贝构造函数被禁用
  X x3; 
  x3=x1;     // 错误，拷贝赋值操作符被禁用
 }
```

### Deleted 函数的特殊用法 <a id="major2"></a>

**可用于禁用类的某些转换构造函数，从而避免不期望的类型转换**

```cpp
class X{ 
public: 
 X(double);              
 X(int) = delete;     
}; 
 
int main(){ 
 X x1(1.2);        
 X x2(2); // 错误，参数为整数 int 类型的转换构造函数被禁用          
}
```

 用来禁用某些用户自定义的类的 `new` 操作符，从而避免在自由存储区创建类的对象

```cpp
#include <cstddef> 
using namespace std; 
 
class X{ 
public: 
 void *operator new(size_t) = delete; 
 void *operator new[](size_t) = delete; 
}; 
 
int main(){ 
 X *pa = new X;  // 错误，new 操作符被禁用
 X *pb = new X[10];  // 错误，new[] 操作符被禁用
}
```

deleted 函数必须在类体里（inline）定义，而不能在类体外（out-of-line）定义

