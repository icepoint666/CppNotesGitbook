# void \* 与 member function指针 的转换

**不能cast成员函数指针为void\***，因为成员的指针不仅仅是一个地址，建议的解决办法是给member function **wrap成一个常规函数** 

You [cannot cast a pointer-to-member to `void *`](https://isocpp.org/wiki/faq/pointers-to-members#cant-cvt-memfnptr-to-voidptr) or to any other "regular" pointer type. Pointers-to-members are not addresses the way regular pointers are. 

What you most likely will need to do is **wrap your member function in a regular function**. The C++ FAQ Lite [explains this](https://isocpp.org/wiki/faq/pointers-to-members) in some detail. The main issue is that the data needed to implement a pointer-to-member is not just an address, and in fact [varies tremendously](http://www.codeproject.com/KB/cpp/FastDelegate.aspx) based on the compiler implementation.

