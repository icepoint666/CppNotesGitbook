# namespace, ::运算符

### 为什么需要namespace命名空间

C++中经常使用多个独立开发的库来完成项目，由于库的作者或开发人员没见过面，因此库与库之间命名冲突在所难免

命名空间作用就是对这些命名进行 逻辑空间划分（不是物理单元划分）通过使用namespace XXX把库中的变量、函数、类型、结构等包含在名字空间中，形成自己的作用域，避免 名字冲突

```cpp
 namespace xxx
 {

 }// 没有分号
```

### 一些原则

* 同名的namespace有自动合并，同名的namespace中如果有重名的依然会命名冲突
* namespace使用方法：
  * space::function
  * using namespace space
* namespace的嵌套

  ```cpp
  n1::n2::num;
   namespace n1
   {
       int num = 1;
       namespace n2
       {   
           int num = 2;
           namespace n3
           {

           }
       }
   }
  ```

### :: 范围解析运算符

**分类**

1. 全局作用域符（`::name`）：用于类型名称（类、类成员、成员函数、变量等）前，表示作用域为全局命名空间
2. 类作用域符（`class::name`）：用于表示指定类型的作用域范围是具体某个类的
3. 命名空间作用域符（`namespace::name`）:用于表示指定类型的作用域范围是具体某个命名空间的

**:: 使用**

```cpp
int count = 11;         // 全局（::）的 count

class A {
public:
	static int count;   // 类 A 的 count（A::count）
};
int A::count = 21;

void fun()
{
	int count = 31;     // 初始化局部的 count 为 31
	count = 32;         // 设置局部的 count 的值为 32
}

int main() {
	::count = 12;       // 测试 1：设置全局的 count 的值为 12

	A::count = 22;      // 测试 2：设置类 A 的 count 为 22

	fun();		        // 测试 3

	return 0;
}
```

