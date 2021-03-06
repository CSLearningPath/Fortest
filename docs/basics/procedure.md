# 从源代码到二进制

首先来看一个简单的程序

=== "hello.cpp"

    ```cpp
    #include <iostream>
    using namespace std;

    int main() {
        cout << "Hello World!" << endl;
        return 0;
    }
    ```

我们都知道，我们只需要运行 `g++ hello.cpp` 就可以将这个 C++ 源代码文件编译为可执行文件 `a.out` 了。其实，`g++` 只是一个驱动程序（Driver），它背后发生了一系列复杂的操作，调用了许多不同的程序。

## 预处理

在编译开始之前，预处理器会处理源文件中的宏定义（例如 `#!cpp #define`、`#!cpp #include`、`#!cpp #ifdef` 等），对源代码进行展开或者替换。例如，对于 `#!cpp #include <iostream>`，预处理器会直接将 `iostream` 这个文件原封不动地拷贝到源文件中。然而，我们的源代码目录中并没有 `iostream` 这个文件，预处理器是怎么知道去哪里查找这个文件呢？

实际上，预处理器并不知道我们的头文件都在哪里，需要我们手动提供包含路径（Include Path）。对于 `g++` 而言，就是 `-I` 选项。预处理器会在我们提供的路径中依次搜索，直到找到头文件为止。当然，像 `iostream` 这种常用的系统头文件，如果每次都需要我们手动指定查找目录，将会非常繁琐，所以 `g++` 已经内置了许多常用的包含路径，同时，包含路径默认还会包括源文件所在的目录。

这一步的输出是经过宏展开之后的源代码文件，并不是所有语言都像 C/C++ 一样有类似文本替换的预处理步骤。

???+note
    在 C/C++ 中，预处理程序是 `cpp`，全称是 **C** **P**re**P**rocessor。可以运行 `cpp hello.cpp` 查看预处理器的输出。输出可能比较长，可以将输出重定向到一个文件方便查看。

???+info
    使用 `-E` 选项，可以使 `g++` 只输出预处理结果而不进行进一步的编译。可以运行 `g++ -E hello.cpp` 查看输出，和直接运行 `cpp` 的结果比较。
### 试试看

下面我们来试试看如果包含路径不在默认提供的范围里，我们应当如何让预处理器找到文件。首先，创建两个文件夹，分别是 `src` 和 `include`。在里面分别放入以下文件：

=== "src/a.cpp"

    ```cpp
    #include "a.h"
    
    int main() {
        return a;
    }
    ```

=== "include/a.h"

    ```cpp
    int a = 1;
    ```

首先我们直接运行 `g++ -E src/a.cpp`，可以看到，这时候预处理器表示自己找不到 `a.h` :thinking:：

```bash
$ g++ -E src/a.cpp
# 1 "src/a.cpp"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 1 "<command-line>" 2
# 1 "src/a.cpp"
src/a.cpp:1:10: fatal error: a.h: No such file or directory
    1 | #include "a.h"
      |          ^~~~~
compilation terminated.
```

这时，就需要我们通过 `-I` 选项额外提供查找目录。这时候预处理器就知道去哪里找头文件了，并且 `a.h` 中的内容也被正确地复制进来了:laughing:。

```bash
$ g++ -E -Iinclude src/a.cpp
# 1 "src/a.cpp"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 1 "<command-line>" 2
# 1 "src/a.cpp"
# 1 "include/a.h" 1
int a = 1;
# 2 "src/a.cpp" 2

int main() {
    return a;
}
```
!!!note
    如果需要指定多个查找目录，需要提供多个 `-I` 选项，预处理器会依次查找。例如 `g++ -Iinclude1 -Iinclude2`。
## 编译

源文件经过预处理后，便可以调用编译器对源文件进行编译了。简单来讲，每一个 `.cpp` 文件可以被称为编译单元（Compilation Unit）。每一个编译单元都可以独立进行编译。编译器的工作是解析源文件，经过词法分析、语法分析、中间代码生成、代码优化等一系列复杂的操作，最终生成一个目标平台的汇编文件。

这一步的输出是汇编文件，不同平台所使用的汇编语言不一样，这里以 `x86-64` 平台为例。

???+note
    `g++` 所使用的 C++ 编译器为 `cc1plus`，这个程序并不在默认的 `PATH` 中，而是在库文件中。如果你使用的是 `g++` 9，那么它就在 `/usr/lib/gcc/x86_64-linux-gnu/9/cc1plus`。

### 试试看

我们首先将之前的 `hello.cpp` 进行预处理，保存到 `hello.preprocessed.cpp` 中，然后调用 `cc1plus` 将 `hello.preprocessed.cpp` 编译到汇编文件 `hello.preprocessed.s`。

???+note
    如果你找不到 `cc1plus`，可以使用 `-S` 选项，`g++` 会帮我们预处理好文件，然后编译，并输出一个汇编文件后便停止编译过程。也就是说，我们可以将下面两个命令用 `g++ -S hello.cpp -o hello.preprocessed.s` 代替。

```bash
$ cpp hello.cpp hello.preprocessed.cpp
$ /usr/lib/gcc/x86_64-linux-gnu/9/cc1plus hello.preprocessed.cpp
# ... 省略很多输出
Analyzing compilation unit
Performing interprocedural optimizations
 <*free_lang_data> <visibility> <build_ssa_passes> <opt_local_passes> <remove_symbols> <targetclone> <free-fnsummary>Streaming LTO
 <whole-program> <hsa> <fnsummary> <inline> <free-fnsummary> <single-use> <comdats>Assembling functions:
 <materialize-all-clones> <simdclone> int main() void __static_initialization_and_destruction_0(int, int) void _GLOBAL__sub_I_main()
Time variable                                   usr           sys          wall               GGC
 phase setup                        :   0.00 (  0%)   0.00 (  0%)   0.01 (  2%)    1471 kB (  5%)
 phase parsing                      :   0.19 ( 86%)   0.24 ( 77%)   0.48 ( 81%)   25530 kB ( 83%)
 phase lang. deferred               :   0.02 (  9%)   0.06 ( 19%)   0.08 ( 14%)    3548 kB ( 12%)
 phase opt and generate             :   0.01 (  5%)   0.01 (  3%)   0.02 (  3%)     289 kB (  1%)
 |name lookup                       :   0.00 (  0%)   0.05 ( 16%)   0.05 (  8%)    1463 kB (  5%)
 |overload resolution               :   0.02 (  9%)   0.04 ( 13%)   0.02 (  3%)    1788 kB (  6%)
 callgraph optimization             :   0.00 (  0%)   0.00 (  0%)   0.01 (  2%)       0 kB (  0%)
 ipa inlining heuristics            :   0.00 (  0%)   0.01 (  3%)   0.00 (  0%)       0 kB (  0%)
 preprocessing                      :   0.01 (  5%)   0.02 (  6%)   0.04 (  7%)     377 kB (  1%)
 parser (global)                    :   0.04 ( 18%)   0.07 ( 23%)   0.18 ( 31%)    8434 kB ( 27%)
 parser struct body                 :   0.04 ( 18%)   0.02 (  6%)   0.05 (  8%)    5710 kB ( 19%)
 parser function body               :   0.03 ( 14%)   0.02 (  6%)   0.00 (  0%)    1043 kB (  3%)
 parser inl. func. body             :   0.02 (  9%)   0.00 (  0%)   0.01 (  2%)    1395 kB (  5%)
 parser inl. meth. body             :   0.02 (  9%)   0.06 ( 19%)   0.11 ( 19%)    2446 kB (  8%)
 template instantiation             :   0.05 ( 23%)   0.11 ( 35%)   0.17 ( 29%)    9572 kB ( 31%)
 integrated RA                      :   0.01 (  5%)   0.00 (  0%)   0.00 (  0%)      72 kB (  0%)
 thread pro- & epilogue             :   0.00 (  0%)   0.00 (  0%)   0.01 (  2%)       4 kB (  0%)
 TOTAL                              :   0.22          0.31          0.59          30848 kB
$ ls
hello.cpp  hello.preprocessed.cpp  hello.preprocessed.s
```

可以看到 `cc1plus` 输出了编译后的汇编文件 `hello.preprocessed.s`。可以打开来查看其中的内容。

???+note "扩展知识 - C++ 命名混淆（Mangling）"

    在汇编语言文件中，你可能会看到类似 `_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_` 的奇怪符号。这是 C++ 编译器对函数进行命名混淆后的结果。C++ 中有函数重载，以及命名空间等 C 所没有的语言特性。为了实现这些特性，C++ 编译器会将函数的命名空间，以及调用参数的类型都编码到函数符号中，也就是相当于为每一个函数都创建了新的名字。这样就能区分开不同命名空间以及不同函数重载的同名函数了。

    需要注意的是，如何混淆一个函数的名称完全取决于编译器，C++ 标准中并没有规定编译器应当如何混淆函数名称。编译器只需要保证能够区分同名函数的不同重载即可。C++ 只规定了不允许只通过不同的返回值类型来重载函数，所以编译器**可能**会在混淆的时候省略返回值类型，也有可能不省略。

    系统中也自带了 `c++filt` 这个工具来帮助我们还原混淆符号对应的函数签名。我们运行 `c++filt` 后，输入混淆后的名称并回车，就能得到原始函数签名了。

    ```bash
    $ c++filt
    _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
    std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)
    _ZNSt8ios_base4InitC1Ev
    std::ios_base::Init::Init()
    ```

    可以看到编译器有时候会省略返回值类型有时候不会。
## 汇编

在得到汇编文件后，就可以使用汇编器将汇编文件汇编为目标文件了（Object File）。汇编器的工作简单很多，只需要翻译汇编文件中的指令，输出对应的二进制机器指令，并将指令组装为操作系统的目标文件格式即可。

这一步的输出是对应平台的二进制目标文件，其中包含了函数符号，以及对应的二进制机器码。

???+note
    汇编器的程序名字是 `as`。

### 试试看

我们使用 `as` 命令，将编译器输出的汇编文件汇编到目标文件。

```bash
$ as hello.preprocessed.s -o hello.preprocessed.o
$ ls
hello.cpp  hello.preprocessed.cpp  hello.preprocessed.o  hello.preprocessed.s
```

???+note
    使用 `-c` 选项，可以让 `g++` 只生成目标文件后，就停止编译过程。例如，我们可以使用 `g++ -c hello.cpp -o hello.preprocessed.o` 得到同样的目标文件。

???+note "使用 `file` 命令查看文件类型"
    我们可以使用 `file` 命令，来查看一个文件的类型。
    ```bash
    $ file hello.preprocessed.o
    hello.preprocessed.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
    ```
    可以看到这是一个 `ELF` 文件。

???+note "使用 `readelf` 命令查看一个 ELF 文件的信息"
    我们可以使用 `readelf` 命令，来查看一个 ELF 文件的文件信息。
    ```bash
    $ readelf -a hello.preprocessed.o
    ELF Header:
        Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
        Class:                             ELF64
        Data:                              2's complement, little endian
        Version:                           1 (current)
        OS/ABI:                            UNIX - System V
        ABI Version:                       0
        Type:                              REL (Relocatable file)
        Machine:                           Advanced Micro Devices X86-64
        Version:                           0x1
        Entry point address:               0x0
        Start of program headers:          0 (bytes into file)
        Start of section headers:          1832 (bytes into file)
        Flags:                             0x0
        Size of this header:               64 (bytes)
        Size of program headers:           0 (bytes)
        Number of program headers:         0
        Size of section headers:           64 (bytes)
        Number of section headers:         15
        Section header string table index: 14

    Section Headers:
        [Nr] Name              Type             Address           Offset
            Size              EntSize          Flags  Link  Info  Align
        [ 0]                   NULL             0000000000000000  00000000
            0000000000000000  0000000000000000           0     0     0
        [ 1] .text             PROGBITS         0000000000000000  00000040
            0000000000000091  0000000000000000  AX       0     0     1
        [ 2] .rela.text        RELA             0000000000000000  00000548
            0000000000000108  0000000000000018   I      12     1     8
        [ 3] .data             PROGBITS         0000000000000000  000000d1
    # ...省略大段输出
    ```

???+note "使用 `nm` 命令输出目标文件中的符号"
    我们可以使用 `nm` 命令，来查看一个二进制目标文件中包含哪些符号。
    ```bash
    $ nm hello.preprocessed.o
                     U __cxa_atexit
                     U __dso_handle
                     U _GLOBAL_OFFSET_TABLE_
    000000000000007c t _GLOBAL__sub_I_main
    0000000000000000 T main
    0000000000000033 t _Z41__static_initialization_and_destruction_0ii
                     U _ZNSolsEPFRSoS_E
                     U _ZNSt8ios_base4InitC1Ev
                     U _ZNSt8ios_base4InitD1Ev
                     U _ZSt4cout
                     U _ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_
    0000000000000000 r _ZStL19piecewise_construct
    0000000000000000 b _ZStL8__ioinit
                     U _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
    ```

## 链接

得到目标文件后，我们还不能执行这个文件。这是因为目标文件往往需要依赖本身没有的外部函数实现。如果我们使用 `nm` 命令查看我们的目标文件，就会发现很多 `U`，这就意味着这些符号在当前目标文件找不到定义。同时，为了执行程序，还有许多需要和操作系统交互的函数，例如分配内存、输入输出等。这些我们的目标文件通通都没有。为了能让目标文件中的程序运行起来，还需要最后一步：链接。

链接就是将许多目标文件合并起来，最终输出一个可执行的文件。

???+note
    链接器程序名字是 `ld`。

### 试试看

接下来你将看到本章最复杂的一条命令，不过不要害怕。

```bash
$ ld -static hello.preprocessed.o \
    /usr/lib/x86_64-linux-gnu/crt1.o \
    /usr/lib/x86_64-linux-gnu/crti.o \
    /usr/lib/gcc/x86_64-linux-gnu/9/crtbeginT.o \
    -L/usr/lib/gcc/x86_64-linux-gnu/9 \
    -L/usr/lib/x86_64-linux-gnu \
    -L/lib/x86_64-linux-gnu \
    -L/lib \
    -L/usr/lib \
    -lstdc++ \
    -lm \
    --start-group \
    -lgcc \
    -lgcc_eh \
    -lc \
    --end-group \
    /usr/lib/gcc/x86_64-linux-gnu/9/crtend.o \
    /usr/lib/x86_64-linux-gnu/crtn.o
$ ls
a.out  hello.cpp  hello.preprocessed.cpp  hello.preprocessed.o  hello.preprocessed.s
$ ./a.out
Hello, world
```

可以看到，经过复杂的链接过程，我们终于得到了熟悉的可执行文件。

???+note "`a.out` 的由来"
    `a.out` 这个名字是起源于古老的 Unix 系统，是 "assembler output" 的缩写，来自于 Ken Thompson 的 PDP-7 汇编器。`a.out` 也是一种二进制格式，由于欠缺比较多的功能，逐渐被 ELF、PE 等格式替代。早期 Linux 系统也支持 `a.out` 文件格式，现在的 Linux 系统常用的都是 ELF 格式的二进制，而 `a.out` 格式在 Linux 内核 5.1 版本中被正式移除支持。而 `a.out` 这个名字仍然被用来作为编译器产物的默认名称。更多故事可以参考[维基百科](https://en.wikipedia.org/wiki/A.out)。

调用 `ld` 命令，只需要将我们希望链接的目标文件一股脑传进去就可以了。而一些系统库并不是以 `.o` 的目标文件形式存放的，而是以 `.a` 文件形式存放的。`.a` 文件其实就是将许多 `.o` 文件拼起来的文件。`.a` 文件往往都以 `lib***.a` 的形式命名。如果我们传参 `-labc`，那么链接器就会去寻找 `libabc.a` 这个文件。这个相当于是一个捷径。

链接器和预处理器一样，并不知道 `.a` 文件都在哪，所以需要我们通过 `-L` 选项来指定库路径（Library Path）。配合上面的 `-l`，链接器就能找到 `.a` 文件并链接起来了。

一些链接器为了提高效率，只会保留参数列表前面的目标文件中未定义的函数符号，并试图在后面的目标文件中找到这些未定义符号。

比如说，我们有 `a.o`、`b.o` 和 `c.o` 三个目标文件，其中只有 `c.o` 存在未定义的函数符号，那么假如我们调用 `ld a.o b.o c.o` 将会出现链接错误，即使 `a.o` 和 `b.o` 中含有 `c.o` 需要的符号定义。而是需要调用 `ld c.o a.o b.o`。

当然，有时候我们不可避免地会写出循环引用的情况。例如 `a.o` 需要 `b.o` 的定义，`b.o` 需要 `c.o` 的定义，`c.o` 需要 `a.o` 的定义。此时我们无论如何调整链接顺序都不可能链接成功。所以，针对这种特殊情况，链接器提供了 `--start-group` 和 `--end-group` 这一对选项。位于这一对选项中的目标文件，链接器会不断地循环解决未定义符号，直到没有新的未定义符号可以被解决为止。当符号比较多的时候，可以想象这是一个很慢的过程，所以我们需要尽可能避免这种情况的发生。

还有一个问题，链接器怎么知道程序应该从哪开始执行？实际上 `main` 函数并不是程序的真正入口。默认情况下，`ld` 会寻找 `_start` 符号，作为程序的起点。而这个符号，就包含在系统运行库当中。库当中的 `_start` 函数，最终会在初始化运行环境后，调用我们的 `main` 函数。

???+note "使用 `g++` 进行链接"
    `g++` 调用时，会检查文件的类型，如果是目标文件，`g++` 会自动调用链接器。也就是说，直接调用 `g++ hello.preprocessed.o` 也可以得到最终的可执行文件。

???+note "查看可执行文件和目标文件的区别"
    ```bash
    $ readelf -h a.out
    ELF Header:
        Magic:   7f 45 4c 46 02 01 01 03 00 00 00 00 00 00 00 00
        Class:                             ELF64
        Data:                              2's complement, little endian
        Version:                           1 (current)
        OS/ABI:                            UNIX - GNU
        ABI Version:                       0
        Type:                              EXEC (Executable file)
        Machine:                           Advanced Micro Devices X86-64
        Version:                           0x1
        Entry point address:               0x404a90
        Start of program headers:          64 (bytes into file)
        Start of section headers:          2368032 (bytes into file)
        Flags:                             0x0
        Size of this header:               64 (bytes)
        Size of program headers:           56 (bytes)
        Number of program headers:         8
        Size of section headers:           64 (bytes)
        Number of section headers:         30
        Section header string table index: 29
    ```
    可以看到此时的 `Type` 变为可执行文件，同时多了一个 `Entry point address`，标志着程序的起点。

    我们使用 `nm` 命令，来查看这个地址对应的符号。

    ```bash
    $ nm a.out | grep 404a90
    0000000000404a90 T _start
    ```

    可以看到，程序入口并不是 `main` 函数，而是库所提供的 `_start` 函数。

## 不同操作系统的知识迁移

在不同的操作系统上，程序的编译链接过程是一样的，只是使用了不同的二进制存储格式，例如，Windows 使用 COFF 和 PE 格式存储库和可执行文件。不同的二进制格式虽然在细节上有所差别，但是保存的内容基本上都是一样的，例如程序的入口、程序的运行方式（比如 Windows 会在 PE 文件中标识一个程序是用图形界面运行还是命令行界面运行）、程序代码等。[这里](https://en.wikipedia.org/wiki/Comparison_of_executable_file_formats)是不同操作系统使用的二进制程序格式对比。

???+note "查看 Windows 的二进制信息"
    Windows 上可以使用 DUMPBIN 程序来查看一个可执行文件的信息。要使用这个工具，可以单独下载，也可以安装 Visual Studio，然后启动 Developer PowerShell for VS 2019。

    Windows 上的编译器是 `cl`，如果你用的是 Developer Powershell 的话，可以用这个命令编译程序。使用 `g++` 命令编译也可以。

    ```powershell hl_lines="39 52"
    > cl /EHsc .\hello.cpp
    Microsoft (R) C/C++ Optimizing Compiler Version 19.28.29337 for x86
    Copyright (C) Microsoft Corporation.  All rights reserved.

    hello.cpp
    Microsoft (R) Incremental Linker Version 14.28.29337.0
    Copyright (C) Microsoft Corporation.  All rights reserved.

    /out:hello.exe
    hello.obj
    > dumpbin /headers hello.exe
    Microsoft (R) COFF/PE Dumper Version 14.28.29337.0
    Copyright (C) Microsoft Corporation.  All rights reserved.
    
    
    Dump of file .\hello.exe
    
    PE signature found
    
    File Type: EXECUTABLE IMAGE
    
    FILE HEADER VALUES
                 14C machine (x86)
                   4 number of sections
            60FA3CB7 time date stamp Fri Jul 23 11:51:19 2021
                   0 file pointer to symbol table
                   0 number of symbols
                  E0 size of optional header
                 102 characteristics
                       Executable
                       32 bit word machine
    
    OPTIONAL HEADER VALUES
                 10B magic # (PE32)
               14.28 linker version
               1D000 size of code
               11600 size of initialized data
                   0 size of uninitialized data
                6905 entry point (00406905)
                1000 base of code
               1E000 base of data
              400000 image base (00400000 to 0042FFFF)
                1000 section alignment
                 200 file alignment
                6.00 operating system version
                0.00 image version
                6.00 subsystem version
                   0 Win32 version
               30000 size of image
                 400 size of headers
                   0 checksum
                   3 subsystem (Windows CUI)
                8140 DLL characteristics
                       Dynamic base
                       NX compatible
                       Terminal Server Aware
              100000 size of stack reserve
                1000 size of stack commit
              100000 size of heap reserve
                1000 size of heap commit
                   0 loader flags
                  10 number of directories
                   0 [       0] RVA [size] of Export Directory
               2B34C [      28] RVA [size] of Import Directory
                   0 [       0] RVA [size] of Resource Directory
                   0 [       0] RVA [size] of Exception Directory
                   0 [       0] RVA [size] of Certificates Directory
               2E000 [    1E84] RVA [size] of Base Relocation Directory
               2989C [      1C] RVA [size] of Debug Directory
                   0 [       0] RVA [size] of Architecture Directory
                   0 [       0] RVA [size] of Global Pointer Directory
                   0 [       0] RVA [size] of Thread Storage Directory
               298B8 [      40] RVA [size] of Load Configuration Directory
                   0 [       0] RVA [size] of Bound Import Directory
               1E000 [     128] RVA [size] of Import Address Table Directory
                   0 [       0] RVA [size] of Delay Import Directory
                   0 [       0] RVA [size] of COM Descriptor Directory
                   0 [       0] RVA [size] of Reserved Directory

    SECTION HEADER #1
       .text name
       1CFD8 virtual size
        1000 virtual address (00401000 to 0041DFD7)
       1D000 size of raw data
         400 file pointer to raw data (00000400 to 0001D3FF)
           0 file pointer to relocation table
           0 file pointer to line numbers
           0 number of relocations
           0 number of line numbers
    60000020 flags
             Code
             Execute Read
    
    # ... 省略
    
      Summary
    
            2000 .data
            E000 .rdata
            2000 .reloc
           1D000 .text
    ```

    程序有类似的 Entry Point 地址，还有 Windows 上标识一个程序是不是图形界面程序的标志。这里 CUI 表示这个程序是 Console UI，也就是控制台程序。

    我们可以看看记事本程序的标志：

    ```powershell hl_lines="41"
    > dumpbin /headers C:\Windows\System32\notepad.exe
    Microsoft (R) COFF/PE Dumper Version 14.28.29337.0
    Copyright (C) Microsoft Corporation.  All rights reserved.
    
    
    Dump of file C:\Windows\System32\notepad.exe
    
    PE signature found
    
    File Type: EXECUTABLE IMAGE
    
    FILE HEADER VALUES
                8664 machine (x64)
                   7 number of sections
            86FCBD69 time date stamp
                   0 file pointer to symbol table
                   0 number of symbols
                  F0 size of optional header
                  22 characteristics
                       Executable
                       Application can handle large (>2GB) addresses
    
    OPTIONAL HEADER VALUES
                 20B magic # (PE32+)
               14.20 linker version
               24A00 size of code
                E200 size of initialized data
                   0 size of uninitialized data
               23DB0 entry point (0000000140023DB0)
                1000 base of code
           140000000 image base (0000000140000000 to 0000000140037FFF)
                1000 section alignment
                 200 file alignment
               10.00 operating system version
               10.00 image version
               10.00 subsystem version
                   0 Win32 version
               38000 size of image
                 400 size of headers
               36BB0 checksum
                   2 subsystem (Windows GUI)
                C160 DLL characteristics
                       High Entropy Virtual Addresses
                       Dynamic base
                       NX compatible
                       Control Flow Guard
                       Terminal Server Aware
               80000 size of stack reserve
               11000 size of stack commit
              100000 size of heap reserve
                1000 size of heap commit
                   0 loader flags
                  10 number of directories
    # ...
    ```

    记事本不出意外的是 GUI 程序了。如果你好奇有什么其他的 Subsystem，可以看[微软的官方文档](https://docs.microsoft.com/en-us/cpp/build/reference/subsystem-specify-subsystem)。

## 总结

至此，发生在 `g++` 背后的故事你已经简单地有了一个概念。理解这些过程以及原理是构建更大项目的基础。
