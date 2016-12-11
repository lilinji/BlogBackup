前言
---
---
开发环境，实验/测试/性能分析环境，轮子和开发工具都只是工具而已，与工程/学术能力关系不太大，但是了解基本的这些东西，可以让我们的工作/科研/学习生活更轻松有趣，哈哈 :smile: 。

开发环境
---
---
操作系统我喜欢用 [Fedora](http://ftp.cuhk.edu.hk/pub/Linux/fedora/releases/23/Workstation/x86_64/iso/) 。我喜欢用Linux作为工作环境主要是因为：其方便的包管理器和更简洁的内核设计，比学windows内核的代价要小很多，配置开发环境也更容易。

写C++小程序，编辑器可以使用 [Atom](https://github.com/atom/atom) ，加上各种插件，内置terminal的集成，可以各种语法高亮（包括Cmake，C++），基于 [clang](http://clang.llvm.org/) 的静态代码检查，支持分多个Panel比较代码。

写C++项目，Linux下用一款 [Jetbrain](https://www.jetbrains.com/) 公司推出的 [Clion](https://www.jetbrains.com/clion/), 然后用 [Cmake](https://cmake.org/) 作为Build工具，来生成对应的Makefile，编译器用gcc5.x或6.x。Clion有很多插件可以安装，挺方便的。

如果项目需要用 [Cuda](https://developer.nvidia.com/cuda-downloads)，尽管 Clion 对 Cuda 集成不太好；但是我们可以用 Nvidia 在 Eclipse 基础上的 [Nsight](http://www.nvidia.com/object/nsight.html)(在装了cuda toolkit自带的) ，有一点好处是可以用它来进行程序的profiling。

实验/测试/性能分析环境
---
---
Terminal的shell使用 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) ，然后可以装个 [the-fuck](https://github.com/nvbn/thefuck) , 用起来比较爽。连学校(HKUST)Itsc管理的Gpu或者Mic服务器跑实验，直接ssh就可以了，都装的CentOS。

调试C++程序，我还是用gdb。Clion制作的基于gdb进行Debug的Gui也挺不错的，方便了我们Debug，[相关视频链接](https://www.youtube.com/watch?v=wUZyoAnPdCY&t=1s)。

C++程序性能Profiling，用 [Intel Vtune Amplifier](https://software.intel.com/en-us/intel-vtune-amplifier-xe) 来看cache miss, memory bottleneck分析，不过只有一个月试用期，如果过期需要重装。

写完C++程序，然后打开 [Pycharm](https://www.jetbrains.com/pycharm/) 写些Python(先装上 [Anaconda](https://www.continuum.io/downloads) 和 [Pip](https://pypi.python.org/pypi/pip) )来跑跑实验，处理文件，画图。当然Python的强大社区证明了python不仅仅只有这些用处，只不过有些用途我还没有接触。

轮子和开发工具
---
---
记得利用的基本库，C++14标准定义的STL，最新版本Boost，然后有什么其他需要的组件可以在Github上搜，这个链接总结了许多，[awesome-cpp](https://github.com/fffaraz/awesome-cpp) 。

其他主要还有一些并行计算的基于 Pthread 封装或者基于 Gpu模型 封装的库会用到，虽然我现在还没怎么用，比如 Intel TBB ， Nvidia ModernGPU 等，具体的话可以参考下 [我写的高性能程序开发库总结](https://github.com/CheYulin/Primitives-and-Graph-Processing-on-GPU)。

其他
---
---
写C++程序的时候，代码的版本管理靠git, 并且使用下学生的无限免费私有仓库。

跟老板汇报工作进度前，可以用Markdown写点总结放Github私有仓库上。


:smile: 哈哈就这么多，平时常用的东西。
