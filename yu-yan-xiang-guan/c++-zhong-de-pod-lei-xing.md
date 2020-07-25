# C++中的POD类型

### POD类型

POD类型是C++中常见的概念，用来说明类/结构体的属性，具体来说它是指**没有使用面向对象的思想来设计的类/结构体**

POD的全称是Plain Old Data，Plain表明它是一个普通的类型，没有虚函数虚继承等特性；Old表明它与C兼容。

**特性：**

POD类型在C++中有两个独立的特性：

* 支持静态初始化（static initialization）
* 拥有和C语言一样的内存布局（memory layout）

这两个特性分布对应两个概念：trivial classes和standard-layout。现在提起POD类型通常是指有这两个特性的类，且这个类的非静态成员也是POD类型。

### **POD优点**

POD类型相对非POD类型有以下优点：  
1、字节赋值。POD类型变量可以不使用构造函数、赋值操作符赋值，直接通过`memset()`、`memcpy()`初始化赋值。  
2、兼容C内存布局。C++程序可以和C进行交互，或者可以和其他语言交互。  
3、保证静态初始化安全有效。静态初始化很多时候可以提高程序性能，POD类型初始化更加简单。

### **Trivial classes**

 trivial classes支持静态初始化（static initizlization），如果一个类包含以下特点，那么它就是trivial class:

* 所有的拷贝构造函数都是trivial
* 所有的移动构造函数都是trivial
* 所有鹅赋值操作符都是trivial
* 所有的移动赋值操作符都是trivial
* 默认构造函数和析构函数是trivial

这里说的trivial构造函数是编译器生成的构造函数，而不是用户自定义的；且它的基类也有这样的特性。C++11中的模版`template <typename T> struct std::is_trivial`可判断类是否是trivial。

### **Standard-layout**

这里的standard是指可以和其他语言通信，因为standard-lay类型的内部布局和C结构体一样。Standard layout定义如下：

* 所有非静态成员都是standard-layout
* 没有虚函数和虚基类
* 非静态成员访问控制权一样
* 基类是standard-lay
* 没有静态成员变量，或者在整个继承树中，只有一个类有静态成员变量。
* 第一个非静态成员不是基类

C++11中可以使用`template <typename T>struct std::is_standard_layout`判断一个类是否是standard-layout。

### **vector不能代替C数组的原因**

vector不是POD

```cpp
struct A {
int a;
char buff[100]; // 这里用数组
int b;
};

struct B {
int a;
std::vector<char> buff; // 这里用vector
int b;
};
```

对于以上两个结构体，A用的是数组，对与A类型的变量，可以用memset将整个变量清零，可以用memcpy将内容进行完全拷贝。而B就行不行，因为它里面有一个vector。

用POD来表达数据，无论嵌套多少层，最终的数据依然都是一块连续的内存，可以方便拷贝、清除，可以直接dump到硬盘上去，或是直接通过socket send到另一个进程。嵌套的struct例如：

```cpp
struct C {
int c;
A buffs[100]; // 嵌套struct A类型
int d;
};
```

如果用改成vector，就要写拷贝构造函数来拷贝，还要写一个类似clear的函数来清除。写起来更复杂，效率上也更差。

