# 两个类互相调用：前向声明

## Forward Declarations 前向声明

```cpp
//file: bank.h
class Account; //forward declaration
class Report{
public:
void Output(const Account& account); //fine: Account is known to be a not-yet defined class
};
class Account {
Report rep;
public:
void Show() {Report::Output(*this);}
};
```

