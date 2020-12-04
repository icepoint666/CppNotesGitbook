# 智能指针

{% embed url="https://avdancedu.com/9683d88/" %}

* shared\_ptr 
* unique\_ptr
* weak\_ptr
* auto\_ptr（被 C++11 弃用）



### **shared\_ptr**

多个智能指针可以共享同一个对象，对象的最末一个拥有着有责任销毁对象，并清理与该对象相关的所有资源。\(类似于原本c指针的行为）

* 支持定制型删除器（custom deleter），可防范 Cross-DLL 问题（对象在动态链接库（DLL）中被 new 创建，却在另一个 DLL 内被 delete 销毁）、自动解除互斥锁
* **不是线程安全**

**shared\_ptr本质实现**

![](../.gitbook/assets/ss.png)

简化为两个部分：ptr与ref\_count的use\_count值

**`shared_ptr<Foo>x = new Foo();`**

![](../.gitbook/assets/28051716-748d3a6587fa498ca055ac6879dbb5f9.png)

**`shared_ptr<Foo>y = x;`**

![](../.gitbook/assets/28051717-7cfd55d9140043b19a9596b3f641965a.png)

### **weak\_ptr**

weak\_ptr 允许你共享但不拥有某对象，一旦最末一个拥有该对象的智能指针失去了所有权，任何 weak\_ptr 都会自动成空（empty）。因此，在 default 和 copy 构造函数之外，weak\_ptr 只提供 “接受一个 shared\_ptr” 的构造函数。

* 可打破环状引用（cycles of references，两个其实已经没有被使用的对象彼此互指，使之看似还在 “被使用” 的状态）的问题

### **unique\_ptr**

unique\_ptr 是 C++11 才开始提供的类型，是一种在异常时可以帮助避免资源泄漏的智能指针。采用独占式拥有，意味着可以确保一个对象和其相应的资源同一时间只被一个 pointer 拥有。一旦拥有着被销毁或编程 empty，或开始拥有另一个对象，先前拥有的那个对象就会被销毁，其任何相应资源亦会被释放。

* unique\_ptr 用于取代 auto\_ptr
* **unique\_ptr因为无法同时拥有两个引用，所以转移指针需要`std::move`来完成**

**unique\_ptr初始化（不能直接new T\(\)返回,  C++11需要自定义一个std::make\_unique函数\)**

```cpp
#if __cplusplus >= 201402L //c++14
    #include <utility>
    using namespace std::make_unique;
#else                      //c++11
    template<class T, class... Args>
    std::unique_ptr<T> make_unique(Args&&... args){
        return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
    }
#endif
//使用
std::unique_ptr<Epoller> p = make_unique<T>(args);
```

### **auto\_ptr**

被 c++11 弃用，原因是缺乏语言特性如 “针对构造和赋值” 的 `std::move` 语义，以及其他瑕疵。

**auto\_ptr 与 unique\_ptr 比较**

* auto\_ptr 可以赋值拷贝，复制拷贝后所有权转移；unqiue\_ptr 无拷贝赋值语义，但实现了`move` 语义；
* auto\_ptr 对象不能管理数组（析构调用 `delete`），unique\_ptr 可以管理数组（析构调用 `delete[]` ）；

