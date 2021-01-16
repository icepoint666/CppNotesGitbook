# 并发时用std::atomic而不是volatile

**当需要并发时使用 std::atomic ，特定内存才使用 volatile**

volatile本身没有关于并发的能⼒。但是在其他 编程语⾔中（⽐如，Java和C\#）， volatile 是有并发含义的

* 使⽤ std::atmoic 的代码，保证原子的加减，保证并发

```cpp
std::atomic<int> ai(0); // initialize ai to 0 
ai = 10; // atomically set ai to 10 
std::cout << ai; // atomically read ai's value 
++ai; //atomically increment ai to 11 
--ai; // atomically decrement ai to 10
```

* 使⽤ volatile 在多线程中不保证任何事情



