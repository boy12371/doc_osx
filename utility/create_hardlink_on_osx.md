# Mac：怎么在 Mac 下把百度网盘映射到另一个文件夹
---
背景：
例如百度云存储的路径是 /Users/Biao/Baidu/百度云同步盘，路径中有中文字符。
有一个 Qt 工程叫 QtTest 存储在 百度云同步盘 里面: /Users/Biao/Baidu/百度云同步盘/QtTest
因为 Qt 工程 QtTest 的路径中有中文字符，所以 QtCreator 是不能编译这个 Qt 工程的。

于是我们就想把这个含有中文的路径映射到一个没有中文的路径，例如 /Users/Biao/BaiduYun，
修改这两个目录中的任何一个目录里的数据，另一个目录会自动同步，但是路径却不一样。

很多不支持路径中有中文的软件也可以使用此方法解决。

如果在 Linux 上则很好解决，使用 mount --bind source_directory destination_directory 则就完成了。
但是在 Mac 上却不支持这个命令，不过我们可以用下面两个小程序来创建和删除文件夹的硬链接。

## 一、创建文件夹的硬链接

```c
#include <unistd.h>
#include <stdio.h>

// Build: gcc -o hlink hlink.c -Wall

int main(int argc, char *argv[]) {
    printf("Usage: assume the program name is hlink\n");
    printf("sudo ./hlink source_directory destination_directory\n");

    if (argc != 3) return 1;

    int ret = link(argv[1], argv[2]);

    if (ret == 0) {
        printf("\n");
        printf("****** Create hard link successful ******\n");
    } else {
        perror("link");
    }

    return ret;
}
```

### 1. 在终端里使用命令编译程序，生成可执行文件 hlink：
```shell
gcc -o hlink hlink.c -Wall
```

### 2. 创建 /Users/Biao/Baidu/百度云同步盘 的硬链接 /Users/Biao/BaiduYun 
(首先BaiduYun这个目录是不存在的，是在执行下面创建硬链接的命令后自动生成的)
```shell
./hlink /Users/Biao/Baidu/百度云同步盘 /Users/Biao/BaiduYun
```

## 二、删除文件夹的硬链接
```c
#include <unistd.h> 
#include <stdio.h>

// Build: gcc -o hunlink hunlink.c

int main(int argc, char *argv[]) {
    printf("Usage: assume the program name is hunlink\n");
    printf("sudo ./hunlink destination_directory\n");

    if (argc != 2) return 1;

    int ret = unlink(argv[1]);

    if (ret == 0) {
        printf("\n");
        printf("****** Delete hard link successful ******\n");
    } else {
        perror("unlink");
    }

    return ret;
}
```

### 1. 在终端里使用命令编译程序，生成可执行文件 hunlink：
```shell
gcc -o hunlink hunlink.c
```

### 2. 删除 /Users/Biao/Baidu/百度云同步盘 的硬链 /Users/Biao/BaiduYun 
（注意，这里的参数是/Users/Biao/BaiduYun，而不是源文件夹 /Users/Biao/Baidu/百度云同步盘）
```shell
./hunlink /Users/Biao/BaiduYun
```

[参考文档](http://www.cppblog.com/biao/archive/2013/09/09/203123.html)
---
另外windows下可以用符号链接来解决
mklink /d E:\workspace D:\Documents\百度同步盘
删除可以直接删除符号链接，不会影响原目录。