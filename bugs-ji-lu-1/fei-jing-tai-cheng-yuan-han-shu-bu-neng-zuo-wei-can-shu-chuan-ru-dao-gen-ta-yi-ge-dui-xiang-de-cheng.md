# 非静态成员函数不能作为参数传入到跟它一个对象的成员函数中

例如对于signal handle场景

```cpp
HttpServer::signal_handler();

HttpServer::add_siganl(int sig, void(handler)(int));

add_signal(sig, signal_handler());
// 会报错：invalid use of non-static member function
```

