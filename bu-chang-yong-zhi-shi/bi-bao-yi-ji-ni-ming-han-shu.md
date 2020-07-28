---
description: 函数式编程相关
---

# 闭包以及匿名函数

### 什么是闭包

闭包是带有上下文的函数。说白了，就是有状态的函数

一个函数, 带上了一个状态, 就变成了闭包了。什么叫 "带上状态" 呢? 意思是这个闭包有属于自己的变量，**这些个变量的值是创建闭包的时候设置的**，**并在调用闭包的时候，可以访问这些变量**

函数是代码，状态是一组变量，将代码和一组变量捆绑，就形成了闭包，内部包含 static 变量的函数不是闭包，因为这个 static 变量不能捆绑。**闭包的状态捆绑，必须发生在运行时。**

**其实就是lambda函数的显示捕获**

### **匿名函数lambda**

```cpp
auto func1 = [](int i) {return i + 4; };
std::cout << "func1: " << func1(4) << '\n';
```

**隐式捕获**  
为了指示编译器推断捕获列表，我们可以在捕获列表中写一个 `&` 或 `=`。`&` 告诉编译器采用引用捕获，`=` 则为值捕获。

```cpp
int round = 2;
auto f = [=](int f) -> int { return f + round; } ;
cout << "result = " << f(1) << endl;
```

**lambda在C++14允许设置默认参数**

```cpp
// since C++14, lambda could own default arguments
auto func1 = [](int i = 6) { return i + 4; };
std::cout << "func1: " << func1() << '\n';
```

### **std::bind 强大的语法糖**

\#include &lt;functional&gt;中定义

* **std::function**
* **std::bind**
* **std::placeholders**

**std::placeholders占位符，专门配合用于std::bind，来占位表示bind的新函数需要传入的参数：**

**`_1`表示调用时第一个参数，`_2`就同样表示调用时第二个参数**

```cpp
#include <functional>

int func(int tmp, int round) {
    return tmp + round;
}

using namespace std::placeholders;    // adds visibility of _1, _2, _3,...

int round = 2;
std::function<int(int)> f = std::bind(func, _1, round);
cout << "result = " << f(1) << endl;
```

\*\*\*\*

\*\*\*\*

