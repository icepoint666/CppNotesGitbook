# volatile shared\_ptr 不能与 nullptr进行比较

参考：[https://stackoverflow.com/questions/36334749/in-c-is-it-possible-to-compare-volatile-shared-ptr-to-nullptr](https://stackoverflow.com/questions/36334749/in-c-is-it-possible-to-compare-volatile-shared-ptr-to-nullptr)



```cpp
auto main() -> int {
    volatile shared_ptr<int> a;
    if (a == nullptr) // fails
        ; // empty
    if (a) // fails
        ; // empty
}
```

**解决：只能建议使用传统new的方式创建对象，而不是shared\_ptr**

**volatile T a = new T\(\);**

