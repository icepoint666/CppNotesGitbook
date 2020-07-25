---
description: （阅读笔记）
---

# 输出STL容器值

源码地址：[https://github.com/Light-City/CPlusPlusThings/blob/master/tool/output/output\_container.h](https://github.com/Light-City/CPlusPlusThings/blob/master/tool/output/output_container.h)

缺陷：不能处理嵌套容器的情况，容器中每个元素必须存储string，pair或者基本类型

改进：TODO

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

### 

### 泛型编程函数定义要写在头文件

因为需要编译时求值，从而确定类型

### 

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

### 

### std::is\_same和std::is\_same\_v

属于`#include<type_traits>`头文件中,`c++11`的特性

**作用：判断两个类型是不是同一种类型**

```cpp
template<class T, class U>
struct is_same : std::false_type {};
 
template<class T>
struct is_same<T, T> : std::true_type {};
```

**std::is\_same\_v**在`c++17`中定义了，如果版本不到c++17，那么自己定义

```cpp
template< class T, class U >
inline constexpr bool is_same_v = is_same<T, U>::value;
//如果inline会报错，那么就去掉inline
```

一些例子：

```cpp
// 'int' is implicitly 'signed'
 std::cout << std::is_same<int, int>::value << "\n";          // true
 std::cout << std::is_same<int, unsigned int>::value << "\n"; // false
 std::cout << std::is_same<int, signed int>::value << "\n";   // true

 // unlike other types, 'char' is neither 'unsigned' nor 'signed'
 std::cout << std::is_same<char, char>::value << "\n";          // true
 std::cout << std::is_same<char, unsigned char>::value << "\n"; // false
 std::cout << std::is_same<char, signed char>::value << "\n";   // false
```

### 

### std::decay\_t

属于`#include<type_traits>`头文件中,`c++14`的特性

作用：可以将类型退化，例如带引用的类型&可以退化成不带引用的

**退化规则，不同类型有所不同**

**int:** const int &,  const int,  const &都会退化成int

**char**: 同样也是都会退化成char

**char \*不一样，const char \*不会退化， char \*也不会退化，只会退化引用**

\*\*\*\*

**示例：判断类型是不是字符串类型，要考虑三种字符串形式**

```cpp
const char * s = "Hello World!";
using std::decay_t;
using element_type = decay_t<decltype(s)>; 
// 对于const char *类型只能一下子decay到const char *，不会与char *匹配
constexpr bool is_char_v = is_same_v<element_type, char>;
constexpr bool is_string_v = is_same_v<element_type, char *> ||       // False
                             is_same_v<element_type, const char *> || // True
                             is_same_v<element_type, std::string>;    // False
int a = 5;
const int& b = a;
const int& c = a;
using element_type_1 = decay_t<decltype(b)>; //可以一下子从const int & 直接decay到int
using element_type_2 = decltype(c); //不decay，还是const int &类型
constexpr bool is_same_const = is_same_v<element_type_2, const int&>; // True
```



### **std::enable\_if和std::enable\_if\_t**

属于`#include<type_traits>`头文件中

 std::enable\_if是`c++11`的特性

 std::enable\_if\_t是`c++14`的特性

作用：应用在有多个重载版本的函数时，可以根据类型来选择不同版本的重载函数

 `std::enable_if` 顾名思义，满足条件时类型有效。

如果只有一个重载版本：

* 如果为true，那么类型就有定义
* 如果为false，那么编译会报错

如果有多个重载版本：

* 当 enable\_if 的条件为true 时，优先匹配 struct enable\_if 这个模板

```cpp
template <bool, typename T=void>
struct enable_if {
};

template <typename T>
struct enable_if<true, T> {
  using type = T;
};
```

**std::enable\_if\_t 大概实现**如下， 也就是说， **enable\_if\_t 只是using enable\_if 的type**

```cpp
//如果c++11，可以按照下面的方式来定义enable_if_t
template< bool B, class T = void >
using enable_if_t = typename enable_if<B,T>::type;
```

**使用：要求类型是整型才有意义**

```cpp
template<typename T, typename std::enable_if_t<std::is_intergral<T>::value> *=nullptr> 
//或者
template<typename T, typename = std::enable_if_t<std::is_intergral<T>::value> >

```

**元素是否为pair类型，输出函数不一样**

```cpp
//如果是pair的类型，输出first与second
template<typename T, typename Cont>
auto output_element(std::ostream &os, const T &element,
                    const Cont &)
-> typename std::enable_if<is_pair<typename Cont::value_type>::value, bool>::type { //这个形式表示auto的返回类型，泛型编程需要编译的时候才能推断出来
    int ftype = ischarOrString(element.first);
    int stype = ischarOrString(element.second);

    output(element.first, ftype, os);
    os << " => ";
    output(element.second, stype, os);
    return true;
}
//不是pair的类型，输出本身内容
template<typename T, typename Cont>
auto output_element(std::ostream &os, const T &element,
                    const Cont &)
-> typename std::enable_if<!is_pair<typename Cont::value_type>::value, bool>::type {
    int etype = ischarOrString(element);
    output(element, etype, os);
    return false;
}

//使用：
output_element(os, elem, container);
```



### auto作为返回类型，-&gt;指定编译时返回类型

很多时候不到编译的时候是无法知道这个类型的

c++不像python，auto作为函数的返回值，C++是需要给这个auto提供推断类型的，不能像python一样运行时推断类型

**示例1：auto配合decltype**

```cpp
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) // return type depends on template parameters
                                      // return type can be deduced since C++14
{
    return t+u;
}
```

**示例2：auto配合typename**

```cpp
auto output_element(std::ostream &os, const T &element,
                    const Cont &)
-> typename std::enable_if<is_pair<typename Cont::value_type>::value, bool>::type {
//如果类型是is_pair那么auto就会被激活为指定返沪Cont类型，重载的时候发现满足类型就会用这个函数
//std::enable_if<T, bool>::type就是std::enable_if_t，只不过c++11，c++14的区别
```

### 

### std::declval

属于`#include<utility>`头文件中

作用：有些类中的默认构造函数是delete的，因为不能实例化，例如iostream中的ostream

对于没有构造函数的类，如果**想调用它的成员函数，可以通过declval不实例化**就返回左值，从而调用

示例1：

```cpp
#include <utility>
#include <iostream>
 
struct Default { int foo() const { return 1; } };
 
struct NonDefault
{
    NonDefault() = delete;
    int foo() const { return 1; }
};
 
int main()
{
    decltype(Default().foo()) n1 = 1;                   // type of n1 is int
//  decltype(NonDefault().foo()) n2 = n1;               // error: no default constructor
    decltype(std::declval<NonDefault>().foo()) n2 = n1; // type of n2 is int
    std::cout << "n1 = " << n1 << '\n'
              << "n2 = " << n2 << '\n';
}
```

示例2：

```cpp
template<class U>
static auto output(U *ptr)
-> decltype(std::declval<std::ostream &>() << *ptr, std::true_type()); 
//为什么ostream要用declval，因为ostream类定义中把default constructor delete了,不能实例化
```

### 

### 模板：编译期间检查类型T的成员函数是否存在

```cpp
// Non-templated helper struct:
struct _test_has_foo {
    template<class T>
    static auto test(T* p) -> decltype(p->foo(), std::true_type());

    template<class>
    static auto test(...) -> std::false_type;
};

// Templated actual struct:
template<class T>
struct has_foo : decltype(_test_has_foo::test<T>(0))
{};
```

示例：**判断输出容器本身有没有定义ostream的输出操作（成员函数）**，如果没有的话，才可以用上我们定义的ostream &lt;&lt;函数

```cpp
template<typename T>
struct has_output_function {
    template<class U>
    static auto output(U *ptr)
    -> decltype(std::declval<std::ostream &>() << *ptr,  //为什么ostream要用declval，因为ostream类定义中把default constructor delete了： 
            std::true_type());                           

    template<class U>
    static std::false_type output(...); 

    static constexpr bool value =
            decltype(output<T>(nullptr))::value;
};

template<typename T>
inline constexpr bool has_output_function_v = has_output_function<T>::value;
```

**使用has\_function的模板：**

```cpp
// 针对没有输出函数的容器处理,如果有输出函数的容器，直接用它们自己定义的
template<typename T,
        typename = std::enable_if_t<!has_output_function_v<T>>> //std::enable_if_T里面的只有判定为true的时候 typename T才生效
auto operator<<(std::ostream &os, const T &container)
-> decltype(container.begin(), container.end(), os) {
    os << "{ ";
    if (!container.empty()) {
        bool on_first_element = true;
        for (auto elem:container) {
            if (!on_first_element) {
                os << ", ";
            } else {
                on_first_element = false;
            }
            output_element(os, elem, container);
        }
    }
    os << " }";
    return os;
}

```

