# 深入std::move的实现

### std::move：右值转换器

将左值转为右值，将右值也转为右值，总是**无条件的将它的参数转换为右值**

源码：

```cpp
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
    return static_cast<typename remove_reference<T>::type&&>(t);
}
```

接下来一步步解释为什么这个结构可以**识别引用，并完成右值转换**，还不报错

涉及知识点：

* 模板的类型推导
* 通用引用
* 类型成员
* 引用折叠

### 模板的类型推导

**模板类型推导就是：根据传入参数的类型，来推断typename T**

有如下三种类型:

* 传值的模板
* 传引用的模板
* 通用模板

```cpp
template <typename T>
void f(T param);

template <typename T>
void func(T& param);

template <typename T>
void function(T&& param);

int main(int argc, char *argv[]) {

    int x = 10;         // x是int
    int & rr = x;       // rr是 int &
    const int cx = x;   // cx是const int
    const int& rx = x;  // rx是const int &
    int *pp = &x;       // pp是int *

    //下面是传值的模板
    //由于传入参数的值不影响原值，所以参数类型退化为原始类型
    f(x);               // T是int
    f(cx);              // T是int
    f(rx);              // T是int
    f(rr);              // T是int
    f(pp);              // T是int*，指针比较特殊，直接使用

    //下面是传引用模板:
    //如果输入参数类型有引用，则去掉引用
    //如果没有引用，则输入参数类型就是T的类型
    func(x);            // T为int
    func(cx);           // T为const int
    func(rx);           // T为const int
    func(rr);           // T为int
    func(pp);           // T是int*，指针比较特殊，直接使用

    //下面是通用引用模板
    //与引用模板规则一致，可以接收左值和右值（下面介绍通用引用）
    function(x);        // T为int
    function(5);        // T为int
}
```

### 通用引用

我们前面**move的输入参数类型**`T&& t`**称为通用引用类型**。

**通用引用**就是它**既可以接收左值也可以接收右值**

**有两种通用引用：\(1\)auto \(2\)T&&**

```cpp
template<typename T>
void f(T&& param){
    std::cout << "the value is "<< param << std::endl;
}

int main(int argc, char *argv[]){

    int a = 123;
    auto && b = 5;   //auto通用引用，可以接收右值
    int && c = a;    //错误，右值引用，不能接收左值
    auto && d = a;   //auto通用引用，可以接收左值
    
    const auto && e = a; //错误，加了const就不再是通用引用了

    f(a);         //T&&通用引用，可以接收左值
    f(10);        //T&&通用引用，可以接收右值
}
```

**接下来讨论一下move的返回类型**

```cpp
typename remove_reference<T>::type&&
```

这个返回类型是什么意思呢，首先了解一下这个typename ::type的意思

### 类型成员

**C++的类成员**有三种类型

* 成员函数
* 成员变量
* 静态成员

C++11之后又增加了一种成员称为**类型成员**

 类型成员与静态成员一样，它们都属于类而不属于对象，访问它时也与访问静态成员一样用`::`访问

因为类型成员是类型，所以前面加typename，所以这个`typename remove_reference<T>::type&&`

就是remove\_reference类成员中的一个类型成员叫做type的类型，然后&&表示返回的是右值引用

**继续看，分析一下remove\_reference类**

```cpp
template <typename T>
struct remove_reference{
    typedef T type;  //定义T的类型别名为type
};

template <typename T>
struct remove_reference<T&> //左值引用
{
    typedef T type;
}

template <typename T>
struct remove_reference<T&&> //右值引用
{
   typedef T type;
}
```

在C++中struct与class基本是相同的，不同点是class默认成员是private，而struct默认是public，所以使用struct代码会写的更简洁一些。

这就是前面的**模板推导的原理**：remove\_reference利用模板的自动推导获取到了实参去引用后的类型。

简化后的std::move就是这样：

```cpp
//输入右值
int && move(int&& && t){
    return static_case<int&&>(t);
}

//输入左值
int && move(int& && t){
    return static_case<int&&>(t);
}
```

### 引用折叠

 C++中根本就不存 `int& &&`、`int && &&`这样的语法，但在编译器内部是能将它们识别出来的

实际上，当编译器遇到这类形式的时候它会使用引用折叠技术，将它们变成我们熟悉的格式。其规则如下：

* `int & &` 折叠为 `int&`
* `int & &&` 折叠为 `int&`
* `int && &` 折叠为 `int&`
* `int && &&` 折叠为 `int &&`

经过这一系列的操作之后，对于一个具体的参数类型`int & a`，std::move就变成了下面的样子：

```cpp
int && move(int& t){
    return static_cast<int&&>(t);
}
```



