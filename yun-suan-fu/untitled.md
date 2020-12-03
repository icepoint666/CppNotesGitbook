# sizeof，内存对齐

sizeof是运算符，可以查看**类型字节大小**，也可以查看**字符串字节大小**

### **sizeof查看字符串字节大小**

```cpp
sizeof("天青色等烟雨，而我在等你!"); //输出:38  UTF-8编码，一个中文三字节 + ! + '\0'
```

### sizeof计算类大小原则

* **空类**的大小为**1字节**（但如果里面存了8字节的数据，大小就是8字节，有东西的就不是空类，所以不用加上空类的1字节）
* 类中**虚函数本身**、**成员函数**（包括静态与非静态）和**静态数据成员**都是不占用类对象的存储空间 （因为本身类成员函数并不是真正的封装在对象中）
* 普通继承，派生类继承了所有基类的函数与成员，要按照**字节对齐**来计算大小
* 对于包含虚函数的类，**不管有多少个虚函数，只有一个虚指针**,vptr的大小

  **vptr指针大小是8\(32位操作系统4字节，64位操作系统 8字节\)**

* **虚函数继承**，不管是单继承还是多继承，**都是继承了基类的vptr**

  派生类虚继承多个虚函数会继承所有函数的vptr

**示例0：\(64位\)**

```cpp
typedef enum {T_NULL, T_FALSE} json_type;

typedef struct json_value json_value;
typedef struct json_member json_member;
struct json_value {
    union {
        struct { json_member* m; size_t size, capacity; }o; //24 = 8 + 8 + 8
        struct { json_value*  e; size_t size, capacity; }a; //24 = 8 + 8 + 8
        struct { char* s; size_t len; }s;                   //16 = 8 + 8
        double n;                                           //8
    }u;             //24 = max(24, 24, 16, 8)
    json_type type; //4
};                  //32 (内存对齐)
struct json_member {
    char* k; size_t klen;   //16 = 8 + 8
    json_value v;           //32
};                          //48
int main()
{

  std::cout << sizeof(size_t) << std::endl;      //8：size_t本质是unsigend long int
  std::cout << sizeof(json_type) << std::endl;   //4：enum是int
  std::cout << sizeof(double) << std::endl;      //8
  std::cout << sizeof(char*) << std::endl;       //8：64位指针长度为8
  std::cout << sizeof(json_member) << std::endl; //48
  std::cout << sizeof(json_value) << std::endl;  //32
}
```

**示例1：**

```cpp
#include<iostream>
using namespace std;
class A
{
    public:
        char b;
        virtual void fun() {};
        static int c;
        static int d;
        static int f;
};

int main()
{
    cout<<sizeof(A)<<endl; //16  字节对齐、静态变量不影响类的大小、vptr指针=8
    return 0;
}
```

示例2：

```cpp
#include<iostream>
using namespace std;
class A{
    virtual void fun();
    virtual void fun1();
    virtual void fun2();
    virtual void fun3();
};
int main()
{
    cout<<sizeof(A)<<endl; // 8 不管有多少个虚函数，只有一个虚指针,vptr的大小
    return 0;
}
```

示例3：

```cpp
#include<iostream>
using namespace std;
class A
{
    public:
        char a;
        int b;
};
class B:A
{
    public:
        short a;
        long b;
};
class C
{
    A a;
    char c;
};
class A1
{
    virtual void fun(){}
};
class C1:public A1
{
};

int main()
{
    cout<<sizeof(A)<<endl; // 8
    cout<<sizeof(B)<<endl; // 24
    cout<<sizeof(C)<<endl; // 12
    cout<<sizeof(C1)<<endl; // 8  对于虚单函数继承，派生类也继承了基类的vptr，所以是8字节
    return 0;
}
```

示例4：

```cpp
#include<iostream>
using namespace std;
class A
{
    virtual void fun() {}
};
class B
{
    virtual void fun2() {}
};
class C : virtual public  A, virtual public B
{
    public:
        virtual void fun3() {}
};

int main()
{
    cout<<sizeof(A)<<" "<<sizeof(B)<<" "<<sizeof(C);
    //8 8 16  派生类虚继承多个虚函数，会继承所有虚函数的vptr
    return 0;
}
```

### 内存对齐是什么

概念：实际的计算机系统**对基本类型数据在内存中存放的位置有限制**，它们会**要求这些数据的首地址的值是某个数k（通常它为4或8）的倍数**，这就是所谓的**内存对齐**。

结构体第一个成员的**偏移量（offset）**为0，以后每个成员相对于结构体首地址的 offset 都是**该成员大小与有效对齐值中较小那个**的**整数倍**，如有需要编译器会在成员之间加上填充字节。是**编译器**加的**乱码**来**填充**字节

**32位系统**下，int占4byte，char占一个byte，那么将它们放到一个结构体中应该占4+1=5byte；但是实际上，通过运行程序得到的结果是8 byte，这就是内存对齐所导致的。

```cpp
struct{
    int x;
    char y;
}s;

sizeof(s)  // 输出8
```

### 为什么要内存对齐

大部分**处理器并不是按字节块来存取内存的**.它一般会以双字节,四字节,8字节,16字节甚至32字节为单位来存取内存。上述这些存取单位称为**内存存取粒度**

**例如对于32位系统，寄存器大小是32位的（也就是4字节），那么就是4字节存取，当然对于64位系统（现阶段大部分都是x86\_64），寄存大小虽然是64位，也按4字节存取**。所以平常使用的大部分处理器只能从地址为4的倍数的内存开始读取数据。

**如果没有内存对齐：**

一个int变量存放在从地址1开始的联系四个字节地址中，该处理器去取数据时，要先从0地址开始读取第一个4字节块,剔除不想要的字节（0地址）,然后从地址4开始读取下一个4字节块,同样剔除不要的数据**，**,最后留下的两块数据合并放入寄存器.这需要做很多工作.

### 内存对齐控制代码

每个特定平台上的编译器都有自己的默认“对齐系数”（也叫对齐模数）。**gcc**中默认`#pragma pack(4)`，可以通过预编译命令`#pragma pack(n)`，n = 1,2,4,8,16来改变这一系数。

查看内存对齐`#pragma pack(show)`

取消内存对齐`#pragma pack()`

### 内存对齐带来的问题:memcmp

在**比较类对象相等**的时候能不能用**memcmp？**

**不能**，有字节对齐的机制，里面是乱码

想用怎么办？**两个办法**：

一，把字节对齐的机制取消

二，把整个内存初始化一遍，全部清零，再赋值

所以一般**比较类对象相等**可以使用上面两种方法处理再用memcmp，或者**重载==运算符，把每个对象都比较一下**

### 内存对齐底层原理

{% embed url="https://zhuanlan.zhihu.com/p/83449008" %}

**内存对齐最最底层的原因是内存的IO是以8个字节64bit为单位进行的。** 对于**64位数据宽度的内存**，假如cpu也是64位的cpu（现在的计算机基本都是这样的），每次内存IO获取数据都是从同行同列的8个chip中各自读取一个字节拼起来的。从内存的0地址开始，0-7字节的数据可以一次IO读取出来，8-15字节的数据也可以一次读取出来。

指定要获取的是0x0001-0x0008，也是8字节，**但是不是0开头的，内存需要怎么工作呢？**没有好办法，内存只好先工作一次把0x0000-0x0007取出来，然后再把0x0008-0x0015取出来，把两次的结果都返回给你。 CPU和内存IO的硬件限制导致没办法一次跨在两个数据宽度中间进行IO。这样你的应用程序就会变慢



