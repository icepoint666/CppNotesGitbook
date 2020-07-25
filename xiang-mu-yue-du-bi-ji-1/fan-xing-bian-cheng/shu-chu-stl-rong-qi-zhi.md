---
description: （阅读笔记）
---

# 输出STL容器值

源码地址：[https://github.com/Light-City/CPlusPlusThings/blob/master/tool/output/output\_container.h](https://github.com/Light-City/CPlusPlusThings/blob/master/tool/output/output_container.h)

改进版地址：

### 代码结构概览

* 重载operator &lt;&lt; 函数：`os << container;`
  * 要求1：如果本身这个容器由ostream的输出流函数，那么就不要在上面重载了（利用has\_output\_function判断）
  * 要求2：实现两种版本：容器/pair（用is\_pair判断）: 容器的话，用迭代器对容器进行遍历，然后对每个元素调用output\_element输出函数，pair的话直接输出
* output\_element输出函数：
  * 要求1：利用泛型编程技术判断元素是不是pair，对元素的输出也要**实现两个版本**
  * 要求2：对于元素内容的输出判断元素的类型：调用ischarOrString函数
  * 要求3：因为输出的话，希望遇到字符串有一个`“str”`这个效果，遇到char有一个`‘c’`这个效果，其他类型不需要这类效果
* ischarOrString函数：判断类型T是不是char或者string
* output函数：输出到os流里面
* has\_output\_function判断结构体

### 泛型编程函数定义要写在头文件

因为需要编译时求值，从而确定类型

### std::false\_type和std::true\_type

属于`#include<type_traits>`头文件中,`c++11`的特性

**作用**：用于泛型编程，表示True或者False

本身的值是**static**，文件一开始创建就存储在全局的静态变量

另外也是**constexpr**，确保在编译时可以返回值，从而可以帮助类型的确定

**本身的定义（简化版）**

```cpp
struct false_type {
    static constexpr bool value = false;
    constexpr operator bool() const noexcept { return value; }
    // There is more here, but it doesn't really matter 
};
```

```cpp
struct true_type {
    static constexpr bool value = true;
    constexpr operator bool() const noexcept { return value; }
    // There is more here, but it doesn't really matter for your question
};
```

**用来判断一个类型是否是某个类型：**

是用其他结构体来**继承std::false\_type或者std::true\_type**

例如：判断一个类型是不是pair

* 如果类型T是pair，那么会与模板中带有std::pair&lt;T,U&gt;相匹配，也就自然而然继承了std::true\_type，返回的值是true
* 如果类型T不是pair，就会匹配另一个空的模板，然后返回false

```cpp
// 检测是否是pair
template<typename T>
struct is_pair: std::false_type {
};
template<typename T, typename U>
struct is_pair<std::pair<T, U>> : std::true_type {
};
template<typename T>
inline constexpr bool is_pair_v = is_pair<T>::value;
```

注意：这种类struct不能实例化（instantiated），有时可以加一个`static_assert`来防止实例化的发生

```cpp
template <typename T>
class Sensor
{
    static_assert(is_pair<T>::value, "RadarSensor must be created using Identifier template");
};
```

