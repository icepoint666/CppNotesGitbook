# C++中std::string转换为int类型

## 四种方法

* atoi
* strtol
* stoi
* 自定义（推荐记忆一下，利用stringstream）

### c风格：atoi与strtol

```cpp
string str = "16s";
int a = atoi(str.c_str());
int b = strtol(str.c_str(), nullptr, 10);
```

需要用到`c_str()`函数

特性：

* 不会报异常！：两个函数都是从字符串开始寻找数字或者正负号或者小数点,遇到非法字符终止。

  所以到上述s字符就不输出了，提前结束，也就是说当你的字符串不是数字的时候，或者小数点等非数字，不会报异常！直接输出0.

* strtol相比与atoi来说，支持多种进制转换,例如8进制等

### c++风格：stoi

**最大特点就是，异常！遇到不合理的转换会报异常！！**

```cpp
string str1 = "asq,";
//    int c = stoi(str1);    // 报异常
string str2 = "12312";
int c = stoi(str2);     // ok
```

### 掌握：利用stringstream实现string转Int

```cpp
#include <sstream> //stringstream

int stringToInt(const string &s) {
    int v;
    stringstream ss;
    ss << s;
    ss >> v;
    return v;
}
int main() {
    int i = stringToInt("2.3");
    cout<<i<<endl;
}
```

