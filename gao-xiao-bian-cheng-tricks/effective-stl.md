# Effective STL

### 0.STL分类

#### 序列容器 与 关联容器

序列：vector， string， deque，list，slist（单向链表，非标准）

关联：map，multimap，set，multiset，hashset, hashmultiset...

#### 连续内存容器 与 基于节点容器

连续内存容器，contiguous memory container

* 元素存放在一块或者多块内存中，每块内存中存有多个元素
* 当有新元素插入或者元素删除的时候，同一块内存中的其他元素都要向前向后移动，为了腾出空间
* vector， deque，string
* 弊端：这样会影响效率，并且也会影响异常安全性

基于节点的容器，node-based memory container

* 在每一个动态分配的内存块中只存放一个元素，容器中元素的插入与删除只影响当前节点的指针，不会影响节点内容本身
* list，slist，标准的哈希容器（平衡树实现的），非标准的哈希容器



#### 迭代器的五种类型

五种迭代类型有了解哪些

* 输入迭代器 input iterator 也就是只读迭代器
  * 每个遍历到的位置只能被读取一次
* 输出迭代器 output iterator 类似 只写迭代器
* 前向迭代器 兼顾输入输出迭代器的功能
  * 可以对同一位置重复的读或者写
  * 前向迭代器不支持`—operator`也就是只能前向移动
* 双向迭代器 类似于前面的 支持前向移动与后向移动
  * 示例：list迭代器就是这样
* 随机访问迭代器 random access iterator
  * 拥有双向迭代器的所有功能，而且还提供了迭代器的算数，跳跃等
  * 示例：deque vector string迭代器就是这样

### 1.STL容器的选取原则

关乎到元素排序情况，迭代器相关，元素布局，C的兼容性，查找速度，事务语义，异常行为，效率容器复杂度

* 需要在任意位置插入元素，那么就需要序列容器
* 如果不关心容器中的元素是如何排序，那么哈希容器是一个可选择的方案（哈希容器本身不属于C++标准的一部分）
* 发生元素的插入或者删除时，尽量避免连续内存的容器
* 容器中数据的布局是否需要跟C兼容，如果需要的话，那么只能使用vector
* 元素的查找速度是否很关键，如果是的话，那么就要选用哈希容器
* 容器内部使用了引入计数reference counting技术是否介意，如果是的话，就要避免使用string与rope，因为这两种容器都使用引用计数去实现，string可以替换为vector&lt;char&gt;
* 插入和删除操作需要事务语义吗
  * 也就是说如果插入或者删除操作失败的话，需要具备会滚能力吗
  * 如果需要事务语义，那么就要使用基于节点的容器，例如list
  * 那些需要编写异常安全的代码，事务语义很重要
* 是否需要尽量减少迭代器/引用/指针失效次数
  * 基于节点的容器，插入和删除操作是不会使迭代器失效的
* deque的一个特殊性
  * 作为序列容器，迭代器是随机访问类型，而且只要没有删除操作发生，且插入操作只发生在容器的末尾，迭代器可能会变成无效，但是指针和引用不会无效
  * 只有deque具有这样的特性：迭代器无效，但是指针和引用不会无效

### 2.不要编写容器类型相关的代码

* 只有序列容器支持pushfront，pushback操作
* 只有关联容器支持count，lowbound，equalrange等操作
* insert，erase这些操作针对于不同的容器也有不同的细节实现

### 3.STL容器取出和存入对象都是拷贝，剥离问题的解决

从容器中取出一个对象，不是容器中保存的那份，而是一个拷贝

同样存入一个对象，也是把对象的拷贝存入进去，而不是直接保存原本的对象

（这些基于拷贝构造函数 与 拷贝赋值运算符来实现）

#### 继承类中存在一个问题：剥离

如果对象是存放基类对象的，但是却向其中插入派生类对象，将会导致派生类在拷贝进入容器时，其特有的部分将会丢失（这显然是一种错误的现象）

```cpp
vector<Widget> vec;
SpecialWidget sw;
vec.push_back(sw);
```

如果希望保存的对象依然是派生类的对象本体（没有被剥离掉特性）那么应该把这个容器构造成存放指针：

```cpp
vector<Widget*>vec;
SpecialWidget* sw = new SpecialWidget();
vec.push_back(sw);
```

不过指针可能还会造成一些其他的问题，智能指针也是一个比较好的选择

### 4.调用empty\(\)而不是实际检查size\(\)为0

原因：empty\(\)对于所有容器都是常数的时间操作，但是size\(\)对于一些容器例如list，将会耗费线性时间

list的实现是一种权衡trade-off在size\(\)与splice\(\)

* splice是list独有的一种操作：链接，将list2的某个节点之后的整个部分，链接到list1后面（如果不遍历数一下是不知道有多少个元素链接过来，所以size\(\)就是线性时间数出来有多少个元素
* 选择把splice实现为常数时间，然而舍弃了size\(\)的效率

### 5.学会使用区间操作的成员函数，而不是for循环对单个元素操作

示例1：assign，给定v1 与v2 两个vector，希望v1的内容与v2的后半段相同

```cpp
v1.assign(v2.begin() + v2.size() / 2, v2.end());
```

assign就是赋值的成员函数，只需要区间操作一行代码就可以搞定

示例2: int数组拷贝到一个vector的前端

```cpp
int data[nums];
vector<int> vec;
vec.insert(vec.begin(), data, data + nums);
```

insert函数就可以直接完成从数组拷贝元素到vector中，非常简单高效

区间创建，区间插入，区间删除，区间赋值

```cpp
//创建
vector<int> vec2(vec1.begin(), vec1.end());
//插入
void container::insert(iterator position,
                InputIterator begin,
                InputIterator end);
vec2.insert(vec2.begin(), vec1.begin()+vec1.size()/2, vec1.end());
//删除：顺序容器可以返回下一个迭代器，关联容器没有返回值
iterator container::erase(iterator begin, iterator end);
void container::erase(iterator begin, iterator end);
//赋值
void container::assign(iterator begin, iterator end);
```

### 7. 如果容器中包含了通过new操作创建的指针，切记容器对象析构前要将指针delete掉

```cpp
vector<Widget*>vec;
for(int i = 0; i < N; i++)
    vec.push_back(new Widget);
//析构前delete
for(auto it=vec.begin();it!=vec.end();++it)
    delete *it;
```

### 8.切勿创建包含auto\_ptr的容器对象

### 9.删除元素注意事项

序列容器

* 如果容器是vector，string，deque，则使用erase-remove习惯用法

  \(对vector来说, remove\(\)函数并不是真正的删除，要想真正删除元素则可以使用erase\)

```cpp
c.erase(1963); //只删除一个元素，返回值指向已删除元素的下一个位置 
c.erase(remove(c.begin(), c.end(), 1963), c.end()) //删除所有值为1963的元素
```

* 如果容器是list，则使用remove

```cpp
c.remove(val); //list可以使用remove来删除
```

* **迭代器更新：**标准序列容器，写一个循环来遍历容器中的元素时（迭代器）每次调用erase，它的返回值就是表示更新的迭代器

**关联容器**

* 容器是关联容器，使用erase
* **迭代器更新：**标准关联容器，当把迭代器传给erase时，要对迭代器做后缀递增！因为迭代器没有返回值

```cpp
c.erase(val)
```

### 12.STL的线程安全性

STL的线程安全性其实跟具体的实现有关，一般尽量都能保证同步读的时候都可以共享，修改的时候加锁尽量减少锁粒度，这方面也有很多种不同的实现，github一些库会尝试实现线程安全的STL

## vector和string

### 13.动态分配数组new T\[...\]的时候，考虑用vector和string来代替

因为当vector与string被析构时，它们析构函数会自动析构容器中的元素并释放包含这些元素的内存

### 14.使用reserve来避免不必要的内存重新分配realloc

realloc包含4个部分：

* ①分配当前容量的某个倍数的内存
* ②拷贝元素从旧的内存到新的内存
* ③析构掉旧的内存中的对象
* ④释放旧内存

reserve成员函数能把重新分配的次数讲到最低

### 16.把vector与string传给旧的C API

c++要求vector中的元素存储在连续的内存中

对于vector, 想利用C api访问数组第一个元素的指针 `&v[0]`

对于string, 访问string第一个字符的指针，也就是char\*  `s.c_str()`

### 17.使用“swap技巧”来shrink to fit，除去多余的capacity容量

会将原本容器压缩至适当大小，紧缩capacity\(\)为size\(\)

```cpp
vector<T>contents.swap(contents);

string s;
string (s).swap(s);
```

### 18.避免使用vector&lt;bool&gt;

* 因为首先它不是一个STL容器，不满足STL容器的要求
* 其次也并不存储bool

可以使用deque&lt;bool&gt; bitset来代替这个vector&lt;bool&gt;

## 关联容器

### 20.包含指针的关联容器 需要指定 比较函数（排序方式）

假设有一个包含string\*指针的set，把一些元素插入集合中，元素的有序完全是**按照指针来排序的，并不是按照string数值来排序**

```cpp
set<string*> ssp;
ssp.insert(new string("Anteater");
ssp.insert(new string("Tom");
ssp.insert(new string("Hello");
ssp.insert(new string("Color");
```

**解决问题：自己定义排序方式，使string\*指针在集合中按字符串的值排序**

**\(注意要用比较类！用比较函数的话不能通过编译）**

```cpp
struct StringPtrLess{
    bool operator() (const string* ps1, const string* ps2) const{
        return *ps1 < *ps2;
    }
};
set<string*, StringPtrLess> StringPtrSet;
```

### 21.总是让比较函数在等值情况下返回false

如果对于相等的比较返回true的话，就回修改到原来的比较对象，这个是不希望看到的

不然它们会破坏所有的标准关联容器，不管它们是否允许存储重复的值

### 22.切勿直接修改set或multiset中的键

因为set与multiset按照一定顺序存放自己的元素，理论上是可以随时改变其中的元素，也就是改变了其中的键

**这个条款并没有提到map与multimap，因为本身对于map、multimap直接去修改key是行不通的（）类似于const的约束），不能通过编译，但是set与multiset是有可能的修改，所以尽量不要修改set/multiset中的键**

**示例：存放雇员信息的set容器，因为是用其中的ID进行集合排序的，所以认为ID就是这个set中的键**

```cpp
class MyClass{
public:
    ...
    const string& name() const;
    int idNumber() const;
    ...
};
struct IDNumberLess{
    bool operator() (const Employee& lhs, const Employee& rhs) const{
        return lhs.idNumber() < rhs.idNumber();
    }
};
set<Employee, IDNumberLess> se; //se是按照ID号进行排序的雇员集合
```

**避免修改的解决办法就是拷贝一份，删除后再添加**

### 23.考虑用排序的vector替代关联容器标准

**标准关联容器**通常被实现为平衡的二叉查找树，对实现有一些优化

**主要应对的场景：插入，删除，查找的操作混合在一起**

**但也存在这样的应用场景：**

* ①设置阶段：创建一个新的容器，插入大量元素
* ②查找阶段：主要是查找操作，几乎没有插入与删除操作
* ③重组阶段：主要是改变数据结构的内容，可能是删除原有的元素，然后插入大量的新元素

**这种情况下，往往一个排序的vector容器比关联容器存在更好的性能**

**数量掌握vector&lt;pair&gt;与map的转换也是很有必要的，因为对map中的value进行排序往往需要依赖vector来完成**

### **24.当效率很重要的时候，请在map::operator\[\]与map::insert之间谨慎做出选择**

```cpp
class Widget{
public:
    Widget();
    Widget(double weight);
    Widget& operator=(double weight);
    ...
};

map<int, Widget>m;
m[1] = 1.50;
m[2] = 3.67;
```

**operator\[\]** 具体的工作方式：

对于map&lt;K,V&gt;m, m\[k\] = v;

* operator\[\]返回一个引用，它指向与k相关联的值对象，然后v被赋给该引用所指向的对象
* 如果键k已经有了相关联的值，那么就更新原本的值
* 如果键k没有相关联的值，那么就用值的类型默认构造函数创建一个新的对象，然后返回一个指向改新对象的引用

所以，之所以**相比于insert会降低性能的缘故就是：需要先默认构造一个，然后再赋值给他新值**

**insert操作：直接使用我们需要的值构造一个对象（少了一个赋值操作）**

## 函数

### 32.如果确实需要删除元素，则需要在remove这一类算法之后调用erase

**remove的本质：把删除元素的位置，被不删除元素填充（有点像从vector移除元素那种双指针移除法），但是size没有变**

**返回的一个迭代器指向最后一个不用被删除的元素**

需要一对迭代器来指定所有进行操作的元素区间，它并不接受容器作为参数

所以remove并没有办法从迭代器中推断出容器类型**，所以remove不可能删除元素**

**示例：**

![](../.gitbook/assets/wu-biao-ti-%20%2813%29.png)

![](../.gitbook/assets/wu-biao-ti-%20%2812%29.png)

**remove一般配合erase来使用**

```cpp
v.erase(remove(v.begin(), v.end(), 99), v.end());
```

### 33.对包含new指针的容器使用remove算法一定要注意：很可能内存泄漏

主要因为remove只是让新元素覆盖到原来删除元素的位置

![](../.gitbook/assets/wu-biao-ti-%20%2816%29.png)

本质：**“要被删除”的指针 已经被那些“不会被删除”的指针覆盖了** 

**没有任何指针再指向Widget B,Widget C,所以永远不会被删除**

**解决：智能指针 管理容器元素 可以去remove**

\*\*\*\*

### 44.容器的成员函数优先于同名的算法

STL容器 提供了 count find lowerbound upperbound equalrange

list提供了remove removeif unique sort merge reverse

使用对应容器时，最好应该使用容器内部实现的成员函数，往往效率更高

### 46.考虑使用函数对象而不是函数作为STL算法的参数

像例如对vector降序排序, 传入一个**函数对象**，因为比传入函数**要快得多**

```cpp
vector<double>v;
sort(v.begin(), v.end(), greater<double>());
```

传入函数的代码

```cpp
inline bool doubleGreater(double d1, double d2){
    return d1 > d2;
}
sort(v.begin(), v.end(), doubleGreater);
```



