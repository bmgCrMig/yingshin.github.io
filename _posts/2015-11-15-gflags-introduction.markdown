---
layout: post
title:  "参数解析gflags介绍"
date: 2015-11-15 00:22:58
excerpt: "参数解析gflags介绍"
categories: [C/CPP]
tags: [gflags]
---


gflags是google一个开源的处理命令行参数的库，相比getopt，更加容易使用。  
gflags里参数的定义可以分散在各个源文件处，而不是只能在main文件处，使得使用更加灵活，复用性更强。  
使用时只要链接gflags库和指定所在的头文件路径即可。  

<!--more-->

当然，如果两个源文件定义了相同的flag，链接时会报重复定义的错误。  

先看个简单的例子:   

```
#include <iostream>
#include <gflags/gflags.h>

DEFINE_bool(big_menu, true, "Include 'advanced' options in the menu listing");
DEFINE_string(languages, "english,french,german",
        "comma-separated list of languages to offer in the 'lang' menu");

int main(int argc, char* argv[])
{
    google::ParseCommandLineFlags(&argc, &argv, true);
    std::cout << FLAGS_big_menu << std::endl;
    std::cout << FLAGS_languages << std::endl;
    return 0;
}
```

上述代码包含了引用头文件、定义、解析、使用flags。

输出如下：  

```
1
english,french,german
```

### 首先看下如何定义

flag的定义方法如下：  

```
   #include <gflags/gflags.h>

   DEFINE_bool(big_menu, true, "Include 'advanced' options in the menu listing");
   DEFINE_string(languages, "english,french,german",
                 "comma-separated list of languages to offer in the 'lang' menu");
```

定义flag一共6个宏：  

```
DEFINE_bool: boolean
DEFINE_int32: 32-bit integer
DEFINE_int64: 64-bit integer
DEFINE_uint64: unsigned 64-bit integer
DEFINE_double: double
DEFINE_string: C++string
```

### 如何查看程序支持了哪些flags
例如对上述flags的定义，-help输出如下：  

```
  Flags from flags_help.cpp:
    -big_menu (Include 'advanced' options in the menu listing) type: bool
      default: true
    -languages (comma-separated list of languages to offer in the 'lang' menu)
      type: string default: "english,french,german"
```

### 如何使用
在命令行里指定，例如`./flags_sample -big_menu=0 -languages="english"`  
使用时，对应的变量名为`FLAGS_xxx`。  
如果使用时不想在main文件里定义flag，例如需要在flag1.cpp flags.cpp里定义,可以这么使用:  
在flags1.cpp flags2.cpp里分别定义各自的flag，然后在flag.h声明，需要使用的文件直接`include flags.h`就可以了。  
声明的函数如下：  

```
   DECLARE_bool(big_menu);
```

### 如何解析
`gflags::ParseCommandLineFlags(&argc, &argv, remove_flags)`可以帮助解析出相应的flags  
__argc argv__: main中的对应参数  
__remove\_flags__: 若设置为true，表示解析后将flag以及flag对应的值从argv中删除，并相应的修改argc，即最后存放的是不包含flag的参数。如果设置为false，则仅对参数进行重排，标志位参数放在最前面。   
返回值：文档中说明如下，   
> ParseCommandLineFlags returns the index into argv that holds the first commandline argument: that is, the index past the last flag.  

我觉得是返回处理后的argv第一个非flag值的下标  

### 检查有效性
gflags还提供了一个检查传入flag值是否有效的功能，只要定义检测函数，并且注册就可以了。  
检测函数以及注册方式的例子：  

```
static bool ValidatePort(const char* flagname, int32 value) {
   if (value > 0 && value < 32768)   // value is ok
     return true;
   printf("Invalid value for --%s: %d\n", flagname, (int)value);
   return false;
}
DEFINE_int32(port, 0, "What port to listen on");
static const bool port_dummy = RegisterFlagValidator(&FLAGS_port, &ValidatePort);
```

如果注册成功，regist函数返回值为ture。否则返回false，注册失败一般是一下两种原因：  
1. 第一个不是flag  
2. 该flag已经注册过  

写了一个完整的[例子](https://gist.github.com/yingshin/35a17cc4a6631002d3e0)  

参考资料： 
[https://gflags.github.io/gflags/](https://gflags.github.io/gflags/)