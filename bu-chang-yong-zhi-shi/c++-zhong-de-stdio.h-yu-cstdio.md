# c++中的stdio.h与cstdio

* 两者c++都可以拿来\#include
* c++中推荐使用cstdio，因为cstdio头文件中定义的名字被定义在命名空间std中\(using namespace std对stdio.h没用）

**cstdio的代码：**

```cpp
// cstdio standard header
#pragma once
#ifndef _CSTDIO_
#define _CSTDIO_
#include <yvals.h>

#ifdef _STD_USING
#undef _STD_USING
  #include <stdio.h>
#define _STD_USING

#else 
#include <stdio.h>
#endif 

#define _HAS_CONVENTIONAL_CLIB    1

#define _IOBASE    _base
#define _IOPTR    _ptr
#define _IOCNT    _cnt

#ifndef RC_INVOKED
#if _GLOBAL_USING
_STD_BEGIN
using ::size_t; using ::fpos_t; using ::FILE;
using ::clearerr; using ::fclose; using ::feof;
using ::ferror; using ::fflush; using ::fgetc;
using ::fgetpos; using ::fgets; using ::fopen;
using ::fprintf; using ::fputc; using ::fputs;
using ::fread; using ::freopen; using ::fscanf;
using ::fseek; using ::fsetpos; using ::ftell;
using ::fwrite; using ::getc; using ::getchar;
using ::gets; using ::perror;
using ::putc; using ::putchar;
using ::printf; using ::puts; using ::remove;
using ::rename; using ::rewind; using ::scanf;
using ::setbuf; using ::setvbuf; using ::sprintf;
using ::sscanf; using ::tmpfile; using ::tmpnam;
using ::ungetc; using ::vfprintf; using ::vprintf;
using ::vsprintf;
_STD_END
#endif 
#endif 

#ifndef _Filet
#define _Filet    FILE
#endif 

#ifndef _FPOSOFF
  #define _FPOSOFF(fp)  ((long)(fp))
#endif 

#endif 
```

