# RAII思想

RAII（**R**esource **A**cquisition **I**s **I**nitialization）资源获取即初始化，C++之父提出的

使用**局部对象**来管理**资源**的技术，叫做资源获取初始化，

**资源：**不单纯包括内存\(memory\)，还包括文件句柄\(file handles\)，网络套接字\(network sockets\)，数据库句柄\(database handles\) 总的来说就是有限的供应的东西

**作用域内资源管理 Scope-Bound Resource Management**：思想核心：变量的生命周期是限制于作用域内的，如果出作用域了destructor就会release它的资源



**RAII一个非常有用的性质就是，exception-safety 异常安全**

下面这个例子: 

如果不通过RAII思想实现，那么程序在捕获异常后，就会跳出这个作用域，然后delete语句就会忽视

从而造成内存泄漏：

```cpp
RawResourceHandle* handle=createNewResource();
handle->performInvalidOperation();  // Oops, throws exception
...
deleteResource(handle); // oh dear, never gets called so the resource leaks
```

RAII思想的实现例子：异常被捕获后，离开作用域，会自动调用destructor释放资源

```cpp
class ManagedResourceHandle {
public:
   ManagedResourceHandle(RawResourceHandle* rawHandle_) : rawHandle(rawHandle_) {};
   ~ManagedResourceHandle() {delete rawHandle; }
   ... // omitted operator*, etc
private:
   RawResourceHandle* rawHandle;
};

ManagedResourceHandle handle(createNewResource());
handle->performInvalidOperation();
```

**另外RAII在多线程编程中也经常使用**

**RAII的编程思想**就是：

* 封装一个资源到类中，类的构造函数通常会acquire它，然后析构函数会release它
* 使用资源：通过一个局部实例（类指针）`T obj = new T(...);`
* 当对象离开作用域后资源会被自动释放

但是，RAII远不是在析构函数里delete之类的小tip， RAII所谓的把资源绑定到对象生命周期**实质上意味着资源必须良好封装也就是类要低耦高聚（OO技术为了解释这个很玄的概念，有很多具体化的原则）**

