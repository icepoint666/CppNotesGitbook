# volatile T类型返回时不等于T&

**错误示例：（会报错**binding reference of type ‘**A&**’ to ‘**volatile A**’ discards qualifiers）

```cpp
template <typename T>
T& Singleton<T>::Instance() {
    volatile static T* instance;
    static std::mutex m;
    if(!instance){
        std::lock_guard<std::mutex> lock(m);
        if(!instance){
            instance = new T();
        }
    }
    return *instance;
}
```

**解决**：**通过const cast可以去掉volatile修饰**



