# this

* 一个对象的this指针并不是对象本身的一部分，不会影响sizeof\(对象\)的结果。
* 在类的非静态成员函数中返回类对象本身的时候，直接使用 return \*this。

#### delete this 合法吗？

合法，但：

1. 必须保证 this 对象是通过 `new`（不是 `new[]`、不是 placement new、不是栈上、不是全局、不是其他对象成员）分配的
2. 必须保证调用 `delete this` 的成员函数是最后一个调用 this 的成员函数
3. 必须保证成员函数的 `delete this` 后面没有调用 this 了,且没有再使用该对象了

