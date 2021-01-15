# Modern Effective C++

### C++11新东西

* 类型推导，auto
* const\_iterator, nullptr, noexcept,constexpr等
* 智能指针
* 右值引⽤，移动语意，完美转发
* Lambda表达式
* 并发API

### 21.优先考虑使用std::make\_unique和 std::make\_shared而非new

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

**make函数的主要优点：异常安全**

\*\*\*\*

**不建议使用make函数的情况：**

包括需要指定⾃定义删除器和希望⽤⼤括号初始化 

对于 std::shared\_ptr  make函数可能不被建议的其他情况包括 \(1\)有⾃定义内存管理的类和 \(2\)特别关注内存的系统，⾮常⼤的对象，以及 std::weak\_ptr 比对应的 std::shared\_ptr 活得更久

