# C\#调用C++DLL函数，一般怎么封装这个DLL？

如果你的代码需要大量使用C++类，C++/CLI是最佳选择。原来的C++代码可以不动，所有要用到的类套一层代理类就可以了。既是类型安全的，性能也接近native。

关于C++/CLI: [https://www.codeproject.com/Articles/19354/Quick-C-CLI-Learn-C-CLI-in-less-than-10-minutes](https://www.codeproject.com/Articles/19354/Quick-C-CLI-Learn-C-CLI-in-less-than-10-minutes)

其实C++/CLI除了包含ISOC++和CLI扩展，它之所以强大的更重要的原因就是它**实现了ISO C++和.NET的无缝连接。（.NET框架对于C\#而言，相当于JVM对于Java而言）也就是说C++/CLI可以与C\#无缝衔接**

**使用C++/CLI想创造一个dll供C\#调用**

**示例：用C++实现一个返回字符串的操作，C\#代码可以用**

```cpp
// testclass.h
#pragma once

#include <string>

namespace test
{
    public ref class testclass//类前面必须有ref限定符，创建一个CLI管理的类
    {
      public:
         //这里不能直接像下面这样用！！
         std::string getstringfromcpp()
         {
            return "Hello World";   
         }
    };
}
```

And I want to use it in C\# program, add this dll to reference then:

```csharp
// C#
using test; // from C++ dll
... 
testclass obj = new testclass();
textbox1.text = obj.getstringfromcpp();
...
```

**正确做法**： Don't return a `std::string` \(there is no such thing in C\#\) or a `const char *` \(readable from C\# but you'd have to manage memory deallocation\) or things like that. Return a `System::String^` instead. This is the _standard_ string type in .NET code.

```cpp
#using <mscorlib.dll>
using namespace System

public: System::String^ getStringFromCpp()
{
    return "Hello World";   
}
```

But if you actually have a `const char *` or `std::string` object you'll have to use the `marshal_as` template:

```cpp
#include <msclr/marshal.h>

public: System::String^ getStringFromCpp()
{
    const char *str = "Hello World";
    return msclr::interop::marshal_as<System::String^>(str);
}
```

 Read [Overview of Marshaling in C++](https://msdn.microsoft.com/en-us/library/bb384865.aspx) for more details.

