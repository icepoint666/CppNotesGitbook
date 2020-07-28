# find，remove，erase

可以通过这几个函数，**仅一行代码实现过滤string/vector的一些内容**

**示例：过滤字符串的空格**

```cpp
#include <string> //erase
#include <algorithm> //remove
#include <cctype> //issapce
std::string str1 = "Text with some   spaces";
str1.erase(std::remove(str1.begin(), str1.end(), ' '), str1.end());
std::cout << str1 << '\n'; //Textwithsomespaces

std::string str2 = "Text\n with\tsome \t  whitespaces\n\n";
str2.erase(std::remove_if(str2.begin(), 
                          str2.end(),
                          [](unsigned char x){return std::isspace(x);}),
           str2.end());
std::cout << str2 << '\n'; //Textwithsomewhitespaces
```

**示例：过滤不符合条件的值**

```cpp
std::vector<int> c = {1, 2, 3, 4, 5, 6, 7};
int x = 5;
c.erase(std::remove_if(c.begin(), c.end(), [x](int n) { return n < x; }), c.end());
```

下面主要介绍一下原理

### STL容器erase

**作用：移除迭代器指向的元素，erase很适合配合std::remove使用，总的来说还是O\(n\)复杂度**

在string, vector等各种STL容器都存在这个操作

其中包含两种变体：移除一个元素，移除从st迭代器到ed迭代器之间的元素

```cpp
vector<string> e = {"a","b","c","d","e","f","g"};
e.erase(e.begin());//移除“a”
//另外的用法
e.erase(e.begin()-1,e.end()+1); //移除b","c","d","e","f"
```

### std::remove 以及 std::remove\_if

\#include&lt;algorithm&gt;

**remove本质：**

* **找到符合值的第一个迭代器（用find）**
* **然后需要修改这个迭代器的下一个迭代器指针，需要是指向下一个的符合值的迭代remove并不是单纯返回该迭代器的下一个元素（其实也不一定需要这个操作？）**
* **将第一个迭代器返回给erase，然后erase会移除，然后erase会迭代，自动指向下一个要移除对象（这个是remove的功劳）**

```cpp
//简易实现版本
template< class ForwardIt, class T >
ForwardIt remove(ForwardIt first, ForwardIt last, const T& value)
{
    first = std::find(first, last, value);
    if (first != last)
        for(ForwardIt i = first; ++i != last; )
            if (!(*i == value))
                *first++ = std::move(*i);
    return first;
}
```

**remove\_if：与remove差不多, 不同的就是不是找符合的值，而是找符合先决条件的元素**

（这个先决条件很多时候是一个bool类型lambda函数）

```cpp
template<class ForwardIt, class UnaryPredicate>
ForwardIt remove_if(ForwardIt first, ForwardIt last, UnaryPredicate p)
{
    first = std::find_if(first, last, p);
    if (first != last)
        for(ForwardIt i = first; ++i != last; )
            if (!p(*i))
                *first++ = std::move(*i);
    return first;
}
```

### std::find 以及 std::find\_if

单纯**O\(n\)的暴力查找**，而且**只返回找到的第一个元素的迭代器**

```cpp
# find
template<class InputIterator, class T>
  InputIterator find (InputIterator first, InputIterator last, const T& val)
{
  while (first!=last) {
    if (*first==val) return first;
    ++first;
  }
  return last;
}

# find_if
template<class InputIterator, class UnaryPredicate>
  InputIterator find_if (InputIterator first, InputIterator last, UnaryPredicate pred)
{
  while (first!=last) {
    if (pred(*first)) return first;
    ++first;
  }
  return last;
}
```

