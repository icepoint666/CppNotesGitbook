# 初始化对象方式错误，编译器可能会把它当作函数声明

```text
HttpServer server(); //本来是想构造类对象的，结果却被编译器误认为函数声明
```

报错：which is of non-class type

