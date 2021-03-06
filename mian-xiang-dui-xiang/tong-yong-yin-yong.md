# 通用引用

### T&&是叫通用引用

二重性：既可以是⼀个右值引⽤，也可以是⼀个左值引⽤

还可以绑定 到常量\(const\)和⾮常量\(non-const\)的对象上，也可以绑定到 volatile 和 non-volatile 的对象上，甚至可以绑定到即 const ⼜ volatile 的对象上，**几乎可以绑定到任何东西上**

空前灵活的引用值叫做通用引用\(universal references）也有的叫做转发引用\(forward references\)

* 如果⼀个函数模板参数的类型为 T&& ，并且 T 需要被推导得知，或者如果⼀个对象被声明为 auto&& ，这个参数或者对象就是⼀个通用引用。
* 通用引用，如果它被右值初始化，就会对应地成为右值引用;如果它被左值初始化，就会成为左值通用引用。

### 区分右值引用 与 通用引用

有具体类型的 Type&& 是右值引用

没有具体类型的 T&&, auto&&是通用引用

### **对右值引用使用std::move，对通用引用使用 std::forward**

### 当局部变量就是返回值是，不要使用 std::move 或者 std::forward

### 避免在通用引用上重载

完美转发构造函数 是糟糕的实现，因为对于 non-const 左值不会调用拷贝构造而是完美转发构造， 而且会劫持派⽣类对于基类的拷⻉和移动构造

通用引用重载的替代方法：包括使⽤不同的函数名，通过const左值引⽤传参，按值传递参数，使⽤tag dispatch

（详见effective modern c++, item26，item27）



