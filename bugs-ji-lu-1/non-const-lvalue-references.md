# Non const lvalue references

临时值是不允许绑定到&上，因为**潜在类型转换导致的bug坑**

```text
int a;
double &m = a;
```

如果a是int类型，因为需要类型转换，这时候需要先转换成临时值，但是临时值不能绑定到左值引用上，所以会报错



