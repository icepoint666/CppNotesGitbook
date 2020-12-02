# memcpy, memcmp, memset用法

**\#include&lt;string.h&gt;**

### memcpy

**内存复制**

* **str1** -- 指向用于存储复制内容的目标数组，类型强制转换为 void\* 指针。
* **str2** -- 指向要复制的数据源，类型强制转换为 void\* 指针。
* **n** -- 要被复制的字节数。

该值返回一个指向存储区 str1 的指针 \(或者不返回）

```cpp
void *memcpy(void *str1, const void *str2, size_t n)
```

### memcmp

**比较两个内存块**，是否相等，或者谁大谁小

* **str1** -- 指向内存块的指针。
* **str2** -- 指向内存块的指针。
* **n** -- 要被比较的字节数。

返回：

* 如果返回值 &lt; 0，则表示 str1 小于 str2。
* 如果返回值 &gt; 0，则表示 str2 小于 str1。
* 如果返回值 = 0，则表示 str1 等于 str2。

```cpp
int memcmp ( const void * ptr1, const void * ptr2, size_t num );

<0	the first byte that does not match in both memory blocks 
    has a lower value in ptr1 than in ptr2 (if evaluated as unsigned char values)
0	the contents of both memory blocks are equal
>0	the first byte that does not match in both memory blocks 
    has a greater value in ptr1 than in ptr2 (if evaluated as unsigned char values)
```

### memset

 复制字符 **c**（一个无符号字符）到参数 **str** 所指向的字符串的前 **n** 个字符。

* **str** -- 指向要填充的内存块。
* **c** -- 要被设置的值。该值以 int 形式传递，但是函数在填充内存块时是使用该值的无符号字符形式。
* **n** -- 要被设置为该值的字符数。

该值返回一个指向存储区 str 的指针 \(或者不返回）

```cpp
void *memset(void *str, int c, size_t n)
```

