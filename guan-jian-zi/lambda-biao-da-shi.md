# lambda表达式

lambda没有给语言带来新的表达能力，只是一种函数的便携表示方法

示例：

```cpp
std::find_if(container.begin(), container.end(), 
        [](int val){ return 0 < val && val < 10; }); // 本⾏⾼亮
```

### 避免使用默认捕获模式

C++11中有两种默认的捕获模式

* 按引用捕获：可能会带来悬空引⽤的问题
* 按值捕获：也没有解决悬空引用的问题

