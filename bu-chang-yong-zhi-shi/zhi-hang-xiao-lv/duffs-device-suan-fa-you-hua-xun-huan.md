# Duff's device算法优化循环

一个普通的循环是这样的样子

```cpp
for（int i=0; i< N; i++){
    do_something()l
}
```

一般来说这样的循环是没有什么特别的问题的，是不需要什么特别优化，但是在一些特别情况下，如果`do_something()`的单词操作特别快，这个循环结束判断语句`i<N`对整体性能影响是非常大的

比如内存拷贝函数就是这样的场景

```cpp
for(int i=0; i<N; i++){
    to[i] = from[i];
}
```

显然这里的`i < N`的检查在这里就非常影响性能，那么减少检查次数就可以解决这个问题

Duff's device 处理这个问题更加暴力，直接利用 C 语言的 `switch` 语句不加 `break` 会继续执行的特性来“削平”多余的项：

```cpp
n = (N + 7) / 8;
i = 0;

switch (N % 8) {
case 0:
  do {
    to[i] = from[i];
    i++;
    case 7: 
        to[i] = from[i];
        i++;
    case 6:
        to[i] = from[i];
        i++;
    case 5: 
        to[i] = from[i];
        i++;
    case 4: 
        to[i] = from[i];
        i++;
    case 3:
        to[i] = from[i];
        i++;
    case 2: 
        to[i] = from[i];
        i++;
    case 1:
        to[i] = from[i];
        i++;
  } while (i<N);
}
```

Duff's device 算法充分利用了 C 语言的特点来优化效率，这种暴力又实用的奇技淫巧甚至在 `switch` 是否应该显式 `break` 的争论中为支持者提供了论据。

但是事实上，Duff's device 的优化思想并没有和 `switch` 进行绑定，你也可以轻易的改写出一个不依赖 `switch` 的版本（但不保证效率）。

