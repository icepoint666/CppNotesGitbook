# const

 修饰符**const**说明的类型，常类型的变量或对象的值是**不能被更新的，必须初始化**。

### 作用

1）避免被修改

2）助于编译器的常量优化，在程序运行中直接从内存加载到寄存器，在寄存器中使用，而不用每次使用都去从内存中读取

### const相对于\#define

const常量具有类型，编译器可以进行安全检查

\#define宏定义没有数据类型，只是简单的字符串替换，不能进行安全检查。

### const与extern的影响

**const对象默认为                         文件局部变量**

注意：非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。

```cpp
extern const int ext=12;
```

### const与指针

**四种const指针**

const char \* a  指向该const char对象的指针

char const \* a  同上！ 

char \* __const a 指针是const的，对象内容可以改变，但指针必须绑死了这个对象 __

const char \* const a  指向const对象的const指针，都不能变

分清**常量指针T const \***和**指针常量** **T \* const**

也不能使用void**\*** 指针保存const对象的地址，必须使用const void\* 类型的指针保存const对象的地址。

### const与引用

区别1：本质与普通引用有点不一样的：const引用保证不能通过引用改变原值

区别2：允许为常量引用，**绑定非常量的对象，字面值，甚至是表达式\(应该也需要是常量表达式）**

```cpp
int i = 42;
const int &r1 = i; 合法初始化 绑定对象
const int &r2 = 42; 合法！！绑定字面值
const int &r3 = r2 * 2; 合法！！绑定表达式
int &r4 = r2 * 2;  不合法！！！！
```

### const A &a 作为函数参数

参数为引用，为了增加效率同时防止修改。

```text
void func(const A &a)
```

对于非内部数据类型的参数而言，象void func\(A a\) 这样声明的函数注定效率比较低。因为函数体内将产生A 类型的临时对象用于复制参数a，而**临时对象的构造、复制、析构过程都将消耗时间**。

为了**提高效率**，可以将函数声明改为void func\(A &a\)，因为“引用传递”仅借用一下参数的别名而已，不需要产生临时对象。但是函数void func\(A &a\) 存在一个缺点：

**“引用传递”有可能改变参数a**，这是我们不期望的。解决这个问题很容易，加const修饰即可，因此函数最终成为void func\(const A &a\)。

是否应将void func\(int x\) 改写为void func\(const int &x\)，以便提高效率？完全没有必要，因为内部数据类型的参数不存在构造、析构的过程，而复制也非常快，“值传递”和“引用传递”的效率几乎相当。

### const类成员函数

任何**不会修改数据成员的函数都应该声明为const类型**。如果在编写const成员函数时，不慎修改数据成员，或者调用了其它非const成员函数，编译器将指出错误，这无疑会提高程序的健壮性。

```cpp
public:
    Apple(int i); 
    const int apple_number;
    void take(int num) const;
    int add(int num);
    int add(int num) const;
    int getCount() const;
```

**const成员函数可以 与 它对应的非const的版本 被重载区分**

### **const对象**

一个类对象被声明为const，就是const对象或者叫常对象，只有常成员函数才有资格操作常量或常对象，没有使用const关键字的成员函数不能用来操作常对象。

**const对象只能访问const成员函数**,而非const对象可以访问任意的成员函数,包括const成员函数.

