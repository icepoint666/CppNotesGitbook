# C++语法的复杂度

### IDE的自动导入功能

思考一下很多python或者java的IDE都有自动import功能，输入一个什么东西（函数），就会提示，然后鼠标点击就会自动把刚刚用的东西的Import给自动写到代码前面去

这里主要讨论一下C的复杂性对IDE自动导入的诸多不友好之处

### 宏的复杂性

问题放在 C/C++ 里就完全不一样了。**C/C++ 里有宏，而宏会决定头文件里实际生效的代码**。

如果是 IDE 比较好掌握的条件宏，比如 C++ 里通过 -std= 选项控制标注版本的 \_\_cplusplus 宏，问题还比较容易处理

但是如果是**用户指定的宏**，在用户-D参数的选项中决定实际行为的，那么IDE就很判断出来

对于有些 IDE，比如 vs 可以从它的解决方案的属性里，又比如 CLion 可以从 CMakeLists.txt 里提前掌握到编译时会传哪些宏‘

假如我要用 \_\_**TIME\_\_** 宏（一个定义编译时间的宏）搞事情呢？如果项目是早上编译的那我启用某段代码，如果是下午编译的我启用另一段，那你怎么搞？就很难解决了，可能写起来比编译器还要复杂

### 模板编程特化的情况

> 如果IDE想要自动include

```cpp
class MyIntAllocator;

namespace std
{
    template <typename T, typename Allocator>
    class vector;
}

extern std::vector<int, MyIntAllocator> v1, v2;

void f()
{
    std::swap(v1, v2);
}
```

C++ 的都知道 std::swap\(T&, T&\) 是&lt;utility&gt;中提供的模板函数。那对于上一段代码，应该提示缺少 &lt;utility&gt;头文件吗？

但是&lt;vector&gt;中有针对 std::vector 的 swap 特化啊

根据模板中的最佳匹配原则，应该优先适用特化的版本。如果只 include 了 std::swap\(T&, T&\) 所在的文件，致使编译器在编译时没“看到”特化版本，而只“看到”并用通用的 std::swap\(T&, T&\) 版本去编译，那这就是程序bug，并且这种Bug很难检查出来

### 命名空间同名的情况

不同命名空间可能有同名的东西。比如你写个 vector

那IDE该 include 标准库的&lt;vector&gt;呢？还是 &lt;boost/container/vector.hpp&gt; 呢？肯定还是需要用户主观意见才可以

### 未来发展

实际上，自动 include CLion 就有做，但是我的感觉是还是不太好用，因为经常导入错的头文件。另外就是做分析时太耗资源了，经常出现 12 核的 CPU 跑满。。。

C++20 带来了 module。或许以后，在 C++ 里也将抛弃头文件，改用 import 来导入了。module 与头文件最大的不同就是对宏的使用有一定的限制（比如宏只影响本模块内部，而不会影响到其他模块）。这对解决我开头说的宏给自动 import 带来的难点有些帮助。但，因 C++ 的复杂语法规则给自动 import 带来的难处，仍很难解决。  




