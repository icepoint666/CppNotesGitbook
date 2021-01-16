# Effective Modern C++

### C++11新东西

* 类型推导，auto
* const\_iterator, nullptr, noexcept,constexpr等
* 智能指针
* 右值引⽤，移动语意，完美转发
* Lambda表达式
* 并发API

### 7.区别使用\(\)和{}创建对象

**初始化对象的一些方法（4种）：**

```cpp
int x(0); //使⽤小括号初始化 
int y = 0; //使⽤"="初始化 
int z{0}; //使⽤花括号初始化
int z = {0}; //使⽤"="和花括号
```

一些初始化方法构造的实现原理

```cpp
Widget w1; //调⽤默认构造函数 
Widget w2 = w1; //调⽤拷⻉构造函数（不是赋值运算符）
w1 = w2; //调⽤operator=函数（是⼀个赋值运算符）
```

**"="初始化，括号初始化**能被用于为**非静态数据成员指定默认初始值**

```cpp
class Widget{
    ... 
private: 
    int x{0}; //没问题，x初始值为0 
    int y = 0; //同上 
    int z(0); //错误！ 
}
```

**不可拷贝的对象**可以使用花括号初始化或者小括号初始化，但是不能使用"="初始化：

```cpp
std::vector<int> ai1{0}; //没问题，x初始值为0 
std::atomic<int> ai2(0); //没问题 
std::atomic<int> ai3 = 0; //错误！
```

**括号初始化的缺点**是有时它有⼀些令人惊讶的行为。 这些行为使得括号初始化和std::initializer\_list和构造函数重载决议本来就不清不楚的暧昧关系进⼀步混乱。

在构造函数调⽤中，只要不包含std::initializer\_list参数，那么花括号初始化和小括号初始化都会产⽣⼀ 样的结果：

```cpp
class Widget { 
public: 
    Widget(int i, bool b); //未声明默认构造函数 
    Widget(int i, double d); // std::initializer_list参数 
    … 
};
Widget w1(10, true); // 调⽤构造函数 
Widget w2{10, true}; // 同上 
Widget w3(10, 5.0); // 调⽤第⼆个构造函数 
Widget w4{10, 5.0}; // 同上
```

### 8.优先考虑nullptr而非0和NULL

* 字面值0是⼀个int不是指针
* NULL也是这样的：NULL的实现细节有些不确定因素， 因为实现被允许给NULL⼀个除了int之外的整型类型（⽐如long）

nullptr的优点是它不是整型。 ⽼实说它也不是⼀个指针类型，但是你可以把它认为是通⽤类型的指针。 nullptr的真正类型是std::nullptr\_t

### 10.优先考虑限域枚举而非未限域枚举

**未限域枚举**

```cpp
enum Color { black, white, red }; // black, white, red 和 Color⼀样都在相同作⽤域 
auto white = false; // 错误! white早已在这个作⽤域中存在
```

**限域枚举**是通过enum class声明，所以它们有时候也被称为枚举类\(enum classes\)

```cpp
enum class Color { black, white, red }; // black, white, red 限制在Color域内 
auto white = false; // 没问题，同样域内没有这个名字
Color c = white; //错误，这个域中没有white
Color c = Color::white; // 没问题
auto c = Color::white; // 也没问题
```

### 11.优先考虑使用deleted函数而非使用未定义的私有声明

如果你写的代码要被其他⼈使⽤，你不想让他们调⽤某个特殊的函数，你通常不会声明这个函数

但有时C++会给你⾃动声明⼀些函数，如果你想防⽌客⼾调⽤这些函数，就需要delete

```cpp
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
    …
    basic_ios(const basic_ios& ) = delete;
    basic_ios& operator=(const basic_ios&) = delete;
    …
};
```

### 12.使用override声明重载函数

重写 与 重载 需要区分：虚函数是叫重写，其他是重载

C++11提供⼀个⽅法让你可以显式的将派⽣类 函数指定为应该是基类重写版本：将它声明为 override

**示例：**

```cpp

```

### 13.优先考虑const\_iterator而非iterator

STL const\_iterator等价于指向常量的指针。它们都指向不能被修改的值。**标准实践是能加上const就加上，这也指示我们对待const\_iterator应该如出一辙。**

**在最大程度通用的代码中，优先考虑非成员函数版本的begin，end，rbegin等，而非同名成员函数**

* **标准化c++11只添加了**非成员函数**begin**和**end**
* **c++14**提供了**cbegin，cend，rbegin，rend，crbegin，crend** 这些函数，其中**c开头的就是const iterator**
* 非成员函数cbegin的实现：

  这个cbegin模板接受任何容器或者类似容器的数据结构 C ，并且通过 const 引⽤访问第⼀个实参container。如果 C 是⼀个 普通的容器类型（如 std::vector ），container将会引⽤⼀个常量版本的容器（即 const std::vector& ）。对const容器调用非成员函数begin（由C++11提供\)将产出const\_iterator

```cpp
template <class C>
auto cbegin(const C& container)->decltype(std::begin(container)){
    return std::begin(container); // 解释⻅下
}
```



### 14.如果函数不抛出异常请使用noexcept

函数是否为noexcept和成员函数是否const⼀样重要。如果知道这个函数不会抛异常就 加上noexcept是简单天真的接口说明

**给不抛异常的函数加上noexcept的动机：它允许编译器生成更好的目标代码**

考虑⼀个函数f，它允 许调⽤者永远不会受到⼀个异常。两种表达方式如下：

```cpp
int f(int x) throw(); // C++98⻛格  较少优化
int f(int x) noexcept; // C++11⻛格  极尽所能优化
int f(int x);         //不优化
```

* noexcept是函数接口的⼀部分，这意味着调⽤者会依赖它
* noexcept函数较之于非noexcept函数更容易优化
* noexcept对于**移动语义,swap，内存释放函数和析构函数**非常有用
* 大多数函数是异常中⽴的\(译注：可能抛也可能不抛异常）而不是noexcept

### 15.尽可能的使用constexpr

* constexpr对象是cosnt，它的值在编译期可知
* 当传递编译期可知的值时，constexpr函数可以产出编译期可知的结果

### 16.让const成员函数线程安全

需要注意的是：确保**const成员函数**线程安全，除非你确定它们永远不会在**临界区（concurrent context）**中使用

**const 成员函数，一般就表示着它是⼀个读操作，但是有时候不是线程安全的**

最普遍简单的⽅法就是-------使⽤互斥锁：

```cpp
class Polynomial {
public: 
    using RootsType = std::vector<double>;
    RootsType roots() const {
        std::lock_guard<std::mutex> g(m); // lock mutex 
        if (!rootsAreVaild) { // 如果缓存⽆效 计算/存储roots
            rootsAreVaild = true;
        }
        return rootsVals;
    } // unlock mutex 
private:
    mutable std::mutex m;
    mutable bool rootsAreVaild { false };
    mutable RootsType rootsVals {}; 
};
```

注意：std::atomic 可能⽐互斥锁提供更好的性能，但是它只适合操作单个变量或内存位置

### 17.注意特殊成员函数的生成

**特殊成员函数**是编译器可能⾃动⽣成的函数（6个）：

* 默认构造函数
* 析构函数
* 拷贝构造函数
* 拷贝赋值运算符
* 移动构造函数
* 移动赋值运算符

**移动操作（移动构造函数+移动赋值运算符函数）**一般不会自动生成，在当**类没有没有显式声明**移动操作，拷⻉操作，析构时才⾃动⽣成

**拷贝构造函数** 仅当类没有显式声明构造构造时才⾃动⽣成，并且如果用户声明了移动操作，拷贝构造就是delete

**拷贝赋值运算符**仅当类没有显式声明拷贝赋值运算符时才⾃动⽣成，并且如果用户声明 了移动操作，拷贝赋值运算符就是delete

当用户声明了**析构函数**，**拷贝操作不再自动生成**

### 21.优先考虑使用std::make\_unique和 std::make\_shared而非new

**核心：防止异常安全 + 内存分配更高效**

* std::make\_shared c++11 加入标准库
* std::make\_unique c++14 加入标准库
  * make\_unique 只是将它的参数完美转发到所要创建的对象的构造函数，从新产⽣的原始 指针⾥⾯构造出 std::unique\_ptr ，并返回这个 std::unique\_ptr

```cpp
template<typename T, typename... Ts> std::unique_ptr<T> make_unique(Ts&&... params) { 
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...)); 
}
```

make与new两种写法

```cpp
auto upw1(std::make_unique<Widget>()); // with make func 
std::unique_ptr<Widget> upw2(new Widget); // without make func 
auto spw1(std::make_shared<Widget>()); // with make func 
std::shared_ptr<Widget> spw2(new Widget); // without make func
```

**make函数的优点①：异常安全**

经典异常安全隐患的问题

```cpp
processWidget(std::shared_ptr<Widget>(new Widget), computePriority());
```

相当于有三个操作，new Widget，std::shared\_ptr 的构造函数，compute Priority

编译器优化的特点，执行顺序完全有可能是这样：

* 1.执⾏new Widget
* 2.执行computePriority
* 3.运行std::shared\_ptr 构造函数

在运⾏是computePriority产生了异常，那么第⼀步动态分配的Widget就会泄露。因为它永远都不会被第三步的 std::shared\_ptr 所管理了

解决：

```cpp
processWidget(std::make_shared<Widget>(), computePriority());
```

**make函数的优点②：相对于直接用new，make\_shared、make\_unique内存分配上更高效**

下面这段代码需要内存分配，但是实际上执行了两次

```cpp
std::shared_ptr<Widget> spw(new Widget);//这段代码实际上执行了两次内存分配
auto spw = std::make_shared_ptr<Widget>(); //一次分配就够了
```

这是因为 std::make\_shared 分配⼀块内存，同时容纳了Widget对象和控制块。这种优化减少了程 序的静态⼤小，因为代码只包含⼀个内存分配调⽤，并且它提⾼了可执⾏代码的速度，因为内存只分配 ⼀次。

**不建议使用make函数的情况：**

包括需要指定⾃定义删除器和希望⽤⼤括号初始化 

对于 std::shared\_ptr  make函数可能不被建议的其他情况包括 \(1\)有⾃定义内存管理的类和 \(2\)特别关注内存的系统，⾮常⼤的对象，以及 std::weak\_ptr 比对应的 std::shared\_ptr 活得更久

### 41.如果参数可拷贝并且移动操作开销很低，总是考虑 直接按值传递

对于可复制，移动开销低，而且⽆条件复制的参数，按值传递效率基本与按引⽤传递效率⼀致，而且易于实现，⽣成更少的⽬标代码

通过构造函数拷⻉参数可能⽐通过赋值拷⻉开销⼤的多

### 42.emplace\_back比push\_back效率更高，因为是依靠完美转发实现，而不是构造拷贝

示例：

```cpp
std::vector<std::string> vs; // container of std::string 
vs.push_back("xyzzy"); // add string literal
```

push\_back的本质：

std::vector 的 push\_back 被按左值和右值分别重载：

在 vs.push\_back\("xyzzy"\) 这个调⽤中，编译器看到参数类型（const char\[6\]）和 push\_back 采⽤的 参数类型（ std::string 的引⽤）之间不匹配。它们通过从字符串字⾯量创建⼀个 std::string 类型 的临时变量来消除不匹配，然后传递临时变量给 push\_back，**本质上上述过程等于下面的代码：**

```cpp
vs.push_back(std::string("xyzzy")); 
// create temp std::string and pass it to push_back
```

**并不仅调用了 一次构造器，调用了两次，而且还调用了析构器**

**解决：**不使⽤ push\_back ，使用的是 emplace\_back

**emplace\_back 使用完美转发**，因此只要你**没有遇到完美转发的限制（完美转发里提到的fail case）**，就可以传递任何 参数以及组合到 emplace\_back

* 当然就像预期的那样，emplacement执⾏性能优于insertion，但是，有些场景反而 insertion更快
* 这些情况不是很容易描述，比较依赖于传递的参数类型、容器类型、emplacement或 insertion的容器位置、

总的来说，当执⾏如下操作时，emplacement函数更快 

* 1. 值被构造到容器中，而不是直接赋值
* 2. 传⼊的类型与容器类型不⼀致 （例如刚刚的例子：const char\[\] 与 std::string）
* 3. 容器不拒绝已经存在的重复值

