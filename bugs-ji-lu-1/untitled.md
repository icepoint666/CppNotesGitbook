# vector中存储future变量

vector存放future变量，必须保证引用时

示例：

```cpp
auto K = [=](double z){
vector<future<double>> VF;
for (double i : {1,2,3,4,5,6,7,8,9})
    VF.push_back(async(K,i));
```

It worked successfully but when I tried to retrieve the values via a for\_each call I obtained a compilation error that I do not understand.

```text
 for_each(VF.begin(), VF.end(), [](future<double> x){cout << x.get() << " "; });
```

