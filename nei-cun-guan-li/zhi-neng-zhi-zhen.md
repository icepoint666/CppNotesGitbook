# 智能指针

### 原始指针可能导致的问题（主要3个）

* 没办法区分指针是**对象还是数组**
* 要销毁的时候，没办法决定**是用delete还是析构函数**
* 不知道该指针到底有没有所指之物，因为没有引用计数，所以没有办法告诉你指针是否变成了**悬空指针（dangling pointers）**，即内存中不再存在指针 所指之物。悬空指针会在对象销毁后仍然指向它们

{% embed url="https://avdancedu.com/9683d88/" %}

* shared\_ptr 
* unique\_ptr
* weak\_ptr
* auto\_ptr（被 C++11 弃用）



### **shared\_ptr**

多个智能指针可以共享同一个对象，对象的最末一个拥有着有责任销毁对象，并清理与该对象相关的所有资源。\(类似于原本c指针的行为）

特性：

* 支持定制型删除器（custom deleter），可防范 Cross-DLL 问题（对象在动态链接库（DLL）中被 new 创建，却在另一个 DLL 内被 delete 销毁）、自动解除互斥锁
* **不是线程安全**

**shared\_ptr本质实现**

![](../.gitbook/assets/ss.png)

简化为两个部分：ptr与ref\_count的use\_count值

**`shared_ptr<Foo>x = new Foo();`**

![](../.gitbook/assets/28051716-748d3a6587fa498ca055ac6879dbb5f9.png)

**`shared_ptr<Foo>y = x;`**

![](../.gitbook/assets/28051717-7cfd55d9140043b19a9596b3f641965a.png)

**引用计数实现一些关键点，（**性能会稍微有所折扣）

* std::shared\_ptr ⼤小是原始指针的两倍，因为它内部包含⼀个指向资源的原始指针，还包含⼀ 个资源的引⽤计数值
* 存在一个很关键的实现问题：**引用计数动态分配的情况。** 理论上，**引⽤计数与所指对象关联起来，但是被指向的对象不知道这件事情**（译注：不知道有指向⾃⼰的指针）。因此它们**没有办法存放⼀个引⽤计数值**。不过使用 std::make\_shared 创建 std::shared\_ptr 可以避免引用计数的动态分配，但是还存在⼀些 std::make\_shared 不能使用的场景，这时候引用计数就会动态分配。
* **递增递减引用计数必须是原子性的**，因为多个reader、writer可能在不同的线程。⽐如，指向某种 资源的 std::shared\_ptr 可能在⼀个线程执⾏析构，在另⼀个不同的线程， std::shared\_ptr 指 向相同的对象，但是执⾏的确是拷⻉操作。原⼦操作通常⽐⾮原⼦操作要慢，所以即使是引⽤计 数，你也应该假定读写它们是存在开销的。

**std::shared\_ptr 使用delete作为资源的默认销毁器，但是 它也⽀持自定义的销毁器**。这种支持有别于 std::unique\_ptr 。对于 std::unique\_ptr 来说，销毁器 类型是智能指针类型的⼀部分。对于 std::shared\_ptr 则不是

std::shared\_ptr 为任意共享所有权的资源⼀种**自动垃圾回收的便捷⽅式**。 较之于 std::unique\_ptr ， std::shared\_ptr 对象通常大两倍，控制块会产生开销，需要原子引用计数修改操作。

**std::shared\_ptr的缺陷**

* 当你的资源由 std::shared\_ptr 管理，现在⼜想修改资源生命周期管理⽅式是没有办法的。 即使引⽤计数为⼀，你也不能重新修改资源所有权，改⽤ std::unique\_ptr 管理它。

  所有权和 std::shared\_ptr 指向的资源之前签订的协议是“**除非死亡否则永不分离**”。

* 避免从原始指针变量上创建 std::shared\_ptr，std::shared\_ptr 的**API设计之初就是针对单个对象的**，是不能处理数组这种指针的

### **weak\_ptr**

weak\_ptr 允许你共享但不拥有某对象，一旦最末一个拥有该对象的智能指针失去了所有权，任何 weak\_ptr 都会自动成空（empty）。因此，在 default 和 copy 构造函数之外，weak\_ptr 只提供 “接受一个 shared\_ptr” 的构造函数。



* 可打破环状引用（cycles of references，两个其实已经没有被使用的对象彼此互指，使之看似还在 “被使用” 的状态）的问题

### **unique\_ptr**

**核心特点：①只能通过移动操作来转移，没有拷贝操作 + ②unique\_ptr被销毁，资源对象也被销毁**

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

* 默认情况，资源销毁通过delete，但是⽀持⾃定义delete函数。有状态的删除器和函数指针会增加 std::unique\_ptr 的⼤小
* c++11中将 std::unique\_ptr 转化为 std::shared\_ptr 可以轻松高效完成

```cpp
template<typename... Ts> std::unique_ptr<Investment> makeInvestment(Ts&& params);

std::shared_ptr<Investment> sp = makeInvestment(arguments);
```

### **auto\_ptr**

被 c++11 弃用，原因是缺乏语言特性如 “针对构造和赋值” 的 `std::move` 语义，以及其他瑕疵。

**auto\_ptr 与 unique\_ptr 比较**

* auto\_ptr 可以赋值拷贝，复制拷贝后所有权转移；unqiue\_ptr 无拷贝赋值语义，但实现了`move` 语义；
* auto\_ptr 对象不能管理数组（析构调用 `delete`），unique\_ptr 可以管理数组（析构调用 `delete[]` ）；

