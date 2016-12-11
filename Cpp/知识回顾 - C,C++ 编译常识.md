前言
---
---
本文主要参考  [Compiling Cpp - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_Cpp)，并经过运行测试完成。要自行测试请安装好Gcc，:)，windows下面用cygwin装一下，mac下用包管理器homebrew装一下，linux下应该默认已经安装了gcc。

C/C++程序的编译其实主要有几个阶段：1.编译预处理(包括include头文件的复制，宏定义的展开处理等等)，2.把预处理后的文件经过语法(syntax)分析和语义(sematics)分析之后生成汇编文件，3.然后再利用汇编器和链接器(链接上动态链接库，插入静态链接库代码等等)生成可执行文件，比如Linux下就是elf格式的文件。

本文所用的指令(可执行程序)为`g++`，而非`gcc`，因为`g++`自动链接了C++标准库的动态链接库libstdc++.so，就像`gcc`自动链接了C标准库的动态链接库libglibc.so一样。

Gcc这个编译工具的一些常识
---
---
然后在实际写小程序编译时候，如果编译器版本大于gcc-4.8，我建议大家加上compiler flag `-std=c++11`；如果编译器版本大于gcc5.x，我建议大家加上compiler flag `-std=c++14`，因为最近看到一个youtube的视频 [C++之父在CppCon2016上的KeyNote The Evolution of C++ Past, Present and Future](https://www.youtube.com/watch?v=_wzc7a3McOs) , C++之父说目前建议大家学习C++的baseline至少为C++11标准。

另外请加上`-wall`这个compiler flag，让编译器报出所有可能的警告；然后如果你需要获得Simd(single instruction multiple data)俗称向量话还有其他的一些编译优化效果的话请加上`-O3`；然后如果你要调试程序的话，请加上`-g`，保留符号表信息。

C/C++编译涉及的文件命名规范
---
---
这一部分参考了 [Compiling Cpp - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_Cpp) 这篇Wiki的内容。文件命名规范如所示(包括文件后缀名，文件类型和补充说明):

- **`.h`** --- C/C++头文件  
  - Boost里面是.hpp   
- **`<xxx>,无后缀`** --- C++标准库头文件  
  - 比如写C程序时候`#include<string.h>`，那么在C++中就是`#include<cstring>`，这里所示的xxx里面可以include C风格的头文件，比如在cstring这个文件名对应的文件里面，我们`#include<string.h>`   
- **`.c .cpp`** --- C/C++源代码(这些后缀也允许:.C .cc .cp .cxx .c++)  
  - 需要编译预处理  
- **`.ii`** --- C++源代码  
  - 不需编译预处理，已经经过预处理复制了include头文件里面的内容，展开了宏定义的内容  
- **`.s`** --- 汇编代码文件  
  - 语法和语义分析后的输出   
- **`.o`** --- 对象文件  
  - 经过链接后才生成可执行文件   
- **`.a`** --- 静态库 (archive)  
  - 静态链接库，链接后会增加可执行文件的长度   
- **`.so`** --- 动态库(shared object)  
  - 动态链接库，库函数推迟到程序运行时期载入。用户只需要升级动态链接库，而无需重新编译链接其他原有的代码就可以完成整个程序的升级。   

Gcc-生成可执行文件-给个直观感受(Hello-World)
---
---
- hello_world.cpp的源代码

```cpp
#include <iostream>
int main(int argc,char *argv[])
{
    std::cout << "hello, world" << std::endl;
    return(0);
}
```
- 然后build_hello_world.sh的脚本

```zsh
mkdir -p build
g++ hello_world.cpp -o build/hello_world_elf_file
```

- 然后给我们的build_hello_world.sh脚本执行权限并运行

```zsh
chmod +x build_hello_world.sh
./build_hello_world.sh
build/hello_world_elf_file
```

- 运行可执行文件的结果是熟悉的hello, world

```zsh
hello, world
```


Gcc-编译预处理(Hello-World)
---
---
- 在写好了上一小节的`hello_world.cpp`后，我们只需要一行命令就可以产生预处理文件，命令如下:

```zsh
g++ -E hello_world.cpp -o hello_world.ii
```

引用下 [Compiling Cpp - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_Cpp) 的一段话：
> 选项 -E 使 g++ 将源代码用编译预处理器处理后不再执行其他动作。
> 本文前面所列出的 helloworld.cpp 的源代码，仅仅有六行，而且该程序除了显示一行文字外什么都不做，但是，预处理后的版本将超过 1200 行。这主要是因为头文件 iostream 被包含进来，而且它又包含了其他的头文件，除此之外，还有若干个处理输入和输出的类的定义。

- 查看在我的mac笔记本上生成的代码行数，输出结果为37586，可见预处理的时候展开了很多内容和include了很多内容（复制头文件中所有的内容到当前预处理文件的输出文件，比如`hello_world.ii`）  

```zsh
grep -c "" build/hello_world.ii
```

Gcc-生成汇编代码(Hello-World)
---
---
引用下 [Compiling Cpp - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_Cpp) 的一段话：
> 选项 -S 指示编译器将程序编译成汇编语言，输出汇编语言代码而後结束。下面的命令将由 C++ 源码文件生成汇编语言文件 helloworld.s：  

```zsh
g++ -S hello_world.cpp -o build/hello_world.s
```
- 产生出来的汇编代码有点长，我就不粘贴上来了。

Gcc-处理多个源文件(Multiple-File-Hello-World)
---
---
- 主要包含三个文件，一个头文件(hello_world_util.h)，一个类文件(hello_world_util.cpp)，另一个文件包含main方法(hello_world_main.cpp)
- hello_world_util.h

```cpp
#include <iostream>
class SayUtil
{
    public:
        void sayStr(const char *);
};
```  

- hello_world_util.cpp

```cpp
#include "hello_world_util.h"
void SayUtil::sayStr(const char *str)
{
    std::cout << str << "\n";
}
```

- hello_world_main.cpp

```cpp
#include "hello_world_util.h"
int main(int argc, char const *argv[]) {
  SayUtil say_util;
  say_util.sayStr("hello, world");
  return 0;
}
```
- build的时候，先生成`.o`文件，然后链接产生可执行文件，然后删除中间文件，下面是我的build_mutiple_file_hello_world0.sh脚本内容:  

```zsh
mkdir -p build
g++ -c hello_world_util.cpp -o build/hello_world_util.o
g++ -c hello_world_main.cpp -o build/hello_world_main.o
#link and generate executable
g++ build/hello_world_util.o build/hello_world_main.o -o build/multiple_file_hello_world0
#remove intermediate files
rm build/hello_world_util.o build/hello_world_main.o
```

- 然后也可以让g++帮助我们来写中间的生成`.o`文件的过程，下面是我的build_mutiple_file_hello_world1.sh脚本内容:  

```zsh
mkdir -p build
g++ hello_world_main.cpp hello_world_util.cpp -o build/multiple_file_hello_world1
```

Gcc-创建静态链接库
---
---
引用下 [Compiling Cpp - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_Cpp) 的一段话：
> 静态库是编译器生成的一系列对象文件的集合。链接一个程序时用库中的对象文件还是目录中的对象文件都是一样的。库中的成员包括普通函数，类定义，类的对象实例等等。静态库的另一个名字叫归档文件(archive)，管理这种归档文件的工具叫 ar 。

在讲静态链接库之前，我主要先提一下几个注意点：

- 编译器是比较笨的，所以如果我们需要在一个cpp文件中使用另一个cpp文件中的一个全局变量的时候，我们必须要通过`extern 类名 对象名;`告诉编译器这个对象名已经在其他编译单元中被定义了(分配了stack上的一个空间了)。比如，一个文件中定义了`Say librarysay("Library instance of Say");`，也就是在进程的栈上分配了空间，并进行了初始化；我们在另一个文件中先得通过`extern Say librarysay;`声明一下，表示这个librarysay对象在其他的编译单元中被定义过了，我们后来要用这个全局变量，告诉编译器一下，不要报错。这边的`extern`必须要加上，因为否则编译器会认为我们要在进程的栈上定义一个名为librarysay的变量，调用默认构造函数。
- 然后就是其他编译单元的全局函数使用了，我们必须先声明一下这个函数，表示我们要使用他，这个函数可以是在其他编译单元进行定义的，只要最后链接成可执行文件的时候，我们能找到函数的定义，编译器就不会报错了。然后。比如我`void sayhello(void);`这个函数的定义在其他编译单元，但我要使用，那么我就直接`void sayhello(void);`声明一下，告诉编译器这个东西我要用，然后是在其他编译单元定义的。声明这个函数的时候extern好像可以不加，比如`void sayhello(void);`和`extern void sayhello(void);`在我编译测试运行的时候都可以。因为这和上一点的声明对象不同，函数没有构造语义学。

然后就是例子中要用到的4个文件了, 其中前三个会编译成`.o`文件，然后通过`ar`打包到静态链接库，另一个是包含主方法的文件：

- say_util.h   

```cpp
#include <iostream>
extern void sayhello(void);

class Say {
private:
  char *string;

public:
  Say(char *str) { string = str; }
  void sayOther(const char *str) {
    std::cout << str << " from \"" << string << "\"" << std::endl;
  }
  void sayString(void);
};

```

- say_util.cpp

```cpp
#include "say_util.h"
void Say::sayString() { std::cout << string << std::endl; }

// For built-in-library usage
Say librarysay("Library instance of Say");

```

- say_hello_func.cpp

```cpp
#include <iostream>
void sayhello() { std::cout << "hello from a static library" << std::endl; }

```

- say_hello_main.cpp

```cpp
#include "say_util.h"

int main(int argc, char *argv[]) {
  //告诉编译器这个Say类型的变量(symbol)
  // librarysay在其他的编译单元中，可作为全局变量使用
  extern Say librarysay;
  Say localsay = Say("Local instance of Say");
  sayhello();
  librarysay.sayOther("say Something from librarysay");
  librarysay.sayString();
  localsay.sayString();
  return (0);
}

```

- build_say_hello_with_archive.sh脚本内容，主要三步走：1.生成`.o`文件 2. 通过`ar`打包成静态链接库 3. 然后再通过链接器链接起来，搞成一个可执行文件。

```zsh
mkdir -p build
g++ -c say_util.cpp -o build/say_util.o
g++ -c say_hello_func.cpp -o build/say_hello_func.o
#程序 ar 配合参数 -r 创建一个新库 libsay.a 并将命令行中列出的对象文件插入。
#采用这种方法，如果库不存在的话，参数 -r 将创建一个新的库，而如果库存在的话，将用新的模块替换原来的模块。
ar -r build/libsay.a build/say_util.o build/say_hello_func.o
g++ say_hello_main.cpp build/libsay.a -o build/say_hello_main
rm build/*.o
```

Gcc-创建动态链接库
---
---
引用下 [Compiling C - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_C) 的一段话：
> 共享库是编译器以一种特殊的方式生成的对象文件的集合。对象文件模块中所有地址（变量引用或函数调用）都是相对而不是绝对的，这使得共享模块可以在程序的运行过程中被动态地调用和执行。  

- 基于这一点，我们在生成`.o`文件的时候，就得通过添加编译flag `-fpic` 的形式告诉编译器：生成的对象模块采用浮动的（可重定位的）地址。  

- 关于生成`.so`文件(动态链接库文件的说明)，再引用下 [Compiling C - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_C) 的一段话：  

> 选项 -o 用来为输出文件命名，而文件後缀名 .so 告诉编译器将对象文件链接成一个共享库。通常情况下，链接器定位并使用 main() 函数作为程序的入口，但是本例中输出模块中没有这种入口点，为抑制错误选项 -shared 是必须的。

- 下面我将使用上一小节静态链接库中的源代码，只不过把静态链接库改为使用动态链接库，来展示动态链接库的使用。值得注意的一点是，在使用动态链接库时候，`.so`文件中的代码会在运行时候加载进来，而不是像`.a`文件在编译时候加载进来，所以我们必须要设置环境变量，以供我们的系统在执行可执行文件时候能找到对应的`.so`文件，比如先使用如下指令添加查找动态链接库的目录为`./build`(当前目录的build文件夹目录):   

```zsh
 export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./build/
```

- build_say_hello_with_shared_object.sh的脚本如下:  

```zsh
mkdir -p build
#添加-fpic 这个compiler flag，告诉编译器，
#生成的对象模块采用浮动的（可重定位的）地址。缩微词 pic 代表“位置无关代码”（position independent code）
g++ -c say_util.cpp -fpic -o build/say_util.o
g++ -c say_hello_func.cpp -fpic -o build/say_hello_func.o

g++ -shared build/say_util.o  build/say_hello_func.o -o build/libsay.so
rm build/*.o
```

```zsh
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./build/

g++ say_hello_main.cpp build/libsay.so -o build/say_hello_main_with_so
```




最后：我的实验代码和脚本Github链接
---
---
- [链接地址](https://github.com/CheYulin/CPP11Study/tree/master/GccUsage)
- 具体文件信息可以参看所给链接目录下 `ReadMe.md` 中的说明

参考文章
---
---
* [Compiling Cpp - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_Cpp)
* [Compiling C - By Ubuntu](http://wiki.ubuntu.org.cn/Compiling_C)
* [Linux 静态链接库和动态链接库的区别](http://www.cnblogs.com/zhoutian6214/archive/2008/11/11/1331646.html)
