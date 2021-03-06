# 程序的运行配置

服务器程序通常带有大量的参数需要灵活配置，尽管我们可以通过 `main` 函数的 `argv` 传参，还是稍显麻烦。因此，服务器程序一般需要支持从文件等多种途径读取配置文件启动。另外，我们还需要给不同的配置方法赋予不同的优先级，以便灵活覆盖。通常而言，服务器程序要支持以下几种配置方式（优先级从高到低），优先级高的配置方式，会覆盖掉优先级更低的配置方式的同名参数。

- 命令行传参
- 环境变量配置
- 文件配置
- 远端配置中心拉取
- 写死在程序中的默认值

其中，命令行传参一般用于在调试的时候快速覆盖某些参数而不必修改配置文件。

环境变量同样用于快速覆盖某些参数，但可以避免冗长的命令行参数，另外，通过环境变量配置程序也是容器化部署服务时常用的配置方法。

文件配置通常用于程序正式上线后稳定的程序配置。一般需要支持在多个目录中搜索配置文件，不同目录同样具有优先级，不同的是，配置文件一般只会读取优先级最高的那个文件。使用文件的好处一个是方便查看和修改，另一方面程序可以在运行的时候检测配置文件的变化，这样修改配置文件后不需要重新启动程序就能使新的配置生效，使得配置修改对线上服务影响最小。一般来说搜索配置目录的优先级从高到低是：

- 命令行指定的配置文件
- 命令行参数指定的多个搜索目录
- 当前程序运行的目录
- 每个用户独立的配置目录（例如 $HOME/.myserver/config）
- 系统级别的配置目录（例如 /etc/myserver/config）

远端配置中心拉取则是在分布式系统中，由统一的配置中心服务负责下发配置，保证每个服务器程序使用的都是相同的配置。和文件配置一样，我们的程序可以监听配置中心配置的变化，或者由配置中心主动下发新配置，实现配置的重载。另外，除了服务器程序自己拉取以外，还有一种方式是使用独立的程序负责下载配置，然后写入到服务器程序配置文件的搜索目录里，这样可以更好地解耦。

为了方便使用，一般还会在编译时，就把一些参数编译到程序中作为最后的默认值。这些参数通常会以宏定义或常量方式写在程序里，用宏定义的参数可以很方便的在编译的时候覆盖新的值。

## 读取命令行配置

我们先从最简单的做起。我们都知道 `main` 函数的完整形式是 `#!cpp int main(int argc, char* argv[])`，我们可以从 `argv` 这个参数获取到启动程序的所有命令行参数。最简单的做法就是我们规定不同的参数应该放在第几个位置，然后我们直接读取即可。但是这样很不灵活，也无法处理可选参数的情况。

我们常见到的传参方法是 `--arg val` 或者使用缩写形式 `-a val`，我们希望解析得到一个 `map<string, string>` 方便后续的使用。另外，不同参数的类型可能不一样，有的参数是数字型，有的参数是字符串，还有的可能是数组。尽管编写一个解析命令行的函数并不困难，但是需要考虑比较多的细节。一般来说，解析命令行参数可以使用现成的库。

下面以 `#!cpp boost::program_options` 为例，演示如何使用现成的库来解析命令行参数并使用。

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <string>

using namespace std;

int main(int argc, char *argv[]) {
  namespace po = boost::program_options;
  po::options_description desc("MyServer 1.0");
  desc.add_options()("help", "Show usage")
      ("id", po::value<string>()->required(), "id of this server")
      ("port", po::value<int>()->value_name("port"),
        "listen on which port")
      ("host",
        po::value<string>()->value_name("host")->default_value(
            "127.0.0.1"),
        "bind to which address");

  po::variables_map vm;
  po::store(po::parse_command_line(argc, argv, desc), vm);
  po::notify(vm);

  if (vm.count("help")) {
    cout << desc << "\n";
    return 1;
  }

  if (vm.count("port")) {
    cout << "port was set to " << vm["port"].as<int>() << ".\n";
  } else {
    cout << "port was not set.\n";
  }

  return 0;
}
```

???+note "为什么不使用 `getopt`"
    `getopt` 函数同样提供了一定的命令行解析功能，但只在 Linux 上可用，为了更好的跨平台性，尽量使用其他的库函数。

## 环境变量配置

除了命令行，我们还可以使用环境变量来配置程序。`#!cpp boost::program_options` 同样提供了解析环境变量的功能。

```cpp hl_lines="7-13 29"
#include <boost/program_options.hpp>
#include <iostream>
#include <string>

using namespace std;

std::string mapper(std::string env_var) {
  std::transform(env_var.begin(), env_var.end(), env_var.begin(), ::toupper);

  if (env_var == "PORT")
    return "port";
  return "";
}

int main(int argc, char *argv[]) {
  namespace po = boost::program_options;
  po::options_description desc("MyServer 1.0");
  desc.add_options()("help", "Show usage")
      ("id", po::value<string>()->required(), "id of this server")
      ("port", po::value<int>()->value_name("port"),
        "listen on which port")
      ("host",
        po::value<string>()->value_name("host")->default_value(
            "127.0.0.1"),
        "bind to which address");

  po::variables_map vm;
  po::store(po::parse_command_line(argc, argv, desc), vm);
  po::store(po::parse_environment(desc, boost::function1<std::string, std::string>(mapper)), vm);
  po::notify(vm);

  if (vm.count("help")) {
    cout << desc << "\n";
    return 1;
  }

  if (vm.count("port")) {
    cout << "port was set to " << vm["port"].as<int>() << ".\n";
  } else {
    cout << "port was not set.\n";
  }

  return 0;
}
```

???+note "使用 `getenv` 函数"
    `getenv` 函数是 C 标准库 `stdlib.h` 中的一个函数，是跨平台的，如果不想引入其他库，使用这个函数也是可以的。

## 文件配置

以上两种方式可以实现简单的配置效果，不过由于只能使用 Key-Value 的形式，在面对比较复杂的配置时难免会有些麻烦。使用文件配置的话，可以很方便的表达嵌套等复杂的结构，甚至实现一定的预处理效果，例如 `include` 等。

使用配置文件的话，一般有几种选择。

- 使用 `toml`、`json`、`yaml` 等常见的嵌套格式
- 使用 `properties` 格式，和 Key-Value 格式差不多
- 使用 `ini` 格式，在 Windows 上比较常见
- 像 `nginx` 一样，自己拟定一个格式，并编写解析器

如果没有特殊需求，一般采用现成的格式即可。使用现成的格式还有一个好处，那就是不需要自己编写解析库，可以使用现成的解析库。不同生态使用的配置文件格式有不同偏好，例如 `Go` 语言和 `Rust` 语言喜欢用 `toml` 格式。对于人类而言，`toml` 以及 `ini` 格式比较容易写，`yaml` 通过缩进来指示嵌套容易在复制粘贴时出错，而 `json` 需要匹配括号比较麻烦。`#!cpp boost::program_options` 支持从文件读取类似 `ini` 的配置。

```cpp hl_lines="30"
#include <boost/program_options.hpp>
#include <iostream>
#include <string>

using namespace std;

std::string mapper(std::string env_var) {
  std::transform(env_var.begin(), env_var.end(), env_var.begin(), ::toupper);

  if (env_var == "PORT")
    return "port";
  return "";
}

int main(int argc, char *argv[]) {
  namespace po = boost::program_options;
  po::options_description desc("MyServer 1.0");
  desc.add_options()("help", "Show usage")
      ("id", po::value<string>()->required(), "id of this server")
      ("port", po::value<int>()->value_name("port"),
        "listen on which port")
      ("host",
        po::value<string>()->value_name("host")->default_value(
            "127.0.0.1"),
        "bind to which address");

  po::variables_map vm;
  po::store(po::parse_command_line(argc, argv, desc), vm);
  po::store(po::parse_environment(desc, boost::function1<std::string, std::string>(mapper)), vm);
  po::store(po::parse_config_file("config.ini", desc), vm);
  po::notify(vm);

  if (vm.count("help")) {
    cout << desc << "\n";
    return 1;
  }

  if (vm.count("port")) {
    cout << "port was set to " << vm["port"].as<int>() << ".\n";
  } else {
    cout << "port was not set.\n";
  }

  return 0;
}
```