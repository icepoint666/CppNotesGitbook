# C++ char\* 转换为 std::string

string构造函数

```cpp
char* s = const_cast<char*>("abv");
std::string ss(s);
std::cout << ss <<std::endl;
```

