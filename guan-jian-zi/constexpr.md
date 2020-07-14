# constexpr

两个作用

* 1.拓宽「常量表达式」的范围，可以验证变量值是否是「常量表达式」
* 2.提供显式的方法，去要求表达式**编译时求值**

元编程中经常使用constexpr

## 拓宽「常量表达式」的范围

主要就是希望一些明显很简单的函数，使它的调用也能是常量表达式

**出发点：把「对某些简单函数的调用」加入「常量表达式」定义**

示例：简化元函数

```cpp
template<class U, class V>
constexpr bool is_same_v = std::is_same<U, V>::value;
```

调用的时候可以直接

```cpp
auto x = is_same_v<int, float>;
```

**常量表达式 优势**：编译器处理的时候，可以进行一个**常量折叠**\(constant folding\) 的优化

示例：对于下面的表达式，

```c
1 + 2 + 5
```

在做语法分析+构建AST的时候就可以将其常量折叠为8了

具体关于常量的优化参考：[https://www.zhihu.com/question/55976094/answer/147302764](https://www.zhihu.com/question/55976094/answer/147302764)

**验证变量值是否是一个常量表达式**：如果不是就会报错

```cpp
constexpr int mf =20; //是常量表达式
constexpr int mf = func(argv[2]); //不是常量表达式，会报错
```

## 提供显式的方法，去要求表达式编译时求值

说到这里，**「constexpr 函数」并不能和「编译时求值」划等号**，它只表达了「这个函数具备这样的能力」。

也就是说**constexpr函数**只有同时满足

1. 所有参数都是常量表达式
2. 返回的结果都被用于常量表达式（比如用于初始化constexpr的数据）

才会触发编译时求值

所以constexpr修饰的函数，返回值不一定是编译期常量

**关键理解示例：**

```cpp
#include <iostream>
#include <array>
using namespace std;

constexpr int foo(int i)
{
    return i + 5;
}

int main()
{
    int i = 10;
    std::array<int, foo(5)> arr; // OK！声明类型需要提前编译时求值，因为满足上述编译时求值的条件，所以不会报错

    foo(i); // Call is Ok 调用也是不会报错，就是运行时求值

    // But...
    std::array<int, foo(i)> arr1; // Wrong! 因为不满足条件，无法编译时求值，但是又要在编译时确定声明类型，所以报错

}
```

有两种可能：

* 如果其传入的参数可以在编译时期计算出来，那么这个函数就会产生编译时期的值。
* 如果其传入的参数不能在编译时期计算出来，那么constexpr修饰的函数就和普通函数一样了

const与constexpr的区别：

const并未区分出编译期常量和运行期常量 constexpr限定在了编译期常量

## 元编程经常使用constexpr

用来简化元函数

```cpp
template<class U, class V>
constexpr bool is_same_v = std::is_same<U, V>::value;
```

这个value需要是元编程中的常成员：

```cpp
template<class U, class V>
struct is_same {
    static constexpr int value = false; //示例
}
```

调用的时候可以直接

```cpp
auto x = is_same_v<int, float>;
```

