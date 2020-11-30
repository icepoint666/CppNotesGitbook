# C++中四种cast类型转换

### 强制类型转换

C中类型转换 例如malloc函数分配内存时需要从`void *`转换成你指定的类型指针。如下面这样

```cpp
int* block = (int*)malloc(sizeof(int));
```

上面的代码是将`void*`转换成`int*`，这种转换方式在C语言中称为`强制转换`。它的好处是简洁，灵活；缺点是需要人来决定转换后类型是否正确

C++中最接近于C的强制转换的是reinterpret\_cast

### static\_cast

**static\_cast**主要用于**不同类型变量之间的转换**及**左值转右值，指针和引用的转换不能用static\_cast**

**基本数据类型**之间的相互转换，enum、struct、 int、char、float等

**（理解C/C++编程中一切关于静态与动态的理解：静态就是编译时，动态就是运行时）**

举个例子：

```cpp
//不同类型之间的转换
double d = 10.1;
int i = static_cast<int>(d);

//左值转右值
int lv = 10;
int && rv = static_cast<int&&>(lv);
```

这里需要注意的是：int 转 double是隐式转换，右值转左值也是隐性转换，所以对这两种情况是不需要用static\_cast进行显示转换的。

上面我们说的是普通类型的转换。而**对于类对象来说，static\_cast不能直接将一种类对象转成另一种类对象**。比如：

```cpp
class A{
    public:
        int a;
};

class B:public A {
    public:
        int b;
};

class C {
    public:
        int c;
};

int main(int argc, char *argv[]){
    A a;
    B b;
    b = static_cast<B>(a); //不允许static_cast将一个对象转成另一个对象
}
```

像上面这种用static\_cast将A类型的对象转成B类型对象是不允许的。但你**可以利用static\_cast将基类指针/引用转成子类指针/引用**：

```cpp
A ca;
B & crb = static_cast<B&>(ca);
...
A * pa = new A();
B * cpb = static_cast<B*>(pa);
```

但这里有个前提条件，即**只有有父子关系的类之间才可以做如上转换**，否则编译失败。还有，虽然以上两种使用static\_cast的方式都可以编译通过，但用户自己要防止越界访问的问题。

static\_cast除了上面讨论的几种情况外，还有一点需要牢记，即**static\_cast不允许不同类型之间指针/引用的转换\(有父子关系的类对象除外\)**。看个具体的例子：

```cpp
double *pd  = new double();
int * pi = static_cast<int*>(pd); //报错
```

上面的代码在编译时会报错，因为它不允许不同类型之间的指针或引用转换。对于有父子关系的类对象之间之所以可以转换是因为static\_cast把它们当做同一类型看待了。

总结来说，**static\_cast主要用于不同类型变量之间的转换，指针和引用的转换不能用static\_cast，而应该用reinterpret\_cast。**

\*\*\*\*

### reinterpret\_cast

**用于不同类型指针间的类型转换，但是不能进行类型转换**

reinterpret\_cast类似于C语言中不同类型指针间的类型转换，**最接近于C的强制转换**。举个例子：

```cpp
double *pd  = new double();
int * pi = reinterpret_cast<int*>(pd);
```

上面的代码是将`double*` 转成 `int*`。如果你使用static\_cast做这种转换转换是不允许的，但改用reinterpret\_cast就一切正常。 当然，如果你用reinterpret\_cast做static\_cast善长的变量类型转换也会报错。从上面的描述我们应该知道reinterpret\_cast与static\_cast之间的区别了。

reinterpret\_cast对于对象指针/引用的转换与普通类型的指针/引用转换是一样的。因此不同类型的对象指针/引用可以随意转换，但转换后是否会引起问题它不关心，这要由开发人员自己保证。

```cpp
A * pa = new A();

B b;
B & rb = B();

C * cc = reinterpret_cast<C*>(pa);
C & rcc = reinterpret_cast<C&>(rb);
```

**总结一下，reinterpret\_cast是对指针/引用的转换，其中必须至少有一个是指针或引用，否则它会报错**。



### const\_cast

这个比较简单，它的作用是去掉**指针/引用**中的const限制 / volatile限制。这里要注意的是**被转换的一定是指针/引用的const**，而**常数的const是不能去掉的（相当于是通过指针引用的方式来去掉const）**。举个例子：

```cpp
const int a = 10;
int b  = const_cast<int>(a); //报错
```

```cpp
const int * pca = new int(10);
int * pa = const_cast<int*>(pca); //正确
```

对于const\_cast来说，它只能将同一类型的const 指针/引用 转成非const指针/引用，不能从const int \*转换为double \*

**总结一下，const\_cast是一个专门去掉同一类型的const限制的类型转换方法。它不如static\_cast和reinterpret\_cast应用的广泛。**

\*\*\*\*

### dynamic\_cast

主要用在**类多态**的时候，其他三种都是编译时完成的，dynamic\_cast是运行时处理的，主要特点就是**运行时类型检查**。这个转换方法限制比较多**：**

1. 它只能处理类对象，而且基类中一定要有虚函数，否则编译不通过
2. 它只能处理指针（或引用）
3. 它只能用于将子对象转换成父对象这样的操作

一个例子：

```cpp
A * a;
B * b =new B();
a = dynamic_cast<A*>(b);
```

大多数情况都是只有使用上面这一种情况

**为什么一定要有虚函数**

1. 类中存在虚函数，就说明它有想要让基类指针或引用指向派生类对象的情况，此时转换才有意义。 
2. 主要是由于**运行时类型检查**需要运行时类型信息，而这个信息存储在类的虚函数表，只有定义了虚函数的类才有虚函数表

### **static\_cast对比于dynamic\_cast的优势（在类转换上）**

**对于static\_cast:**

上行转换（派生类----&gt;基类）是安全的

下行转换（基类----&gt;派生类，父类指针转换为子类指针）由于没有动态类型检查，所以是不安全的

**对于dynamic\_cast:**

在类层次间进行**上行转换**时**，**dynamic\_cast和static\_cast的效果是一样的。

在进行**下行转换**时，dynamic\_cast具有类型检查的功能，比 static\_cast更安全。

**向下转换的成功与否**与将要转换的类型有关（所以才需要类型检查），即要转换的指针指向的对象的实际类型与转换以后的对象类型一定要相同，否则转换失败。

