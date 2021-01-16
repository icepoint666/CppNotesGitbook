# pimpl惯用法

### Pointer to implementation

这种技巧可以将⼀个**类数据成员**替换成⼀个**指向包含具体实现的类或结构体的指针**

**示例如下：传统方法**

```cpp
class Widget(){ //定义在头⽂件`widget.h` 
public: 
    Widget(); 
    ... 
private: 
    std::string name; 
    std::vector<double> data; 
    Gadget g1, g2, g3; //Gadget是⽤⼾⾃定义的类型 
}
```

类 Widget 的数据成员包含有类型 std::string ， std::vector 和 Gadget ， 定义有这些类型的头 ⽂件在类 Widget 编译的时候，必须被包含进来，这意味着类 Widget 的使⽤者必须要 \#include &lt;string&gt;&lt;vector&gt; , 以及 gadget.h

**pimpl惯用法：unique\_ptr来实现**

```cpp
//在"Widget.h"中 
class Widget{ 
public: 
    Widget(); 
    ... 
private: 
    struct Impl; //声明⼀个实现结构体
    std::unique_ptr<Impl> pImpl; //使⽤智能指针而不是原始指针
}
//以下代码均在实现⽂件 widget.cpp⾥
#include "widget.h"
#include "gadget.h"
#include <string> 
#include <vector> 
struct Widget::Impl{
    std::string name;
    std::vector<double> data;
    Gadget g1,g2,g3; 
}
Widget::Widget()
    : pImpl(std::make_unique<Imple>())
    {}
```

以上的代码能编译，但是，最普通的 Widget ⽤法却会导致编译出错

```cpp
#include "widget.h"
Wdiget w; //编译出错
```

为了解决这个问题，你只需要确保在编译器⽣成销毁 std::unique\_ptr 的代码之 前， Widget::Impl 已经是⼀个完成类型\(complete type\)。

```cpp
//在"Widget.h"中 
class Widget{ 
public: 
    Widget(); 
    ~Widget(); // declaration only
    ... 
    Widget(Widget&& rhs); //仅声明 declaration only
    Widget& operator=(Widget&& rhs);
private: 
    struct Impl; //声明⼀个实现结构体
    std::unique_ptr<Impl> pImpl; //使⽤智能指针而不是原始指针
}
//以下代码均在实现⽂件 widget.cpp⾥
#include "widget.h"
#include "gadget.h"
#include <string> 
#include <vector> 
struct Widget::Impl{
    std::string name;
    std::vector<double> data;
    Gadget g1,g2,g3; 
}
Widget::Widget()
    : pImpl(std::make_unique<Imple>())
    {}
    
Widget::~Widget() //析构函数的定义(译者注：这⾥⾼亮) 
    {}

Widget(Widget&& rhs) = default; //在这⾥定义
Widget& operator=(Widget&& rhs) = default;
```

pImpl 惯⽤法是⽤来减少类实现者和类使⽤者之间的编译依赖的⼀种⽅法

