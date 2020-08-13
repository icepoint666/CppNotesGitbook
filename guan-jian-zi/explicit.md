# explicit

* explicit 修饰**构造函数**时，可以防止**隐式转换**和**拷贝初始化**
* explicit 修饰**转换函数**时，可以防止**隐式转换**

### **什么是拷贝初始化？**

不使用explicit关键字看看会发生什么？我们发现下面这个代码明明我们给函数f\(\)传递的参数是一个整数，但是编译器却自动调用了类A的构造函数，这种就叫做拷贝初始化。这是超出预期的。

如果你不希望这样那么请在构造函数前面加上explicit关键词禁止编译器这种自动调用拷贝初始化的行为，即**explicit A\(int x\){}**

```cpp
#include <iostream>
using namespace std;

class A{
public:
	  A(int x){
		    cout<<"我被用了"<<endl;
	  }
};

void f(A a){
}
int main( ){
	  f(1);
	  return 0;
}

```

### **编译器会对构造函数的参数进行隐式转换**

假设你想写个类把整数或者字符数组变成字符串。下面代码如果不使用explicit关键字你会发现下面这个程序会有bug。

```cpp
#include <iostream>
using namespace std;
#include<algorithm>
class Str{
	public:
	 Str(int x){
		cout<<"我是想把整数变字符串"<<endl;
	}
	Str(const char* a)
	{
		cout<<"我是想把字符数组变字符串"<<endl;
	}

};
```

### **explicit 使用**

```cpp
struct A
{
	A(int) { }
	operator bool() const { return true; }
};

struct B
{
	explicit B(int) {}
	explicit operator bool() const { return true; }
};

void doA(A a) {}

void doB(B b) {}

int main()
{
	A a1(1);		// OK：直接初始化
	A a2 = 1;		// OK：复制初始化
	A a3{ 1 };		// OK：直接列表初始化
	A a4 = { 1 };		// OK：复制列表初始化
	A a5 = (A)1;		// OK：允许 static_cast 的显式转换 
	doA(1);			// OK：允许从 int 到 A 的隐式转换
	if (a1);		// OK：使用转换函数 A::operator bool() 的从 A 到 bool 的隐式转换
	bool a6（a1）;		// OK：使用转换函数 A::operator bool() 的从 A 到 bool 的隐式转换
	bool a7 = a1;		// OK：使用转换函数 A::operator bool() 的从 A 到 bool 的隐式转换
	bool a8 = static_cast<bool>(a1);  // OK ：static_cast 进行直接初始化

	B b1(1);		// OK：直接初始化
	B b2 = 1;		// 错误：被 explicit 修饰构造函数的对象不可以复制初始化
	B b3{ 1 };		// OK：直接列表初始化
	B b4 = { 1 };		// 错误：被 explicit 修饰构造函数的对象不可以复制列表初始化
	B b5 = (B)1;		// OK：允许 static_cast 的显式转换
	doB(1);			// 错误：被 explicit 修饰构造函数的对象不可以从 int 到 B 的隐式转换
	if (b1);		// OK：被 explicit 修饰转换函数 B::operator bool() 的对象可以从 B 到 bool 的按语境转换
	bool b6(b1);		// OK：被 explicit 修饰转换函数 B::operator bool() 的对象可以从 B 到 bool 的按语境转换
	bool b7 = b1;		// 错误：被 explicit 修饰转换函数 B::operator bool() 的对象不可以隐式转换
	bool b8 = static_cast<bool>(b1);  // OK：static_cast 进行直接初始化

	return 0;
}
```

