# 静态成员函数的最大特点

**静态成员函数** 不接受**隐含的this自变量**。

所以，它就**无法访问自己类的非静态成员**。

实例：写单例模式的时候mutex要在外面创建，不能在类中

```cpp
std::mutex m;
class Log{
public:
    static Log* get_log(){
        if(log == 0){
            m.lock();
            if(log == 0)
                log = new Log();
            m.unlock();
        }
        return log;
    }
private:
    Log(){};
    //错误：std::mutex m;
    static Log* log;
}

```

