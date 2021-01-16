# 返回值优化

### 返回值优化 RVO

正常函数内构造一个对象，返回该对象，再将函数返回值赋值给一个对象

编译器会针对这种情况进行**返回值优化（Return Value Optimization, RVO）**

**如果编译器没有优化，会进行三次构造，大部分C++编译器会把这个过程优化为一步构造**

```cpp
int g_constructCount=0;
int g_copyConstructCount=0;
int g_destructCount=0;

struct A {
    A(){              // 基本构造
        cout<<"construct: "<<++g_constructCount<<endl;    
    }
    
    A(const A& a) {   // 拷贝构造
        cout<<"copy construct: "<<++g_copyConstructCount <<endl;
    }
    
    ~A() {            // 析构
        cout<<"destruct: "<<++g_destructCount<<endl;
    }
};

A getA() {
    A a;            // 第一次构造
    return a;
    // return A();  // 等价，分开写是为了便于说明
}

int main() {
    A aa = getA();    // 第二次构造：把 a 复制给一个临时变量，
                      // 第三次构造：把临时变量复制给 aa
                      // 开启优化后，相当于直接把 a “改名”成 aa 了，所以只有一次构造
    return 0;
}
```

* 关闭编译器优化的结果

  ```text
  construct: 1        // 第一次构造，getA() 中的局部变量 a    
  copy construct: 1   // 第二次构造，将 a 复制给一个临时变量
  destruct: 1           // 析构局部变量 a
  copy construct: 2   // 第三次构造，将临时变量复制给 aa
  destruct: 2           // 析构临时变量
  destruct: 3           // 程序结束，析构变量 aa
  ```

* 开启编译器优化

  ```text
  construct: 1        // 构造局部变量 a，在编译的优化下，相当于直接将 a “改名” aa
  destruct: 1         // 程序结束，析构变量 aa
  ```

