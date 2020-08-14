# C++ double或者int转为string

**通过stringstream来实现**

```cpp
#include <iostream>
#include <string>
#include <sstream>

int main()
{
  std::string name;
  double a = 2.3;
  std::stringstream ss;
  ss << a;
  ss >> name;
  std::cout << name << std::endl; //输出字符串"2.3"
}

```

**iostream中int double数值输出后会变成字符串，应该里面也夹杂了stringstream类似实现的方式**

