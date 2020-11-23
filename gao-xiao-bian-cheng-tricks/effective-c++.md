# Effective C++（浓缩部分）

* 《Effective C++》

[https://github.com/huihut/interview\#effective](https://github.com/huihut/interview#effective)

### 02. 尽量以 const、enum、inline 替换 \#define

**也就是尽量 以编译器 替换 预处理器**

* **常量的话：用const, enum**
* **函数的话：用inline**

\#define经常会有一些误用， 因为define未必会被编译器看到，在预编译期间就被预处理器替换走了

这时候如果有一些错误，编译器是无法再调试期间发现的

例如：本身想使用这个函数宏来代替函数调用，为了减少开销

```cpp
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```

但是会出现一些不可思议的事

```cpp
int a = 5, b = 0;
CALL_WITH_MAX(++a, b);    //a会被累加2次
CALL_WITH_MAX(++a, b+10); //a会被累加1次
```

就会导致函数的行为未定义

**解决办法：**

```cpp
template <typename T>
inline void callWithMax(const T& a, const T& b){
    f(a > b ? a : b);
}
```

### 03. 尽可能使用const

作用：告诉**编译器**和**其他程序员** 该值保持不变

**const实施于成员函数**目的：

* 为了确认该成员函数可作用于const对象的身上
* 另外，当const和non-const成员有着实质等价的实现时，令non-const版本的实现去调用const版本，可以尽量减少代码的重复，少写一份函数代码（这个很重要）

### 04. 确定对象被使用前已经先被初始化

### 06.若不想使用编译器自动生成的函数，就该明确拒绝

面向对象建造类的过程中，**编译器会自动生成 默认构造函数，拷贝构造函数，拷贝赋值运算符，析构函数等**... 

但是有时候是需要禁止拷贝的，例如单例对象，这时候需要禁止掉编译器这些函数

**方法1：在声明中使用delete即可**

```cpp
Uncopyable (const Uncopyable&) = delete;
Uncopyable& operator = (const Uncopyable&) = delete;
```

**方法2：在类中声明为private函数**

然后使用另一个类private继承这个类，只去使用另一个类

```cpp
class Uncopyable{
   ...
private:
    Uncopyable (const Uncopyable&);
    Uncopyable& operator = (const Uncopyable&);
}

class HomeForSale : private Uncopyable{
   ...
}
```

### 08. 别让异常逃离析构函数

析构函数应该**吞下不传播异常**，**或者结束程序**，而不是吐出异常；（C++不禁止但不鼓励）

**关键！！**如果某个操作可能在失败时抛出异常，又要求**必须处理该异常**，那么这个**异常必须来自于析构函数以外的某个函数**，不能是析构函数处理异常

**主要原因：在析构函数抛出异常时，允许它离开这个析构函数，这时候就会造成问题**

**解决办法：使用abort（调用异常，最好是先终止，记录下失败）**

```cpp
DBConn::~DBConn(){
    try {db.close();}
    catch (...){
        //制作运转记录，记下对close的失败
        std::abort();
    }
}
```

### 09. 绝不在构造函数或者析构函数中调用virtual函数

原因：因为这类调用从不下降至derived class，也就是不会有多态的行为

### 10.编写operator=的时候，注意事项

**①要注意，返回reference给自己\(\*this\)**

**②要注意，处理“自己给自己赋值”的特殊情况，自我赋值安全**

**解决样例：**

```cpp
Widget & Widget::operator=(const Widget& rhs){
    if(this == &rhs)return *this; //11
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this; //10
}
```

### 13. RAII思想：资源取得时机便是初始化时机

这一点主要是在std::lock\_guard上体现了这点

* 拿到锁的时候就是初始化管理对象的时候
* 析构的时候就是释放锁的时候

### 14. RAII对象可能要 禁止复制 or 对底层资源采用“引用计数法”

* **禁止复制**：因为管理的资源是唯一的，例如锁，可以通过禁止复制的方法来管理
  * 这时候需要采用条款6的方法
* **对底层资源采用“引用计数法”**：有时候希望保有资源，直到最后一个使用者被销毁的时候才释放
  * 通过shared\_ptr来实现

### 16.使用相同行为的new和delete

错误示例：创建的是数组，但是只删除了一个对象（需要注意，**易错**）

```cpp
std::string* stringArray = new std::string[100]; 
...
delete stringArray;
```

正确示例：\(数组与数组对应）

```cpp
std::string* stringPtr1 = new std::string; //对应的应该是对象删除
std::string* stringPtr2 = new std::string[100]; //对应的应该是对象数组地删除
...
delete stringPtr1;                         //删除对象
delete [] stringPtr2;                      //删除一个对象的数组
```

### 17.要使用独立语句将new对象置入智能指针

因为编译器的优化功能，可能导致指令重排

**错误示例：**

```cpp
//调用函数时，进行智能指针的构造
processWidget(std::shared_ptr)
```



### 20. 传引用替代传值

* **效率很高，减少拷贝构造开销**
* **可以避免切割问题**

### **21.该返回对象的时候，就不要返回reference**

不要以为返回reference可以提升性能

**糟糕的示例1：**

```cpp
const Rational& operator* (const Rational& lhs, const Rational& rhs){
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
    return result;
}
```

**依然要付出一个构造函数的代价**

**返回一个reference指向一个local对象**，在函数退出销毁时，返回的引用指向的一个被销毁的对象，就是一个未被定义的行为

**糟糕的示例2：**

```cpp
const Rational& operator* (const Rational& lhs, const Rational& rhs){
    Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return *result;
}
```

**依然要付出一个构造函数的代价**

**存在的问题：就是谁该对这个被new出来的对象delete，如果将来没有人去delete，就内存泄漏了**

