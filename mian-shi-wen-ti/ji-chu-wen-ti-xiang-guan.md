# 基础问题相关

### 1.为什么头文件里要加\#ifndef \#define

**\#ifndef是if not define 目的防止头文件被重复包含**

```cpp
#ifndef HEADERFILE_H
#define HEADERFILE_H
...
#endif
```

回答思路：

1. 一个源文件或者一个头文件 **\#include指令**是发生在**预编译阶段**
2. **预编译\(gcc -i \)**会把这个头文件的大段代码复制到预编译代码文件中

   举个例子：一个写的头文件有几百行，就放到预编译文件中

3. 假如源文件`#include a.h`和`#include b.h`，其中`a.h`也`#include b.h`

   不加\#ifndef 那么源文件预编译就会有两份b.h的预编译代码部分

   就是为了防止重复多余的包含

