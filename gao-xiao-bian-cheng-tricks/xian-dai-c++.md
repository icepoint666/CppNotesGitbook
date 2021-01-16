# Effective Modern C++

### C++11新东西

* 类型推导，auto
* const\_iterator, nullptr, noexcept,constexpr等
* 智能指针
* 右值引⽤，移动语意，完美转发
* Lambda表达式
* 并发API

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

