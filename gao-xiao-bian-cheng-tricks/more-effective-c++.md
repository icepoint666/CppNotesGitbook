# More Effective C++

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

### 异常



