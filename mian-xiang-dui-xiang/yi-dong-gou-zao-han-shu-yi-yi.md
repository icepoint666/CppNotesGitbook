# 移动构造函数意义

### 移动构造函数的由来 <a id="&#x79FB;&#x52A8;&#x6784;&#x9020;&#x51FD;&#x6570;&#x7684;&#x7531;&#x6765;"></a>

**没有移动构造函数**会导致的**性能瓶颈**：

```cpp
class A {
    public:
        A(){
            std::cout << "A construct..." << std::endl;
            ptr_ = new int(100);
        }

        A(const A & a){
            std::cout << "A copy construct ..." << std::endl;
            ptr_ = new int();
            memcpy(ptr_, a.ptr_, sizeof(int));
        }

        ~A(){
            std::cout << "A deconstruct ..." << std::endl;
            if(ptr_){
                delete ptr_;
            }
        }

        A& operator=(const A & a) {
            std::cout << " A operator= ...." << std::endl;
            return *this;
        }

        int * getVal(){
            return ptr_;
        }
    private:
        int *ptr_;
};

int main(int argc, char *argv[]){
    std::vector<A> vec;
    vec.push_back(A());
}
```

**导致的问题：两个对象拷贝，**就是在本身创建了一个对象A，然后通过移动构造函数的深拷贝，vector中相当于又创建了一个对象A

**输出验证一下确实创造了两个对象：**

```text
A construct...          //main中创建的A对象
A copy construct ...    //vector内部创建的A对象
A deconstruct ...       //vector内部创建的A对象被析构
A deconstruct ...       //main中创建的A对象析构
```

vector中要存放大量的A对象时（如 100000个\)，就会不断的做分配/释放堆空间的操作，这会造成多在的性能消耗呀！

### 移动构造函数的使用 <a id="&#x79FB;&#x52A8;&#x6784;&#x9020;&#x51FD;&#x6570;&#x7684;&#x4F7F;&#x7528;"></a>

```cpp
class A {
    public:
        ...

        A(A && a){
            std::cout << "A move construct ..." << std::endl;
            ptr_ = a.ptr_;
            a.ptr_ = nullptr;
        }
        ...
};

int main(int argc, char *argv[]){
    std::vector<A> vec;
    vec.push_back(std::move(A()));
}
```

**原理：**

* 1.std::move里面的`A()`是一个普通构造函数，创建这个对象
* 2.std::move会将任何值都转换为右值，右值有什么用呢，等会儿说到
* 3.因为vector也是关于A的类型，所以对于右值的push\_back就会调用**移动构造函数**
* 4.移动构造函数：里面将a.ptr\_设置为了nullptr（有没有必要不好说）

返回的是右值，后面讲**std::move**会讲到其中的原理

运行结果：

```cpp
A construct...          //main中创建A对象
A move construct ...    //vector内部通过移动构造函数创建A对象，减少了对堆空间的频繁操作
A deconstruct ...       //释放vector中的A对象
A deconstruct ...       //释放main中创建的A对象
```

大大减了频繁对堆空间的分配/释放操作，从而提高了程序的执行效率

在移动构造函数操作之后原A对象的指针地址已经指向NULL了，因此此时就不能再通过其访问之前的堆空间了**（如果不加这一句还能访问吗？之后调研）**

### 右值引用，右值

右值相对于左值的区分：

**右值是一个临时的值，用完后立刻被销毁**

**左值**相当于是**存储在内存**中，**右值**就是**存储在寄存器**中，所以右值是临时

所以**std::move的作用：**是将任何值都转换为右值，右值也是转为右值，左值也是转为右值，本质上相当于把变量从内存中移动到寄存器，因为只能作为右值使用，所以这个语句过后，这个右值引用也就是寄存器里面存的变量就会被销毁了

