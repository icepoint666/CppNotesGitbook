# 多态（两类四种）

消息以多种形式显示的能力，多态是以封装和继承为基础的。

### C++ 的四种多态及实现

[https://catonmat.net/cpp-polymorphism](https://catonmat.net/cpp-polymorphism)

总的来说分为两类：

* 静态多态（编译时多态）：重载，模板
* 动态多态（运行时多态）：虚函数（重写）

讲多态可以讲这四种

1. 重载多态（Ad-hoc Polymorphism，编译期）：函数重载、运算符重载
2. 子类型多态（Subtype Polymorphism，运行期）：虚函数
3. 参数多态性（Parametric Polymorphism，编译期）：类模板、函数模板
4. 强制多态（Coercion Polymorphism，编译期/运行期）：基本类型转换、自定义类型转换



### **静态多态（编译期/早绑定）**

函数重载

```cpp
class A
{
public:
    void do(int a);
    void do(int a, int b);
};
```

### **动态多态（运行期/晚绑定）**

* 虚函数：用 virtual 修饰成员函数，使其成为虚函数

**注意：**

* 普通函数（非类成员函数）不能是虚函数
* 静态函数（static）不能是虚函数
* 构造函数不能是虚函数（因为在调用构造函数时，虚表指针并没有在对象的内存空间中，必须要构造函数调用完成后才会形成虚表指针）
* 内联函数不能是表现多态性时的虚函数，解释见：[虚函数（virtual）可以是内联函数（inline）吗？](https://github.com/huihut/interview#%E8%99%9A%E5%87%BD%E6%95%B0virtual%E5%8F%AF%E4%BB%A5%E6%98%AF%E5%86%85%E8%81%94%E5%87%BD%E6%95%B0inline%E5%90%97)

### 实现一个动态多态的示例

```cpp
class Shape                     // 形状类
{
public:
    virtual double calcArea()
    {
        ...
    }
    virtual ~Shape();
};
class Circle : public Shape     // 圆形类
{
public:
    virtual double calcArea();
    ...
};
class Rect : public Shape       // 矩形类
{
public:
    virtual double calcArea();
    ...
};
int main()
{
    Shape * shape1 = new Circle(4.0); //虚函数多态一定是指针或者引用
    Shape * shape2 = new Rect(5.0, 6.0);
    shape1->calcArea();         // 调用圆形类里面的方法
    shape2->calcArea();         // 调用矩形类里面的方法
    delete shape1;
    shape1 = nullptr;
    delete shape2;
    shape2 = nullptr;
    return 0;
}
```

