前言
---
---
本文主要参考了C++标准委员会出品的 [Mix-c-and-cpp](https://isocpp.org/wiki/faq/mixing-c-and-cpp) 和  [Learning C++ if you already know C](https://isocpp.org/wiki/faq/c) 两个链接对应的一些问答内容，加上我个人的理解完成。本文先会说明C++和C的发展历史和它们之间关系，然后从编程范式上提纲挈领地总结一下表达下我个人的感受，之后从几个区别点来具体说明它们的不同。有不准确或疏漏的地方还请大家指正，谢谢。

然后，关于C++语言标准参考文档我推荐 [cppreference c++ language document](http://en.cppreference.com/w/cpp/language) ；关于C语言标准参考文档我推荐 [cppreference c language document](http://en.cppreference.com/w/c/language) ；有兴趣的朋友可以看看，当然也可以把它们当成参考文档查阅用。

然后现在最新的C++标准是C++1x，C标准是C1x，希望大家都可以用最新的标准，因为最新的标准填上了很多坑。另外，提醒一下，其实C++包括两个部分：C++语言(language)部分和C++库(library)部分，学C++这两部分都要学好；学习C语言也是一样。

本文的主要内容包括四个部分。第一部分是 **背景:C1x与C++1x是一对兄弟，C95是他们的爹** ，第二部分是 **C/C++区别-从编程范式上来讲** ，第三部分是 **C/C++区别-重要的(区别0-9)** ， 第四部分是 **C/C++区别-其他(区别10-12和其他)** 。本文最末给出了一些相关的参考文章。

(一/四) 背景:C1x与C++1x是一对兄弟，C95是他们的爹
---
---
引用一下 [Learning C++ if you already know C](https://isocpp.org/wiki/faq/c) 中的一段话，概括一下C++1x/C1x和C95的关系。C++98和C99其实已经有不兼容的地方了，那么在之后的进程中这些不兼容的点会更多；因为事实上在语言层面上，C++新标准和C新标准制定的是不同语言的标准，只不过它们的语法有许多相同或者相似的部分。然后下面引用的段落中的几个链接为C++之父推荐看的文档或书籍，大家感兴趣的话，可以看一看这些资料。

> C++ is a direct descendant of C95 (C90 plus an Amendment) that retains almost all of C95 as a subset. C++ provides stronger type checking than C and directly supports a wider range of programming styles than C. C++ is “a better C” in the sense that it supports the styles of programming done using C with better type checking and more notational support (without loss of efficiency). In the same sense, ANSI C90/C95 is a better C than K&R C. In addition, C++ supports data abstraction, object-oriented programming, and generic programming (see [The C++ Programming Language](http://stroustrup.com/4th.html); Appendix B discussing compatibility issues is available for downloading).

> We have never seen a program that could be expressed better in C95 than in C++ (and we don’t think such a program could exist – every construct in C95 has an obvious C++ equivalent). However, there still exist a few environments where the support for C++ is so weak that there is an advantage to using C instead. There aren’t all that many of those left, though; see [Stroustrup’s (incomplete) compilers list](http://stroustrup.com/compilers.html).

> For a discussion of the design of C++ including a discussion of its relationship with C see [The Design and Evolution of C++](http://stroustrup.com/dne.html).

> Please note that “C” in the paragraphs above refers to Classic C and C95 (C90 with an Amendment). C++ is not a descendant of C99; rather, both are derived from C95. C++11 adopted all of C99’s preprocessor extensions and library extensions, but not C99’s new language features, so language features like the restrict keyword that were added in C99 are generally not part of ISO C++. Here is a description of [the differences between C++98 and C99](http://david.tribble.com/text/cdiffs.htm).

(二/四) C/C++区别-从编程范式上来讲
---
---
从编程范式上来讲，C语言是一种遵循面向过程，或者俗称(imperative programming/procedure programming)编程范式的程序设计语言，其关注点在于对解决问题的Solver的抽象，所以C语言更接近于汇编。

而C++与其设计理念不同，C++试图允许大家使用各种编程范式：      

- 比如首先支持使用C语言中的**面向过程**的编程范式。
- 比如支持**面向对象**的编程范式，使得我们可以开始抽象问题本身，通过定义抽象数据类型(Abstract Data Type)来抽象问题，封装问题对应的数据在类作用域内。然后我们可以合理地而不是滥用面向对象设计思想的封装/继承/    多态来使得我们写的代码更容易维护。
- 比如模板的引入，使得我们可以进行**泛型编程**；通过定义函数模板来利用函数参数列表的类型推导生成模板函数，通过定义类模板结合比如偏特化的技术来生成模板类，函数模板和类模板都是编译器看的东西，具体的代码中，只有我们产生的具体模板函数和模板类会占代码段的空间。当然模板的引入还让许多人可以进行**模板元编程**，让那些可以在编译时候计算的东西放在编译时进行，有人还证明了模板元编程是图灵完备的。
- 比如lambda表达式的引入，让我们进行**函数式编程**更加自然，从语言层面而不是库函数的层面来支持我们函数式编程。库层面的话只能使用函数指针/或者C++的函数对象，来进行函数式编程，并且函数一定会绑定有一个名字。有兴趣的朋友可以学一学[MIT Structure and Interpretation of Computer Programs](https://github.com/DeathKing/Learning-SICP)。

当然还有其他的内容，这里我仅仅根据我个人的理解，在这里抛砖引玉。感兴趣的朋友可以搜集资料查看，如果有重要内容补充的话，请进行评论。

(三/四) C/C++区别-重要的
---
---
**区别0，函数重载**：C语言不支持 [函数重载](http://en.cppreference.com/w/cpp/language/overload_resolution) ，其函数签名不包含参数列表。

因而在C语言中，也就不能够像C++那样 [运算符重载](http://en.cppreference.com/w/cpp/language/operators) 了。下面通过两个小代码来说明函数重载的区别，在c语言中我不得不取不同的函数名字来表达相近的操作加的意思。  

区别0，C代码：non_overload.c

```c
#include "stdio.h"
#include "stdlib.h"
#include "string.h"

int add_int(int lhs, int rhs) { return lhs + rhs; }

void add_str(const char *lhs, int lhs_size, const char *rhs, int rhs_size,
             char *res, int *res_size) {
  memcpy(res, lhs, lhs_size);
  memcpy(res + lhs_size, rhs, rhs_size);
  res[lhs_size + rhs_size] = '\0';
  *res_size = lhs_size + rhs_size;
}

int main() {
  printf("%d + %d = %d\n", 1, 2, add_int(1, 2));
  const char *lhs = "abcd";
  const char *rhs = "efgh";
  char *res = malloc(sizeof(char) * 20);
  int res_size = 0;
  add_str(lhs, 4, rhs, 4, res, &res_size);
  printf("%s + %s = %s, size:%d\n", lhs, rhs, res, res_size);
  free(res);
  return 0;
}
```   

区别0，C++代码：with_overload.cpp

```cpp
#include <cstring>
#include <iostream>
using namespace std;

int add(int lhs, int rhs) { return lhs + rhs; }

void add(const char *lhs, int lhs_size, const char *rhs, int rhs_size,
         char *res, int &res_size) {
  memcpy(res, lhs, lhs_size);
  memcpy(res + lhs_size, rhs, rhs_size);
  res[lhs_size + rhs_size] = '\0';
  res_size = lhs_size + rhs_size;
}

int main() {
  cout << "1 + 2 = " << add(1, 2) << endl;
  const char *lhs = "abcd";
  const char *rhs = "efgh";
  char *res = new char[20];
  int res_size = 0;
  add(lhs, 4, rhs, 4, res, res_size);
  cout << lhs << " + " << rhs << " = " << res << ", size:" << res_size;
  delete[] res;
  return 0;
}
```

---

---

**区别1，函数参数传递**：在C语言中不从语言层面支持传引用，需要通过指针类型的值传递实现传引用的功能。

详细的Demo代码如下，可以看到语法层面上的引用传递支持让代码看起来更舒服：

区别1，C代码：only_value_passing.c

```c
#include "stdio.h"

void increment(int *int_ptr) {
  printf("Before: %d\n", *int_ptr);
  *int_ptr += 1;
}

int main() {
  int integer = 0;
  increment(&integer);
  printf("After: %d\n", integer);
}
```

区别1，C++代码：ref_passing.cpp

```cpp
#include <iostream>
using namespace std;

void increment(int &int_ptr) {
  cout << "Before: " << int_ptr << endl;
  int_ptr += 1;
}

int main() {
  int integer = 0;
  increment(integer);
  cout << "After: " << integer << endl;
}
```

---

---

**区别2，函数的声明(Declaration)和使用**：在C语言中使用某个在其其他编译单元的函数可以不先声明(当然这是一个不好的编码习惯，在实际开发中要极力避免)，而在C++中这是不合法的。

下面通过代码来说明这一点，然后希望大家在使用的时候可以遵循好的编码规范：

区别2，C代码:function_declaration.c

```c
#include <stdio.h>
int main() {
  foo();
  return 0;
}

//虽然是一种不太好的编码风格
//一般好的C代码都是要求在使用之前进行函数声明
//打开 -Wall 编译器会警告我们不要这样写
void foo() { printf("Hello world"); }
```

区别2，C++代码:function_declaration.cpp
```cpp
#include <iostream>
using namespace std;

void foo() { cout << "Hello world" << endl; }

int main() { foo(); }
```

---

---

**区别3，模板**：模板在我的理解中，是生成代码的工具，但比C中的宏更安全更强大。

比如函数模板是生成模板函数的工具，本身不会占有代码段空间，而生成的模板函数则会占有代码段空间，这一点是相当有用的，我可不愿意让我的代码中有很多不会被调用的死代码(dead codes)。C语言不支持 [模板](http://en.cppreference.com/w/cpp/language/templates) ，因而不能轻松愉快地进行所谓的**泛型编程/模板元编程**，只能用容易出错，难以读懂的**宏定义**和**预处理元编程**。

在C++中模板主要有以下几种：**函数(function)模板，类(class)模板，变量(variable)模板(一种语法糖，让我们不用繁琐地写类模板)，别名(alias)模板**，具体可以参考 [Cppcon2016](https://github.com/CppCon/CppCon2016) 对应的Youtube上发布的有关 [Template Normal Programming Part1](https://www.youtube.com/watch?v=vwrXHznaYLA) 和 [Template Normal Programming Part2](https://www.youtube.com/watch?v=VIz6xBvwYd8) 的视频。这个作者用通俗易懂的例子给我们讲述了模板在Modern Cpp中的使用，我个人强力推荐大家进行观看，不过大陆的朋友观看Youtube需要翻墙。

下面贴一个知乎的一个高赞同的回答链接 [C++ 模板元编程的应用有哪些，意义是什么](https://www.zhihu.com/question/21656266) ，作者说主要应用有：编译时计算，补充类型系统，Domain Specific Language（是你说的“开发新语言”么？）。不过我个人不推荐大量地使用模板元编程，毕竟代码是写给人看的，而且有其他语言，可以帮助我们用更好的方式做这些事情。我个人对语言又是没有歧视的，哈哈。

我下面用代码展示下一个函数模板，说明一下C++模板的泛型设计，比C语言使用宏定义进行预处理元编程要更可读可写，并且不太容易出错。

区别3，C语言代码:pre_processing.h

```c
ElemType FuncName(ElemType lhs, ElemType rhs) { return lhs + rhs; }
```

区别3，C语言代码:pre_processing.c

```c
#include "stdio.h"

int main() {
//自己做编译器做的事情。。。
#undef ElemType
#undef FuncName
#define FuncName add_int
#define ElemType int
#include "pre_processing.h"
  printf("1 + 2 = %d\n", add_int(1, 2));

#undef ElemType
#undef FuncName
#define FuncName add_double
#define ElemType double
#include "pre_processing.h"
  printf("1 + 2 = %.2lf\n", add_double(1.0, 2));
}
```

区别3，C++代码，function_template.cpp

```cpp
#include <iostream>
using namespace std;
template <typename T> T add(T lhs, T rhs) { return lhs + rhs; }

int main() {
  cout << "1 + 2 =" << add(1, 2) << endl;
  cout << "1.0 + 2.0 = " << add(1.0, static_cast<double>(2)) << endl;
  cout << "1.0 + 2.0 = " << add<double>(1.0, 2) << endl;
}
```

---

---

**区别4，`struct/class/enum/enum class`**， 在讲述之前，我先提醒一下大家在C++中， `struct` 和 `class` 对编译器看来，只有默认的成员变量作用域不同，一个是public，另一个是private，然后其他没有区别；推荐一个知乎不错的回答 [c++为什么要让struct可以定义成员函数？](https://www.zhihu.com/question/49072961)，我引用一下里面的一些精华内容，从3点概括C++编译器可以做的“魔鬼操作”：

> 1、对普通成员函数，为它自动添加this参数，并在调用它时，自动把 obj.method() 转换成 method(obj)格式；并识别出函数中涉及的、没有显式使用this的成员变量、为它加上this。除此之外，别的什么都不用做。2、对虚函数，需要为继承链上的每个类产生一个全局结构体，在这个结构体里按次序安排指向该类所有虚函数的指针，这就是虚函数表；然后在类里添加一个指向属于自己的虚函数表的指针。那么，当用户调用某个对象的第N个虚函数时，到虚函数表查找并获取第N个函数指针指向的内容；然后类似调用普通成员函数一样，把 obj.method() 转换成 method(obj)格式，多态就实现了。3、 当然，除此之外，还要在编译时执行权限检查，避免非法访问类的protect/private成员（struct默认权限是public，class是private）；以及另外一些琐碎工作。

好了，有了这些预备知识之后，我再提醒一下大家，在C语言的`struct`中，我们是不可以定义方法的，我个人对这个的看法是，C语言的设计者应该认为`struct`，`union`和指定内存alignment的一些设计已经足够抽象出数据了，你要写一个solver就自己去写方法，如果你想要搞个函数对象也可以，把函数指针放到结构体里面去就好了。我觉得这很符合C遵循的面向过程(imperative programming/procedure programming)的编程范式，和从**充斥面向对象的cfront**发展而来的C++相比有很大不同。

然后值得注意的是，在C语言中，我们使用`strut`和`enum`定义出来的类型的时候，我们要加上`struct`；如果C++中是 `UserType var;` ，那么在C中就是 `struct UserType var;`。然后C++11引入了 `enum class` 让我们的枚举类型更类型安全，详细可以参考 [C++11 FAQ中enum class的内容](http://www.stroustrup.com/C++11FAQ.html#enum) 和 [一个不错的讲enum class的博客](http://www.cprogramming.com/c++11/c++11-nullptr-strongly-typed-enum-class.html) ，我就不写代码展示这些概念了，大家可以查看这两个链接的内容学习。

---

---

**区别5，面向对象**：C语言不支持**面向对象**，不包含面向对象中的对象语义学。

比如申请空间(可以是进程栈空间或是通过malloc出来的进程堆空间)后进行构造，然后在退出作用域的时候进行析构。因而，我们不能运用所谓的 [RAII(Resource Acquisition Is Initialization)机制](http://en.cppreference.com/w/cpp/language/raii) ，来管理资源(资源可以是堆空间，也可以是文件/网络连接之类)。在C语言中，我们也不能使用封装/继承/和基于虚表(virtual-table)的多态等面向对象特性，只能自己代替C++编译器，通过**函数指针**实现多态。

下面我用一个给大家展示一下C++的RAII机制:
区别5，Raii Demo C++代码：raii_demo.cpp

```cpp
#include <iostream>
using namespace std;
class ResourceManager {
public:
  ResourceManager() {
    cout << "Acquire Resource, e.g, heap memory allocation/file handle "
            "operation/other resources"
         << endl;
  }

  ~ResourceManager() {
    cout << "Release Resource, e.g, heap memory allocation/file handle "
            "operation/other resources"
         << endl;
  }
};

int main() { ResourceManager my_res_man; }
```

运行结果：

```zsh
Acquire Resource, e.g, heap memory allocation/file handle operation/other resources
Release Resource, e.g, heap memory allocation/file handle operation/other resources
```

下面我用简短的代码展示一下C++面向对象的多态，和C语言比较裸的**函数指针(也就是面向对象多态实现的基础)**的多态：
区别5，多态，C代码：polymorphism_demo.c

```c
#include "stdio.h"
#include "stdlib.h"
struct Polygon {
  int width_, height_;
  void (*area_ptr_)(struct Polygon *ptr_ploy);
};

struct Polygon *PolygonConstructor(int width, int height) {
  struct Polygon *ppoly = malloc(sizeof(struct Polygon));
  ppoly->width_ = width, ppoly->height_ = height;
  return ppoly;
}

void area_rectangle(struct Polygon *ptr_ploy) {
  printf("Rectangle Width:%d,Height:%d\n", ptr_ploy->width_, ptr_ploy->height_);
  printf("Rectangle Area:%d\n", ptr_ploy->width_ * ptr_ploy->height_);
}

struct Polygon *RectangleConstructor(int width, int height) {
  struct Polygon *ppoly = PolygonConstructor(width, height);
  ppoly->area_ptr_ = area_rectangle;
  return ppoly;
}

void area_triangle(struct Polygon *ptr_ploy) {
  printf("Rectangle Width:%d,Height:%d\n", ptr_ploy->width_, ptr_ploy->height_);
  printf("Rectangle Area:%d\n", ptr_ploy->width_ * ptr_ploy->height_ / 2);
}

struct Polygon *TriangleConstructor(int width, int height) {
  struct Polygon *ppoly = PolygonConstructor(width, height);
  ppoly->area_ptr_ = area_triangle;
  return ppoly;
}

int main() {
  struct Polygon *ppoly_rec = RectangleConstructor(4, 5);
  ppoly_rec->area_ptr_(ppoly_rec);
  free(ppoly_rec);

  printf("\n");

  struct Polygon *ppoly_tri = TriangleConstructor(4, 5);
  ppoly_tri->area_ptr_(ppoly_tri);
  free(ppoly_tri);
}
```

运行结果：

```zsh
Rectangle Width:4,Height:5
Rectangle Area:20

Rectangle Width:4,Height:5
Rectangle Area:10
```

区别5，多态，C++代码：polymorphism_demo.cpp
```cpp
//参考：http://www.cplusplus.com/doc/tutorial/polymorphism/
#include <iostream>
using namespace std;

class Polygon {
protected:
  int width_, height_;

public:
  Polygon(int width, int height) : width_(width), height_(height) {
    cout << "Polygon Constructing..." << endl;
  }
  virtual ~Polygon() { cout << "Polygon Destructing..." << endl; }
  virtual void area() = 0;
};

class Rectangle : public Polygon {
public:
  Rectangle(int width, int height) : Polygon(width, height) {
    cout << "Rectangle Constructing..." << endl;
  }
  virtual ~Rectangle() { cout << "Rectangle Destructing..." << endl; };
  void area() {
    cout << "Rectangle Width:" << width_ << ",Height" << height_ << endl;
    cout << "Rectangle Area:" << width_ * height_ << endl;
  }
};

class Triangle : public Polygon {
public:
  Triangle(int width, int height) : Polygon(width, height) {}
  virtual ~Triangle() { cout << "Triangle Destructing..." << endl; }
  void area() {
    cout << "Triangle Width:" << width_ << ",Height" << height_ << endl;
    cout << "Triangle Area:" << width_ * height_ / 2 << endl;
  }
};

int main() {
  Polygon *ppoly_rec = new Rectangle(4, 5);
  ppoly_rec->area();
  delete ppoly_rec;

  cout << endl << endl;

  Polygon *ppoly_tri = new Triangle(4, 5);
  ppoly_tri->area();
  delete ppoly_tri;
}

```

运行结果：

```zsh
Polygon Constructing...
Rectangle Constructing...
Rectangle Width:4,Height5
Rectangle Area:20
Rectangle Destructing...
Polygon Destructing...


Polygon Constructing...
Triangle Width:4,Height5
Triangle Area:10
Triangle Destructing...
Polygon Destructing...
```

---

---

**区别6，类型安全**：C语言一些操作在C++中被认为是**非类型安全**的，比如(void *)类型的数据使用。`void *` 是一种特殊的泛型指针(generic pointer)，详细可以看看 [FAQ > Casting malloc](http://faq.cprogramming.com/cgi-bin/smartfaq.cgi?id=1043284351&answer=1047673478) 里面介绍的内容。

下面的这句代码在C中可以编译：

```c
int *x = malloc(sizeof(int) * 10);
```

而在C++中则需要进行强制类型转换才可以:

```cpp
int *x = (int*) malloc(sizeof(int) * 10);
```

---

---

**区别7，内存管理**：C语言没有 `operator new/ operator delete` 和 `operator new[]/operator delete[]`，在全局作用域下也没有用户无法修改的函数 `placement new` 和 `placement delete`；在C语言中只有 `malloc` 和 `free` 这两个全局作用域的函数。

区别7，我就不用代码展示了，我给大家个链接，上面有代码，[C vs C++ - By Alex Allain](http://www.cprogramming.com/tutorial/c-vs-c++.html) 。

---

---

**区别8，Technical Specification提案(library上和language上)**：C++有更多的语言层面和库层面的提案。比如在C++标准委员会搞的这个 [IsoCpp-Current Status](https://isocpp.org/std/status) 上，我们就可以看到C++新标准将考虑支持的一些库层面和语言层面重要的提案，列举几个我比较关注的：Parallelism， Concurrency, Transactional Memory，File System, Networking，Concepts， Modules。这个内容比较复杂，感兴趣的朋友可以查阅链接看看。

(四/四) C/C++区别-其他
---
---
**区别9，命名空间**：C语言没有`namespace`的说法，所以很可能产生命名困难，因为变量都是global的作用域。

**区别10，关键词(语言特性)**：C++引入了一些关键字是C语言中没有的，比如说`class`, `template`, `bool`，C语言也有关键字是C++中没有的，比如说`restrict`。

**区别11，库**：C++有**更大的库**，比如说数学相关的库；比如说C++11引入的多线程有关的库`#include<thread>`；比如说C++11中 `#include<functional>` 中定义的 `std::move`， `std::forward`, `std::function` 等有用的库函数；还有很多，我不一一列举了。

**区别12，主函数返回值**：C语言主函数必须加上 `return 0;` ，而C++的编译器会自动给主函数加上 `return 0;`。

**区别...，还有很多其他区别**: 比如：**异常机制**，**I/O处理库**等等，在这里我就不一一列举了。

最后：示例代码Github链接
---
---
- [链接地址](https://github.com/CheYulin/CPP11Study/tree/master/DifferInCppC)
- 具体文件信息可以参看所给链接目录下 `ReadMe.md`中的说明

参考文章
---
---
* [mix-c-and-cpp - By Isocpp](https://isocpp.org/wiki/faq/mixing-c-and-cpp), 强力推荐，标准委员会出品
* [Learning C++ if you already know C - By Isocpp](https://isocpp.org/wiki/faq/c)，强力推荐，标准委员会出品
* [C++语言说明 -cppreference](http://en.cppreference.com/w/cpp/language)，推荐作为参考看看，cppreference出品
* [C语言说明 - cppreference](http://en.cppreference.com/w/c/language)，推荐作为参考看看, cppreference出品
* [C vs C++ - By Alex Allain](http://www.cprogramming.com/tutorial/c-vs-c++.html)，C和C++区别，Google搜到的
* [C/C++ - By Diffen](http://www.diffen.com/difference/C_vs_C%2B%2B)，C和C++区别，Google搜到的
* [10 major differences between C and C++ - By durofy](http://durofy.com/10-major-differences-between-c-and-c/)，C和C++区别，Google搜到的
* [What is the difference between C and C++? - By Cs Fundamentals](http://cs-fundamentals.com/tech-interview/c/difference-between-c-and-cpp.php)，C和C++区别，Google搜到的
* [What are the fundamental differences between C and C++? - Q&A in stack exchange](http://programmers.stackexchange.com/questions/16390/what-are-the-fundamental-differences-between-c-and-c)，C和C++区别，Google搜到的
* [15 Most Important Differences Between C And C++ - By Study Tips](http://studytipsandtricks.blogspot.hk/2012/05/15-most-important-differences-between-c.html)，C和C++区别，Google搜到的
