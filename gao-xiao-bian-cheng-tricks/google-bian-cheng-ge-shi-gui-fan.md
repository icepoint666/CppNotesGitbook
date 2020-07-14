# Google编程格式规范

参考：Google开源项目风格指南（笔记）

[https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/)

* 命名约定
* 注释
* 格式

## 一、命名约定

### 通用命名规则

尽可能使用描述性的命名, 别心疼空间, 毕竟相比之下让代码易于新读者理解更重要. 不要用只有项目开发者能理解的缩写, 也不要通过砍掉几个字母来缩写单词.

```text
int price_count_reader;    // 无缩写
int num_errors;            // "num" 是一个常见的写法
int num_dns_connections;   // 人人都知道 "DNS" 是什么


int n;                     // 毫无意义.
int nerr;                  // 含糊不清的缩写.
int n_comp_conns;          // 含糊不清的缩写.
int wgc_connections;       // 只有贵团队知道是什么意思.
int pc_reader;             // "pc" 有太多可能的解释了.
int cstmr_id;              // 删减了若干字母.
```

注意, 一些特定的广为人知的缩写是允许的, 例如用 `i` 表示迭代变量和用 `T` 表示模板参数.

### 文件命名

文件名要全部小写, 可以包含下划线 \(`_`\) 或连字符 \(`-`\), 依照项目的约定. 如果没有约定, 那么 “`_`” 更好.

可接受的文件命名示例:

* `my_useful_class.cc`
* `my-useful-class.cc`
* `myusefulclass.cc`
* `myusefulclass_test.cc`
* C++ 文件要以 `.cc` 结尾, 头文件以 `.h` 结尾. 专门插入文本的文件则以 `.inc` 结尾
* 不要使用已经存在于 `/usr/include` 下的文件名 \(Yang.Y 注: 即编译器搜索系统头文件的路径\), 如 `db.h`.

### 类型命名

类型名称的每个单词首字母均大写, 不包含下划线: `MyExcitingClass`, `MyExcitingEnum`.

### 变量命名

变量 \(包括函数参数\) 和数据成员名一律小写, 单词之间用下划线连接. **类的成员变量以下划线结尾, 但结构体的就不用**, 如: `a_local_variable`, `a_struct_data_member`, `a_class_data_member`

```cpp
class TableInfo {
  ...
 private:
  string table_name_;  // 好 - 后加下划线.
  string tablename_;   // 好.
  static Pool<TableInfo>* pool_;  // 好.
};
```

### 常量命名

声明为 `constexpr` 或 `const` 的变量, 或在程序运行期间其值始终保持不变的, 命名时以 “k” 开头, 大小写混合. 例如:

```text
const int kDaysInAWeek = 7;
```

### 函数命名

一般来说, 函数名的每个单词首字母大写 \(即 “驼峰变量名” 或 “帕斯卡变量名”\), 没有下划线. 对于首字母缩写的单词, 更倾向于将它们视作一个单词进行首字母大写 \(例如, 写作 `StartRpc()` 而非 `StartRPC()`\).

```text
AddTableEntry()
DeleteUrl()
OpenFileOrDie()
```

取值和设值函数的命名与变量一致. 一般来说它们的名称与实际的成员变量对应, 但并不强制要求. 例如 `int count()` 与 `void set_count(int count)`.

### 命名空间命名

以**小写字母**命名. 最高级命名空间的名字取决于项目名称. 要注意避免嵌套命名空间的名字之间和常见的顶级命名空间的名字之间发生冲突.

### 枚举命名

枚举的命名应当和 [常量](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/#constant-names) 或 [宏](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/#macro-names) 一致: `kEnumName` 或是 `ENUM_NAME`.

```cpp
enum UrlTableErrors {
    kOK = 0,
    kErrorOutOfMemory,
    kErrorMalformedInput,
};

enum AlternateUrlTableErrors {
    OK = 0,
    OUT_OF_MEMORY = 1,
    MALFORMED_INPUT = 2,
}; //一般建议后者
```

### 宏命名

如果你一定要用, 像这样命名: `MY_MACRO_THAT_SCARES_SMALL_CHILDREN`.

## 二、注释

`//` 或 `/* */` 都可以; 但 `//` _更_ 常用. 要在如何注释及注释风格上确保统一.

### 文件注释

在每一个文件开头加入版权公告.

每个文件都应该包含许可证引用. 为项目选择合适的许可证版本.\(比如, Apache 2.0, BSD, LGPL, GPL\)

如果你对原始作者的文件做了重大修改, 请考虑删除原作者信息.

### 类注释

每个类的定义都要附带一份注释, 描述类的功能和用法, 除非它的功能相当明显.

如果类的声明和定义分开了\(例如分别放在了 `.h` 和 `.cc` 文件中\), 此时, 描述类用法的注释应当和接口定义放在一起, 描述类的操作和实现的注释应当和实现放在一起.

### 函数注释

函数声明处的注释描述函数功能; 定义处的注释描述函数实现.

函数声明处注释的内容:

* 函数的输入输出.
* 对类成员函数而言: 函数调用期间对象是否需要保持引用参数, 是否会释放这些参数.
* 函数是否分配了必须由调用者释放的空间.
* 参数是否可以为空指针.
* 是否存在函数使用上的性能隐患.
* 如果函数是可重入的, 其同步前提是什么?

注释函数重载时, 注释的重点应该是函数中被重载的部分, 而不是简单的重复被重载的函数的注释. 多数情况下, 函数重载不需要额外的文档, 因此也没有必要加上注释.

注释构造/析构函数时, 切记读代码的人知道构造/析构函数的功能, 所以 “销毁这一对象” 这样的注释是没有意义的. 你应当注明的是注明构造函数对参数做了什么

### 变量注释

通常变量名本身足以很好说明变量用途. 某些情况下, 也需要额外的注释说明.

```text
// The total number of tests cases that we run through in this regression test.
const int kNumTestCases = 6;
```

### 不允许的行为

不要描述显而易见的现象, _永远不要_ 用自然语言翻译代码作为注释, 除非即使对深入理解 C++ 的读者来说代码的行为都是不明显的. 要假设读代码的人 C++ 水平比你高, 即便他/她可能不知道你的用意:

你所提供的注释应当解释代码 _为什么_ 要这么做和代码的目的, 或者最好是让代码自文档化.

比较这样的注释:

```text
// Find the element in the vector.  <-- 差: 这太明显了!
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
```

和这样的注释:

```text
// Process "element" unless it was already processed.
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
```

### TODO 注释

对那些临时的, 短期的解决方案, 或已经够好但仍不完美的代码使用 `TODO` 注释.

`TODO` 注释要使用全大写的字符串 `TODO`, 在随后的圆括号里写上你的名字, 邮件地址, bug ID, 或其它身份标识和与这一 `TODO` 相关的 issue. 主要目的是让添加注释的人 \(也是可以请求提供更多细节的人\) 可根据规范的 `TODO` 格式进行查找. 添加 `TODO` 注释并不意味着你要自己来修正, 因此当你加上带有姓名的 `TODO` 时, 一般都是写上自己的名字.

```text
// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke) change this to use relations.
// TODO(bug 12345): remove the "Last visitors" feature
```

如果加 `TODO` 是为了在 “将来某一天做某事”, 可以附上一个非常明确的时间 “Fix by November 2005”\), 或者一个明确的事项 \(“Remove this code when all clients can handle XML responses.”\).

## 三、格式

每一行代码字符数不超过 80.

带有命令示例或 URL 的行可以超过 80 个字符.

包含长路径的 `#include` 语句可以超出80列.

### 非 ASCII 字符

尽量不使用非 ASCII 字符, 使用时必须使用 UTF-8 编码.

### 空格还是制表位

只使用空格, 每次缩进 2 个空格.

**说明**

我们使用空格缩进. 不要在代码中使用制表符. 你应该设置编辑器将制表符转为空格.

### 函数声明与定义

函数看上去像这样:

```text
ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
  DoSomething();
  ...
}
```

如果同一行文本太多, 放不下所有参数:

```text
ReturnType ClassName::ReallyLongFunctionName(Type par_name1, Type par_name2,
                                             Type par_name3) {
  DoSomething();
  ...
}
```

甚至连第一个参数都放不下:

```text
ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
    Type par_name1,  // 4 space indent
    Type par_name2,
    Type par_name3) {
  DoSomething();  // 2 space indent
  ...
}
```

注意以下几点:

* 使用好的参数名.
* 只有在参数未被使用或者其用途非常明显时, 才能省略参数名.
* 如果返回类型和函数名在一行放不下, 分行.
* 如果返回类型与函数声明或定义分行了, 不要缩进.
* 左圆括号总是和函数名在同一行.
* 函数名和左圆括号间永远没有空格.
* 圆括号与参数间没有空格.
* 左大括号总在最后一个参数同一行的末尾处, 不另起新行.
* 右大括号总是单独位于函数最后一行, 或者与左大括号同一行.
* 右圆括号和左大括号间总是有一个空格.
* 所有形参应尽可能对齐.
* 缺省缩进为 2 个空格.
* 换行后的参数保持 4 个空格的缩进.

未被使用的参数, 或者根据上下文很容易看出其用途的参数, 可以省略参数名:

```text
class Foo {
 public:
  Foo(Foo&&);
  Foo(const Foo&);
  Foo& operator=(Foo&&);
  Foo& operator=(const Foo&);
};
```

### 函数调用

要么一行写完函数调用, 要么在圆括号里对参数分行, 要么参数另起一行且缩进四格

如果同一行放不下, 可断为多行, 后面每一行都和第一个实参对齐, 左圆括号后和右圆括号前不要留空格：

```text
bool retval = DoSomething(averyveryveryverylongargument1,
                          argument2, argument3);
```

参数也可以放在次行, 缩进四格：

```text
if (...) {
  ...
  ...
  if (...) {
    DoSomething(
        argument1, argument2,  // 4 空格缩进
        argument3, argument4);
  }
```

### 条件语句

倾向于不在圆括号内使用空格. 关键字 `if` 和 `else` 另起一行.

```text
if (condition) {  // 圆括号里没有空格.
  ...  // 2 空格缩进.
} else if (...) {  // else 与 if 的右括号同一行.
  ...
} else {
  ...
}
```

```text
if(condition)     // 差 - IF 后面没空格.
if (condition){   // 差 - { 前面没空格.
if(condition){    // 变本加厉地差.
if (condition) {  // 好 - IF 和 { 都与空格紧邻.
```

如果能增强可读性, 简短的条件语句允许写在同一行. 只有当语句简单并且没有使用 `else` 子句时使用:

```text
if (x == kFoo) return new Foo();
if (x == kBar) return new Bar();
```

如果语句有 `else` 分支则不允许:

```text
// 不允许 - 当有 ELSE 分支时 IF 块却写在同一行
if (x) DoThis();
else DoThat();
```

### 循环

在单语句循环里, 括号可用可不用：

```text
for (int i = 0; i < kSomeNumber; ++i)
  printf("I love you\n");

for (int i = 0; i < kSomeNumber; ++i) {
  printf("I take it back\n");
}
```

空循环体应使用 `{}` 或 `continue`, 而不是一个简单的分号.

```text
while (condition) {
  // 反复循环直到条件失效.
}
for (int i = 0; i < kSomeNumber; ++i) {}  // 可 - 空循环体.
while (condition) continue;  // 可 - contunue 表明没有逻辑.
```

### 指针和引用表达式

下面是指针和引用表达式的正确使用范例:

```text
x = *p;
p = &x;
x = r.y;
x = r->y;
```

注意:

* 在访问成员时, 句点或箭头前后没有空格.
* 指针操作符 `*` 或 `&` 后没有空格.

在声明指针变量或参数时, 星号与类型或变量名紧挨都可以:

```text
// 好, 空格前置.
char *c;
const string &str;

// 好, 空格后置.
char* c;
const string& str;
int x, *y;  // 不允许 - 在多重声明中不能使用 & 或 *
char * c;  // 差 - * 两边都有空格
const string & str;  // 差 - & 两边都有空格.
```

### 布尔表达式

如果一个布尔表达式超过 [标准行宽](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/formatting/#line-length), 断行方式要统一一下.

**说明**

下例中, 逻辑与 \(`&&`\) 操作符总位于行尾:

```text
if (this_one_thing > this_other_thing &&
    a_third_thing == a_fourth_thing &&
    yet_another && last_one) {
  ...
}
```

注意, 上例的逻辑与 \(`&&`\) 操作符均位于行尾.

### 函数返回值

不要在 `return` 表达式里加上非必须的圆括号.

### 预处理指令

预处理指令不要缩进, 从行首开始.

### 类格式

访问控制块的声明依次序是 `public:`, `protected:`, `private:`, 每个都缩进 1 个空格.

**说明**

类声明 \(下面的代码中缺少注释, 参考 [类注释](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/comments/#class-comments)\) 的基本格式如下:

```text
class MyClass : public OtherClass {
 public:      // 注意有一个空格的缩进
  MyClass();  // 标准的两空格缩进
  explicit MyClass(int var);
  ~MyClass() {}

  void SomeFunction();
  void SomeFunctionThatDoesNothing() {
  }

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

 private:
  bool SomeInternalFunction();

  int some_var_;
  int some_other_var_;
};
```

注意事项:

* 所有基类名应在 80 列限制下尽量与子类名放在同一行.
* 关键词 `public:`, `protected:`, `private:` 要缩进 1 个空格.
* 除第一个关键词 \(一般是 `public`\) 外, 其他关键词前要空一行. 如果类比较小的话也可以不空.
* 这些关键词后不要保留空行.
* `public` 放在最前面, 然后是 `protected`, 最后是 `private`.
* 关于声明顺序的规则请参考 [声明顺序](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/classes/#declaration-order) 一节.

### 命名空间格式化

命名空间内容不缩进.

**说明**

[命名空间](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/scoping/#namespaces) 不要增加额外的缩进层次, 例如:

```text
namespace {

void foo() {  // 正确. 命名空间内没有额外的缩进.
  ...
}

}  // namespace
```

不要在命名空间内缩进:

```text
namespace {

  // 错, 缩进多余了.
  void foo() {
    ...
  }

}  // namespace
```

声明嵌套命名空间时, 每个命名空间都独立成行.

```text
namespace foo {
namespace bar {
```

### 留白

**循环和条件语句**

```text
if (b) {          // if 条件语句和循环语句关键字后均有空格.
} else {          // else 前后有空格.
}
while (test) {}   // 圆括号内部不紧邻空格.
switch (i) {
for (int i = 0; i < 5; ++i) {
switch ( i ) {    // 循环和条件语句的圆括号里可以与空格紧邻.
if ( test ) {     // 圆括号, 但这很少见. 总之要一致.
for ( int i = 0; i < 5; ++i ) {
for ( ; i < 5 ; ++i) {  // 循环里内 ; 后恒有空格, ;  前可以加个空格.
switch (i) {
  case 1:         // switch case 的冒号前无空格.
    ...
  case 2: break;  // 如果冒号有代码, 加个空格.
```

**操作符**

```text
// 赋值运算符前后总是有空格.
x = 0;

// 其它二元操作符也前后恒有空格, 不过对于表达式的子式可以不加空格.
// 圆括号内部没有紧邻空格.
v = w * x + y / z;
v = w*x + y/z;
v = w * (x + z);

// 在参数和一元操作符之间不加空格.
x = -5;
++x;
if (x && !y)
  ...
```

**模板和转换**

```text
// 尖括号(< and >) 不与空格紧邻, < 前没有空格, > 和 ( 之间也没有.
vector<string> x;
y = static_cast<char*>(x);

// 在类型与指针操作符之间留空格也可以, 但要保持一致.
vector<char *> x;
```

