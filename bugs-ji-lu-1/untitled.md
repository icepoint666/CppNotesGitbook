# vector中存储future变量

vector存放future变量，必须保证引用时

参考：[https://stackoverflow.com/questions/36596398/vector-of-future-in-c11](https://stackoverflow.com/questions/36596398/vector-of-future-in-c11)

示例：

```cpp
auto K = [=](double z){
vector<future<double>> VF;
for (double i : {1,2,3,4,5,6,7,8,9})
    VF.push_back(async(K,i));
```

**错误方式：会报错**

```cpp
 for_each(VF.begin(), VF.end(), [](future<double> x){cout << x.get() << " "; });
```

**正确方式：加引用**

```cpp
 for_each(VF.begin(), VF.end(), [](future<double> &x){cout << x.get() << " "; });
```

**或者用数组来存，也不会报错**

**（目前还不太清楚原因）**

