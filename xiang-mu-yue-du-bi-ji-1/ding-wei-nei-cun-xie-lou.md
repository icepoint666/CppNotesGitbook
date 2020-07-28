# 定位内存泄漏

### Linux定位内存泄漏

{% embed url="https://stackoverflow.com/questions/18455698/lightweight-memory-leak-debugging-on-linux" %}

### Windows定位内存泄漏

利用在DEBUG环境下检查

```cpp
#ifdef _WIN32
    #include <crtdbg.h>
    #ifdef _DEBUG

    #define new new(_NORMAL_BLOCK,__FILE__,__LINE__)
    #endif
#endif

#ifdef _WIN32
	_CrtSetDbgFlag(_CrtSetDbgFlag(_CRTDBG_REPORT_FLAG)|_CRTDBG_LEAK_CHECK_DF);
#endif
```

自动化，单元测试的时候配合这个代码使用

c++怎么检测内存泄露，怎么定位内存泄露？ - Chen Moore的回答 - 知乎 [https://www.zhihu.com/question/63946754/answer/214793614](https://www.zhihu.com/question/63946754/answer/214793614)

c++怎么检测内存泄露，怎么定位内存泄露？ - vczh的回答 - 知乎 [https://www.zhihu.com/question/63946754/answer/215606096](https://www.zhihu.com/question/63946754/answer/215606096)

