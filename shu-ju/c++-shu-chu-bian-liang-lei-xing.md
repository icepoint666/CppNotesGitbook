# 输出变量类型

### 利用auto得到运算变量，想查看类型

使用typeinfo中的typeid\(\).name\(\)

```cpp
// Example program
#include <iostream>
#include <string>
#include <typeinfo>
using namespace std;

        
int main()
{
  string s1 = "a value", s2 = "ano";
  auto n = (s1 + s2);
  cout << typeid(n).name() <<endl;
}

//结果会输出Ss
```

### 可以用来判断变量类型

```cpp
cout << (typeid(t) == typeid(b)) << endl;
```

