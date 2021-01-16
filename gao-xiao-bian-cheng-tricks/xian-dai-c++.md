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

