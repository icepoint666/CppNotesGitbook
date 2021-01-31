# if-else有什么不好，分支预测

参考：[https://www.zhihu.com/question/441518636/answer/1701252133](https://www.zhihu.com/question/441518636/answer/1701252133)



作者：程序喵大人  
链接：https://www.zhihu.com/question/441518636/answer/1701252133  
来源：知乎  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。  
  


一段经典的代码，并统计它的执行时间：

```cpp
// test_predict.cc
#include <algorithm>
#include <ctime>
#include <iostream>

int main() {
    const unsigned ARRAY_SIZE = 50000;
    int data[ARRAY_SIZE];
    const unsigned DATA_STRIDE = 256;

    for (unsigned c = 0; c < ARRAY_SIZE; ++c) data[c] = std::rand() % DATA_STRIDE;

    std::sort(data, data + ARRAY_SIZE);

    {  // 测试部分
        clock_t start = clock();
        long long sum = 0;

        for (unsigned i = 0; i < 100000; ++i) {
            for (unsigned c = 0; c < ARRAY_SIZE; ++c) {
                if (data[c] >= 128) sum += data[c];
            }
        }

        double elapsedTime = static_cast<double>(clock() - start) / CLOCKS_PER_SEC;

        std::cout << elapsedTime << "\n";
        std::cout << "sum = " << sum << "\n";
    }
    return 0;
}
~/test$ g++ test_predict.cc ;./a.out
7.95312
sum = 480124300000
```

此程序的执行时间是7.9秒，如果把排序那一行代码注释掉，即

```text
// std::sort(data, data + ARRAY_SIZE);
```

结果为：

```text
~/test$ g++ test_predict.cc ;./a.out
24.2188
sum = 480124300000
```

改动后的程序执行时间变为了24秒。

### 指令流水线加载机制

### 分支预测

上一次预测选择分支A，那么下一次加载指令的时候也会去尝试加载分支A的指令

（所以这也就是为什么排序后会快很多了）

