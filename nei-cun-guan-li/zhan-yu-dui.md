# 栈与堆

**栈**：栈指的是函数过程的调用栈 \(call stack\)，在 x86 机器上，栈由操作系统分配，并向低地址方向增长。**栈的大小由编译器/链接器指定**，**信息保存在可执行文件**中 \(executable file\)。

**堆：** 是一个由操作系统或 CRT \(C Runtime Library\) 运行时库管理的内存分配空间，有**特定的内部数据结构**，**及其分配策略算法**——解决内存的（重新）分配、回收与碎片问题。堆提供一组 API 给用户

**实际应用场景到底使用堆还是栈？**

### 局部变量

它的存储在栈中分配，如果它是一个 array, struct, union, class 统称复合类型，那么它的每个成员都在栈中分配。

* size 是编译时确定的，所以在栈中构造，编译时即可预留空间（通过减小 esp/rsp 寄存器的值）
* 内存分配的**效率要比在堆中高**，因为堆要在运行时执行分配算法

注意：但是复合类型可能内含 `std::string` 成员，或者 `char* p` 成员，这些会在构造函数中，p = new char\[BUF\_SIZE\]，这些**即使在栈中构造对象，其实还是会对其成员执行堆分配算法，编译器确定的只是静态部分的 size**

### 静态成员

静态变量（包括全局变量、编译单元级的 static、名字空间和类内的 static 和局部 static）。静态变量在编译时就预留了空间（信息放在目标和可执行文件中）

同样：和局部变量类似，要看它们内部成员有没有动态分配的。如果非局部的静态对象内部有动态分配的成员，CRT 会在进入 main\(\) 函数前执行堆分配算法。因为静态对象的生存期是到程序结束，所以它们**内部的动态分配成员需要手工释放**（如用 copy-and-swap 惯用法释放 `std::vector` 或 `std::basic_string`），以避免内存泄露检测例程给出误判

### 栈大小是固定的，而且很小

由编译器/链接器决定的栈的大小是固定的，对于 MSVC 默认是 1MB，称为 reserved size of stack allocation in virtual memory

相比于堆，栈的大小很小，可能才1MB

如果你利用栈实现一些算法——典型地是递归算法，例如递归下降分析器 \(recursive descent parser\) 或深度优先搜索 \(depth-first search\)，当问题规模很大时就会**栈溢出 \(stack overflow\)**

**自定义栈**：将递归算法变换为非递归算法，考验程序员功力。简单问题容易用循环替代，复杂问题呢**用自定义的栈结构代替调用栈，而在堆中分配自定义的栈结构（比如用 `std::deque`）**。

### STL动态容器都是分配的动态内存（堆）

C++ 中的 `operator new/new[]`、`std::vector` 等，其实都是调用接口（一种约定的形式），**它们实际上是在堆、栈，还是特殊的内存空间中分配存储，都是可以自定义的**。`std::shared_ptr`是比 `operator new` 更良好的形式，而 `std::vector` 或 `std::basic_string`（当元素是 char/wchar\_t 时）是比 `operator new[]`更良好的形式。

一个例子：vector存放string，这些string是在连续的地址空间吗？

这个不一定，因为如果string内容超过SSO长度，就要分配给另一个堆空间了，肯定保证不了内存连续存放

### 特殊的堆内存分配

即使要分配的是普通的堆内存，**为了特殊的效率需求，有时也要用自定义分配算法**，代替默认的系统或 CRT 堆算法。

**分配池：**最简单的情况是，无需写 allocator，在某个过程初始化时，预分配足够的堆内存（如调用 `std::vector::reserve()`），然后应用逻辑运行时，只是赋值、操作这些已分配内存。这种思想叫做分配池 \(pooling\)。

### 面试题：如何定义一个只能在堆上（栈上）生成对象的类？

**定义一个只能在堆上生成的类**

方法：将**析构函数设置为私有**

原因：C++ 是静态绑定语言，编译器管理栈上对象的生命周期，编译器在为类对象分配栈空间时，会先检查类的析构函数的访问性。若析构函数不可访问，则不能在栈上创建对象。

**定义一个只能在栈上生成的类\(禁止堆对象）**

方法：将 **new 和 delete 重载为私有**

```cpp
class UPNumber { 
private: 
 static void *operator new(size_t size); 
 static void operator delete(void *ptr); 
 ... 
};
```

原因：在堆上生成对象，使用 new 关键词操作，其过程分为两阶段：第一阶段，使用 new 在堆上寻找可用内存，分配给对象；第二阶段，调用构造函数生成对象。将 new 操作设置为私有，那么第一阶段就无法完成，就不能够在堆上生成对象。

**判断对象是否在堆，栈，静态变量区域**

不仅没有一种可移植的方法来判断对象是否在堆上，而且连能在多数时间 正常工作的“准可移植”的方法也没有。如果你实在非得必须判断一个地址是否在堆上，你 必须使用完全不可移植的方法，其实现依赖于系统调用，只能这样做了。因此你最好重新设计你的软件，以便你可以不需要判断对象是否在堆中。

