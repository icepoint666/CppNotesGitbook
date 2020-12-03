# Table of contents

* [INTRO](README.md)
* [知识树](zhi-shi-shu.md)
* [相关链接](xiang-guan-lian-jie.md)

## 语言相关

* [C/C++语言上的特性](yu-yan-xiang-guan/cc++-yu-yan-shang-de-te-xing.md)
* [C++中的POD类型](yu-yan-xiang-guan/c++-zhong-de-pod-lei-xing.md)
* [C/C++语法的复杂度](yu-yan-xiang-guan/c++-yu-fa-de-fu-za-du.md)
* [C/C++关于ABI兼容的问题](yu-yan-xiang-guan/cc++-guan-yu-abi-jian-rong-de-wen-ti.md)
* [C++序列化，protobuf](yu-yan-xiang-guan/c++-xu-lie-hua-protobuf.md)

## 运算符

* [sizeof，内存对齐](yun-suan-fu/untitled.md)

## 数据

* [输出变量类型](shu-ju/c++-shu-chu-bian-liang-lei-xing.md)
* [类型转换](shu-ju/lei-xing-zhuan-huan/README.md)
  * [C++中std::string转换为int类型](shu-ju/lei-xing-zhuan-huan/c++-zhong-stdstring-zhuan-huan-wei-int-lei-xing.md)
  * [C++ double或者int转为string](shu-ju/lei-xing-zhuan-huan/c++-double-huo-zhe-int-zhuan-wei-string.md)
  * [C++中四种cast类型转换](shu-ju/lei-xing-zhuan-huan/c++-zhong-si-zhong-cast-lei-xing-zhuan-huan.md)

## 字符串

* [strcpy, strcmp, strstr的实现](zi-fu-chuan/strcpy-strcmp-de-shi-xian.md)
* [ASCII, Unicode, UTF-8编码](zi-fu-chuan/ascii-unicode-utf8-bian-ma.md)

## 关键字

* [const](guan-jian-zi/const.md)
* [constexpr](guan-jian-zi/constexpr.md)
* [extern](guan-jian-zi/extern.md)
* [static](guan-jian-zi/static.md)
* [this](guan-jian-zi/this.md)
* [using](guan-jian-zi/using.md)
* [inline](guan-jian-zi/inline.md)
* [volatile](guan-jian-zi/volatile.md)
* [decltype](guan-jian-zi/decltype.md)
* [=default, =delete](guan-jian-zi/default-delete.md)
* [namespace, ::运算符](guan-jian-zi/namespace.md)
* [union](guan-jian-zi/union.md)
* [explicit](guan-jian-zi/explicit.md)

## 内存管理

* [栈与堆](nei-cun-guan-li/zhan-yu-dui.md)
* [C++ 垃圾回收\(gc\)](nei-cun-guan-li/c++-la-ji-hui-shou-gc.md)
* [RAII思想](nei-cun-guan-li/raii-si-xiang.md)
* [malloc、calloc、realloc、alloca](nei-cun-guan-li/malloc-calloc-realloc-alloca.md)
* [new, delete, 定位 new](nei-cun-guan-li/new-delete-ding-wei-new.md)
* [memcpy, memcmp, memset用法](nei-cun-guan-li/memcpy-memcmp-memset-yong-fa.md)
* [智能指针](nei-cun-guan-li/zhi-neng-zhi-zhen.md)
* [尾递归优化](nei-cun-guan-li/wei-di-gui-you-hua.md)

## 面向对象

* [多态（两类四种）](mian-xiang-dui-xiang/duo-tai.md)
* [默认构造函数](mian-xiang-dui-xiang/mo-ren-gou-zao-han-shu.md)
* [虚函数](mian-xiang-dui-xiang/xu-han-shu.md)
* [虚表 函数绑定](mian-xiang-dui-xiang/xu-biao-han-shu-bang-ding.md)
* [虚函数，虚继承内存模型](mian-xiang-dui-xiang/xu-han-shu-xu-ji-cheng-nei-cun-mo-xing.md)
* [单例模式](mian-xiang-dui-xiang/dan-li-mo-shi.md)
* [移动构造函数意义，右值，返回值优化](mian-xiang-dui-xiang/yi-dong-gou-zao-han-shu-yi-yi.md)
* [深入std::move的实现](mian-xiang-dui-xiang/shen-ru-stdmove-you-zhi-yin-yong.md)
* [深入std::forward完美转发的实现](mian-xiang-dui-xiang/shen-ru-stdforward-wan-mei-zhuan-fa.md)

## STL

* [find，remove，erase](stl/untitled.md)

## 高效编程tricks

* [Effective C++（浓缩部分）](gao-xiao-bian-cheng-tricks/effective-c++.md)
* [More Effective C++](gao-xiao-bian-cheng-tricks/more-effective-c++.md)
* [Modern Effective C++](gao-xiao-bian-cheng-tricks/xian-dai-c++.md)
* [Effective STL](gao-xiao-bian-cheng-tricks/effective-stl.md)
* [Google编程格式规范](gao-xiao-bian-cheng-tricks/google-bian-cheng-ge-shi-gui-fan.md)
* [Google C++ Style Guide](gao-xiao-bian-cheng-tricks/google-c++-style-guide.md)
* [代码整洁之道（Java示例）](gao-xiao-bian-cheng-tricks/dai-ma-zheng-jie-zhi-dao-java-shi-li.md)

## 不常用知识

* [执行效率](bu-chang-yong-zhi-shi/zhi-hang-xiao-lv/README.md)
  * [Duff's device算法优化循环](bu-chang-yong-zhi-shi/zhi-hang-xiao-lv/duffs-device-suan-fa-you-hua-xun-huan.md)
* [汇编语言相关](bu-chang-yong-zhi-shi/hui-bian-yu-yan-xiang-guan/README.md)
  * [\[1\]汇编语言概念，寄存器，内存模型](bu-chang-yong-zhi-shi/hui-bian-yu-yan-xiang-guan/1-hui-bian-yu-yan-gai-nian-ji-cun-qi-nei-cun-mo-xing.md)
  * [\[2\] C程序与汇编语言对应代码](bu-chang-yong-zhi-shi/hui-bian-yu-yan-xiang-guan/2-c-cheng-xu-yu-hui-bian-yu-yan-dui-ying-dai-ma.md)
* [c++中的stdio.h与cstdio](bu-chang-yong-zhi-shi/c++-zhong-de-stdio.h-yu-cstdio.md)
* [闭包以及匿名函数](bu-chang-yong-zhi-shi/bi-bao-yi-ji-ni-ming-han-shu.md)
* [C\#调用C++DLL函数，一般怎么封装这个DLL？](bu-chang-yong-zhi-shi/c-tiao-yong-c++dll-han-shu-yi-ban-zen-me-feng-zhuang-zhe-ge-dll.md)
* [函数指针](bu-chang-yong-zhi-shi/han-shu-zhi-zhen.md)
* [static \_\_inline\_\_ / extern \_\_inline](bu-chang-yong-zhi-shi/static-__inline__-extern-__inline.md)

## 项目阅读笔记 <a id="xiang-mu-yue-du-bi-ji-1"></a>

* [C++项目流程](xiang-mu-yue-du-bi-ji-1/c++-xiang-mu-liu-cheng/README.md)
  * [配置Clion + Cmake + gdb编译开发环境](xiang-mu-yue-du-bi-ji-1/c++-xiang-mu-liu-cheng/pei-zhi-clion-+-cmake-+-gdb-bian-yi-kai-fa-huan-jing.md)
  * [单元测试，gtest](xiang-mu-yue-du-bi-ji-1/c++-xiang-mu-liu-cheng/dan-yuan-ce-shi-gtest.md)
* [定位内存泄漏](xiang-mu-yue-du-bi-ji-1/ding-wei-nei-cun-xie-lou.md)
* [泛型编程](xiang-mu-yue-du-bi-ji-1/fan-xing-bian-cheng/README.md)
  * [Traits编程技法获取迭代器取值类型](xiang-mu-yue-du-bi-ji-1/fan-xing-bian-cheng/traits-bian-cheng-ji-fa-huo-qu-die-dai-qi-qu-zhi-lei-xing.md)
  * [输出STL容器值](xiang-mu-yue-du-bi-ji-1/fan-xing-bian-cheng/shu-chu-stl-rong-qi-zhi.md)

## 面试问题

* [基础问题相关](mian-shi-wen-ti/ji-chu-wen-ti-xiang-guan.md)
* [面向对象相关](mian-shi-wen-ti/mian-xiang-dui-xiang-xiang-guan.md)
* [高阶问题相关](mian-shi-wen-ti/gao-jie-wen-ti-xiang-guan.md)

## bugs记录 <a id="bugs-ji-lu-1"></a>

* [volatile shared\_ptr 不能与 nullptr进行比较](bugs-ji-lu-1/volatile-sharedptr-bu-neng-yu-nullptr-jin-hang-bi-jiao.md)
* [vector中存储future变量](bugs-ji-lu-1/untitled.md)
* [volatile T类型返回时不等于T&](bugs-ji-lu-1/volatiletlei-xing-fan-hui-shi-bu-deng-yu-t.md)
* [static静态函数中用到的static静态变量声明](bugs-ji-lu-1/static-jing-tai-han-shu-zhong-yong-dao-de-static-jing-tai-bian-liang-sheng-ming.md)

