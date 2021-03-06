# 快速上手项目

在了解一个新的项目的时候，如果没有多少经验的话往往会被庞大的代码库所震撼而感到不知所措，但毕竟项目也是人写出来的，不用担心理解不了的问题，只是需要找到一个正确的方法来快速上手。

## 尝试编译

了解一个项目之前，最好先按照文档尝试搭建开发环境，然后尝试编译整个项目。在这个过程中，你就可以大致了解到项目使用的开发工具、开发时依赖以及运行时依赖，从而对项目的外部环境有一定了解。另外，一些项目可能会需要根据编译环境来生成配置头文件，也有一些代码可能需要生成，将项目编译一次可以确保将需要的文件都生成完毕。

## 配置 IDE

对于大型项目而言，一个好用的 IDE 必不可少，IDE 强大的代码分析功能可以帮我们快速分析项目结构，查找类、函数的使用和定义等，对于阅读代码也非常有帮助。如果使用 CMake 的话，可以要求生成 `compile_commands.json` 来提供智能提示所必要的编译信息，如宏定义以及平台相关的函数等。当然，也可以使用 `Visual Studio` 生成器，来生成 Visual Studio 项目。

## 运行程序

当开发环境搭建完成，以及检查项目能够正常编译之后，就需要尝试运行其中的代码。对于服务器项目，例如 MySQL、TiDB 等，往往会编译出来一个可执行文件，只需要根据文档，配置好运行环境后检查程序能否正常运行即可。对于库项目，例如 rocksdb 等，则往往会编译出一个库文件，这时候是没有办法直接执行的。但是，库项目一般都会带有测试（test）或者示例程序（example），在编译的时候往往也会将它们编译出来，我们这时候可以尝试运行这些测试或者示例程序来检查库是否正常。

## 阅读源码

一般而言，大型软件项目往往包含了错综复杂的多种模块，如果直接扎进去读源码的话肯定是会迷失在代码的海洋中，而且收效甚微。一般而言，读源码的时候需要带着一个特定的目的，例如你需要修复某个模块的 Bug，或者你想了解某一个功能是如何实现的。对于大型项目而言，往往会使用到运行时多态的功能，光是阅读静态的代码可能很难理解，因此，推荐你使用 IDE 的调试器功能来运行程序/测试/示例，在你感兴趣的函数中下断点，或者让调试器在程序入口处暂停，然后一步步地运行，来帮助你了解程序的执行过程以及中间状态。

当然，如果是多线程程序或者你懒得使用调试器，可以打开 `Debug` 级别的日志，使用输出大法来调试程序。