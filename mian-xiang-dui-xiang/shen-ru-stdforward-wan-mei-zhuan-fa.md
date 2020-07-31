# 深入std::forward完美转发的实现

### 完美转发的意义

如果现在**模板函数**要**调用更底层的函数**，但是不知道输入参数是左值还是右值类型，那就完美原封不动的转发给下层函数，再去判断

```cpp
template<class T>
void func(T &&a) {
	
	return F(std::forward<A>(a));  //1
	
	return F(a);		//2
}
```

### forward实现原理

要分析forward实现原理，我们首先来看一下forward代码实现。由于之前已经有了分析[std::move](https://app.gitbook.com/@1023553676/s/c-notes/~/drafts/-MDYARX2oPGsadLMEqRk/mian-xiang-dui-xiang/shen-ru-stdmove-you-zhi-yin-yong)的基础，所以再来看forward代码应该不会太困难。

```cpp
……

template <typename T>
T&& forward(typename std::remove_reference<T>::type& param)
{
    return static_cast<T&&>(param);
}

template <typename T>
T&& forward(typename std::remove_reference<T>::type&& param)
{
    return static_cast<T&&>(param);
}

……
```

forward实现了两个模板函数，一个接收左值，另一个接收右值

std::forward模板函数对传入的参数进行强制类型转换，转换的目标类型符合引用折叠规则，因此左值参数最终转换后仍为左值，右值参数最终转成右值。

