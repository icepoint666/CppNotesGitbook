# 虚继承，以及作用是什么

From: [https://mp.weixin.qq.com/s/NyNHXIrEjG7vcAS8wZxU2Q](https://mp.weixin.qq.com/s/NyNHXIrEjG7vcAS8wZxU2Q)

**虚继承：**

虚继承是**解决 C++** [**多重继承**](https://www.jianshu.com/p/821fe2979a05)问题的**一种手段**

从不同途径继承来的同一基类，会在子类中存在多份拷贝。这将存在两个问题：第一，浪费存储空间；第二，存在二义性问题。

针对这种情况，C++ 提供**虚基类（virtual base class）的方法**，使得在继承间接共同基类时**只保留一份成员**。

**虚继承的作用是减少了对基类的重复，代价是增加了虚表指针的负担（增加了更多的需指针）**

**Divde1继承Base, Divide2继承Base， DIvide继承Divide1，Divide2内存分布：**

```cpp
class Divide1 size(20) :
   +-- -
   0 | {vbptr}
   4 | c
   +-- -
   +-- - (virtual base Base)
   8 | {vfptr}
  12 | a
  16 | b
   +-- -

  Divide1::$vbtable@:
   0 | 0
   1 | 8 (Divide1d(Divide1 + 0)Base)

  Divide1::$vftable@:
 | -8
   0 | &Divide1::run

  Divide1::run this adjustor: 8
  vbi:    class  offset o.vbptr  o.vbte fVtorDisp
              Base       8       0       4 0

  class Divide2 size(20) :
   +-- -
   0 | {vbptr}
   4 | d
   +-- -
   +-- - (virtual base Base)
   8 | {vfptr}
  12 | a
  16 | b
   +-- -

  Divide2::$vbtable@:
   0 | 0
   1 | 8 (Divide2d(Divide2 + 0)Base)

  Divide2::$vftable@:
 | -8
   0 | &Divide2::run

  Divide2::run this adjustor: 8
  vbi:    class  offset o.vbptr  o.vbte fVtorDisp
              Base       8       0       4 0

  class Divide size(32) :
   +-- -
   0 | +-- - (base class Divide1)
   0 | | {vbptr}
   4 | | c
 | +-- -
   8 | +-- - (base class Divide2)
   8 | | {vbptr}
  12 | | d
 | +-- -
  16 | d
   +-- -
   +-- - (virtual base Base)
  20 | {vfptr}
  24 | a
  28 | b
   +-- -

  Divide::$vbtable@Divide1@:
   0 | 0
   1 | 20 (Divided(Divide1 + 0)Base)

  Divide::$vbtable@Divide2@:
   0 | 0
   1 | 12 (Divided(Divide2 + 0)Base)

  Divide::$vftable@:
 | -20
   0 | &Divide::run
```

总结：通过内存分布可知，`Divide1`和`Divide2`都是两个虚表，Divide中却是成了3个虚表，只有一份base；所以说：**虚继承的作用是减少了对基类的重复，代价是增加了虚表指针的负担（增加了更多的需指针）**

