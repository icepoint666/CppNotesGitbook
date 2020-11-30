# More Effective C++

## 基础

### 1.仔细区别指针（pointer）和引用（reference）

* 指针可以指向 null，引用不允许指向 null。
* 指针可以重新被赋值，引用则不行，它总是指向最初获得的那个对象，所以引用初始化时必须有初值。

### 2.尽量使用C++风格的类型转换

使用新式转型符比较容易辨析，无论对于人还是工具而言。

* `static_cast` 基本拥有和 C 旧式转型相同的效果和威力，以及相同的限制。
* `const_cast` 用于改变表达式中的 **常量性（const）** 或 **易变性（volatile）**。
* `dynamic_cast` 用来执行继承体系中 “安全的向下转型或跨系转型动作”。
* `reinterpret_cast` 用来将一种类型重新解释为另一种类型，而不关心它们是否相关。

### 3.不要对数组使用多态

**示例：**

```cpp
class BST { ... }; 
class BalancedBST: public BST { ... };
BST BSTArray[10];
void printBSTArray(ostream& s, const BST array[], int numElements) 
//但是往里传入的是BalanceBST对象
BalancedBST bBSTArray[10];
printBSTArray(cout, bBSTArray, 10);
```

编译器将会毫无警告地编译这个函数

**但是会造成严重的后果，派生类的长度通常都比基类要长。我们料想 BalancedBST 对象长度的比 BST 长，没有人知道如果用 BalancedBST 数组 来执行 printBSTArray 函数将会发生什么样的后果。**

### **4.**避免无用的缺省构造函数（default constructor）

默认构造函数即没有参数的构造函数。添加无意义的默认构造函数显得画蛇添足，也会影响效率。

### 5.对定义的 “类型转换函数” 保持警觉

隐式类型转换操作符，是一个拥有奇怪名称的成员函数：关键字 operator 之后加上一个类型名称。如：

```cpp
class Rational {
public:
    // ...
    operator double() const; // 将 Rational 转换为 double
};
```

然而它们的出现可能导致错误（非预期）的函数被调用。为了避免发生这种情况，**解决办法是以功能对等的另一个函数取代类型转换操作符**，例如使用一个名为 `asDouble` 的函数取代它。

单自变量 constructor 是指能够以单一自变量成功调用的构造函数。我们可以**将构造方法声明为 explicit** 避免发生隐式转换带来的非预期错误。

### 6.区分 `++`，`--` 操作符的前置形式和后置形式，前置式效率更高

 `++` 或 `--` 操作符前置式和后置式的重载函数都没有参数。为了填平这个语法上的漏洞，

只好让后置式有一个 int 自变量，并且在在它被调用时，编译器默默地为该 int 指定一个 0 值。

**前置递增运算符通常如下：**

```cpp
Date& operator ++ () {
    // increment
    return *this; // 后取出
}
```

**后置递增运算符的返回值类型不同，且有一个输入参数，通常如下：**

```cpp
const Date operator ++ (int) {
    Date copy (*this); // 先取出
    // increment
    return copy; // 返回先前取出的值
}
```

**返回 const 类型是为了禁止 Date++++ 这样的错误操作。**

**单从效率来看，前置式比后置式要好。**

### 7. 千万不要重载 `&&`、`||` 和 `,` 操作符

对于短路与（&&）、短路非（\|\|）和逗号运算符（,）不管你多么努力，都无法令其行为像它们应有的那样，所以千万不要重载它们。

### **8.区分new, operator new，placement new，new\[\]**

**`new = operator new + placement new`**

**new operator：**

如果希望对象产生于 heap，请使用 **new operator**。它**不但分配内存**还会**调用一个构造函数**进行初始化。

```cpp
string *p = new string("Hello");
```

**operator new**

如果**只打算分配内存**，请使用 **operator new**。它不会调用任何构造函数，你也可以写一个自己的 operator new。

```cpp
void *p = operator new(sizeof(string));
```

**placement new**

如果你打算**在指定的内存位置构造对象**（需要先分配），请使用 **placement new**。这常用于 shared memory 或 memory-mapped I/O。

```cpp
#include <new>
void* operator new(size_t, void* location) {
    return location;
}
```

**new\[\] 新建数组**

delete 会先调用析构函数，再执行 operator delete 释放内存。  
delete\[\] 会为数组中的每个元素调用析构函数，再执行 operator delete\[\] 释放内存。

## 异常

### 9.坚持一个原则，将资源封装在对象内（RAII的思想）

这样即使发生异常，局部对象在自动销毁的时候也可以调用其析构函数释放资源，避免泄漏

### 10.需要在 constructor 内阻止资源泄漏

**注意：** C++ 只会析构 **已构造完成** 的对象。也就是说在构造函数中发生异常的话，析构函数是不会执行的。

**9，10里的思想改进对比**

**原本**

```cpp
void processAdoptions(istream& dataSource) 
{ 
    while (dataSource) { // 还有数据时,继续循环 
        ALA *pa = readALA(dataSource); //得到下一个动物 
        pa->processAdoption(); //处理收容动物 
        delete pa; //删除 readALA 返回的对象 
    } 
}
```

**异常安全**

```cpp
void processAdoptions(istream& dataSource) 
{ 
     while (dataSource) { 
         ALA *pa = readALA(dataSource); 
         try { 
             pa->processAdoption(); 
         } 
         catch (...) { // 捕获所有异常 
            delete pa; // 避免内存泄漏 
            // 当异常抛出时 
            throw; // 传送异常给调用者 
         } 
         delete pa; // 避免资源泄漏 
     } // 当没有异常抛出时 
}
```

构造资源时用**auto\_ptr**代替**new指针**

```cpp
template<class T> 
class auto_ptr { 
public: 
    auto_ptr(T *p = 0): ptr(p) {} // 保存 ptr，指向对象 
    ~auto_ptr() { delete ptr; } // 删除 ptr 指向的对象 
private: 
    T *ptr; // raw ptr to object 
};
```

```cpp
void processAdoptions(istream& dataSource) 
{ 
    while (dataSource) { 
        auto_ptr<ALA> pa(readALA(dataSource)); 
        pa->processAdoption(); 
    } 
}
```

C++11后普遍用**unique\_ptr代替auto\_ptr（因为auto\_ptr允许copy，就会存在问题,unique\_ptr禁用了copy，主要通过move来完成）**

### 11. 禁止异常流出 destructor 之外

有两个好处：

* 第一是它可以避免 terminate 函数在异常传播过程的栈展开机制中被调用，你的程序将被立即结束。
* 第二是它可以协助确保 destructor 完成其应该完成的所有事情。

#### 12. 了解 “抛出一个异常” 与 “传递一个参数” 或 “调用一个虚函数” 之间的差异

“抛出一个异常”，异常对象总是会被复制一次，如果以 by value 方式捕捉，则会发生两次复制。

“传递一个参数”，如果是 by reference 方式则不会发生复制，如果是 by value 方式则发生复制。（简单的说就是抛出异常会比传递参数多发生一次复制）

抛出的异常不会发生隐式转型（即 int 类型不会默默转为 double 类型而被捕获）。只有两种转换可以发生，一种是继承关系中的向上转型，另一种是 “有型指针” 转为 “无型指针”。`const void*` 指针可捕捉任何指针类型的异常。

异常的捕捉遵循 “最先吻合” 策略，即找到第一个匹配者执行。调用虚函数则是 “最佳吻合” 策略，即执行的是与对象类型最吻合的函数。

## 效率

## 技术

### 34. 如何在同一个程序中结合 C 和 C++





