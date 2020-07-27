# namespace

### 为什么需要namespace命名空间

C++中经常使用多个独立开发的库来完成项目，由于库的作者或开发人员没见过面，因此库与库之间命名冲突在所难免

命名空间作用就是对这些命名进行 逻辑空间划分（不是物理单元划分）通过使用namespace XXX把库中的变量、函数、类型、结构等包含在名字空间中，形成自己的作用域，避免 名字冲突

```cpp
 namespace xxx
 {

 }// 没有分号
```

### 一些原则

* 同名的namespace有自动合并，同名的namespace中如果有重名的依然会命名冲突
* namespace使用方法：
  * space::function
  * using namespace space
* namespace的嵌套

  ```cpp
  n1::n2::num;
   namespace n1
   {
       int num = 1;
       namespace n2
       {   
           int num = 2;
           namespace n3
           {

           }
       }
   }
  ```

