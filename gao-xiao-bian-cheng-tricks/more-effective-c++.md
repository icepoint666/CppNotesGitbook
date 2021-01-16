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

### 9.坚持将资源封装在对象内的原则（RAII的思想）

这样即使发生异常，局部对象在自动销毁的时候也可以调用其析构函数释放资源，避免泄漏

### 10.构造函数的内存安全——需要在 constructor 内阻止资源泄漏

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

### 11.**析构函数的异常安全——**禁止异常信息exceptions传递到析构函数destructor外

**两种情况**下会**调用析构函数**

* ①正常情况：删除一个对象，例如对象**超出了作用域**或被**显式地 delete**
* ②异常情况：异常传递的堆栈辗转开解（stack-unwinding）过程中，由**异常处理系统删除一个对象**。

遗憾的是**没有办法在析构函数内部区分出这两种情况，**因此在写析构函数时你**必须保守地假设有异常被激活**

**异常导致触发的析构，其内部再次抛出异常将会导致程序终止**

因为如果在**一个异常被激活的同时，析构函数同时也会抛出异常**，并导致程序控制权转移到析构函数外，**C++将调用 terminate 函数**，它会终止你程序的运行，**而且是立即终止，甚至连局部对象都没有被释放！**

**下面就是展示一下terminate触发的示例：**

函数 logCreation 和 logDestruction 被分别用于记录对象的建立与释放。我们因此可以这样编写 Session 的析构函数： 

```cpp
Session::~Session() 
{ 
 logDestruction(this); 
}
```

**如果 logDestruction 抛出一个异常**，异常没有被 Session 的析构函数捕获住，所以它被传递到析构函数的调用者那里。而正如前面所说的，如果**这次析构函数本身的调用就是源自于某些其它异常的抛出**，那么 terminate 函数将被自动调用，彻底**终止你的程序**

**解决：**你必须防止在 logDestruction 内抛出的异常传递到 Session 析构函数的外面。唯一的方法是用 try 和 catch blocks

```cpp
Session::~Session() { //一个错误版本
    try { 
        logDestruction(this); 
    } 
    catch (...) { 
        cerr << "Unable to log destruction of Session object " 
             << "at address " 
             << this 
             << ".\n"; 
    } 
}
```

但这样做并不比你原来的代码安全。**如果在 catch 中调用 operator&lt;&lt;时导致一个异常被抛出，我们就又遇到了老问题**

**终极解决**：因此**我们在释放 Session 时必须忽略掉所有它抛出的异常**

```cpp
Session::~Session() { 
   try { 
      logDestruction(this); 
   } 
   catch (...) { } 
}
```

\*\*\*\*

**因此禁止异常传递到析构函数外**有两个原因**：**

* **第一**能够在异常转递的堆栈辗转开解（stack-unwinding）的过程中，**防止 terminate 被调用**。
* **第二**它能帮助**确保析构函数总能完成我们希望它做的所有事情（也就是防止析构函数中途退出，从而很可能导致内存泄漏）**





## 效率

### 24.理解虚拟函数、多继承、虚基类和RTTI所需的代价

**在程序中**

**每个类只要声明了虚函数或继承了虚函数，它就有自己的 vtbl，**并且类中 vtbl 的item是指向虚函数实现体的指针

**每个这样的对象有自己的vptr，指向虚表**

![](../.gitbook/assets/wu-biao-ti-%20%2815%29.png)

**运行时类型识别（RTTI） 能让我们在运行时找到对象和类的有关信息**

所以肯定有某个地方存储了这些信 息让我们查询。这些信息被存储在类型为 type\_info 的对象里，你能通过使用 typeid 操作 符访问一个类的 type\_info 对象。

vtbl 数组的索引 0 处可以包含一个 type\_info 对象的指针，这个对象属于该 vtbl 相对应的类。上述 C1 类的 vtbl 看上去是这样：所以使用这种实现方法，**RTTI 耗费的空间是在每个类的 vtbl 中的占用的额外单元再加上存 储 type\_info 对象的空间**

![](../.gitbook/assets/wu-biao-ti-%20%2814%29.png)

下面这个表各是对虚函数、多继承、虚基类以及 RTTI 所需主要代价的总结： 

|  | Increases Size of Objects | Increases Per-Class Data | Reduces Inlining |
| :--- | :--- | :--- | :--- |
| Virtual Functions虚函数 | 是 | 是 | 是 |
| Multiple Inheritance多继承 | 是 | 是 | 不是 |
| Virtual Base Classes虚基类 | 经常 | 偶尔 | 不是 |
| RTTI | 不是 | 是 | 不是 |

### 27.要求或禁止在堆中产生对象

有时你想这样管理某些对象，要让某种类型的对象能够自我销毁,  而其它一些时候你想获得一种 保障：“不在堆中分配对象，从而保证某种类型的类不会发生内存泄漏

## 技术

### 34. 如何在同一个程序中结合 C 和 C++

四个要考虑的问题：名变换，静态初始化，内存动态分配，数据结构兼容。

**名变换：**

名变换就是 C++编译器给程序的每个函数换一个独一无二的名字

在 C 中，这个过程是不需要的，因为没有函数重载

如果是一个C库函数，编译后的函数仍然是原来的名字，没有产生名变换动作，但是C++编译器链接的时候，将会得到一个错误

**要解决这个问题，你需要一种方法来告诉 C++编译器不要在这个函数上进行名变换**

**所以C++中使用extern "C"的一个作用就是禁止名变换**

```cpp
extern "C" 
void drawLine(int x1, int y1, int x2, int y2);
```

经常，你有一堆函数不想进行名变换，为每一个函数添加 extern 'C'是痛苦的。幸好， 这没必要。extern 'C'可以对一组函数生效，只要将它们放入一对大括号中：

```cpp
extern "C" { // disable name mangling for 
 // all the following functions 
 void drawLine(int x1, int y1, int x2, int y2); 
 void twiddleBits(unsigned char bits); 
 void simulate(int iterations); 
 ... 
}
```

一般使用方法

```cpp
#ifdef __cplusplus 
extern "C" { 
#endif 
 void drawLine(int x1, int y1, int x2, int y2); 
 void twiddleBits(unsigned char bits); 
 void simulate(int iterations); 
 ... 
#ifdef __cplusplus 
} 
#endif
```

**静态初始化**

在 main 执行前和执行后都有大量代 码被执行。尤其是，静态的类对象和定义在全局的、命名空间中的或文件体中的类对象的构 造函数通常在 main 被执行前就被调用。这个过程称为静态初始化

```cpp
extern "C" // implement this 
int realMain(int argc, char *argv[]); // function in C 
int main(int argc, char *argv[]) // write this in C++ 
{ 
 return realMain(argc, argv); 
}
```

只要可能，用 C++写 main\(\)

**数据结构的兼容性**

在 C++和 C 之间传递数据

