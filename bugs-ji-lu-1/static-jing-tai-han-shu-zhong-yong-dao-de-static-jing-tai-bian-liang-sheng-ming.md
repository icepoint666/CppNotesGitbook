# static静态函数中用到的static静态变量声明

**错误做法**：声明在类实例对象中**（会报编译错误）**

```cpp
template <typename T>
class Singleton {
 public:
  static T& Instance();
  static T* instance; //！！
  static std::mutex m; //！！
  Singleton(const Singleton&) = delete;
  Singleton& operator=(const Singleton&) = delete;

 private:
  Singleton() = default;
  ~Singleton() = default;
};

template <typename T>
T& Singleton<T>::Instance() {
    static T* instance;
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

**正确做法**：声明在static函数中

static function调用时是不会初始化示例对象的

