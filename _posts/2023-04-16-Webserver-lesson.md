---
title: WebServer learning
author: Huyang
date: 2023-04-16 23:05:00 +0800
categories: [Blogging, coding]
tags: [C++]
math: true
mermaid: true
---

# webserver项目

## **Table of Contents**

this unordered seed list will be replaced by toc as unordered list
{:toc} 

## 远程连接linux流程

#### Xshell连接Linux

#### VSCode连接Linux

## C++异常处理

[C++异常处理（try catch throw）完全攻略](http://c.biancheng.net/view/422.html)

程序运行时常会碰到一些异常情况，例如：

- 做除法的时候除数为 0；
- 用户输入年龄时输入了一个负数；
- 用 new 运算符动态分配空间时，空间不够导致无法分配；
- 访问数组元素时，下标越界；打开文件读取时，文件不存在。

这些异常情况，如果不能发现并加以处理，很可能会导致程序崩溃。

- 鉴于上述原因，C++引入了**异常处理机制**。其基本思想是：函数 A 在执行过程中发现异常时可以不加处理，而只是“拋出一个异常”给 A 的调用者，假定为函数 B。  
- 拋出异常而不加处理会导致函数 A 立即中止，在这种情况下，函数 B 可以选择捕获 A 拋出的异常进行处理，也可以选择置之不理。如果置之不理，这个异常就会被拋给 B 的调用者，以此类推。  如果一层层的函数都不处理异常，异常最终会被拋给最外层的 main 函数。main 函数应该处理异常。如果main函数也不处理异常，那么程序就会立即异常地中止
- 
- C++ 通过 throw 语句和 try...catch 语句实现对异常的处理。

```cpp
throw  表达式;
```

该语句拋出一个异常。异常是一个表达式，其值的类型可以是基本类型，也可以是类。

```cpp
try {  
    语句组  
}  
catch(异常类型) {  
    异常处理代码  
}  
```

try...catch 语句的**执行过程**是：

- 执行 try 块中的语句，如果执行的过程中没有异常拋出，那么执行完后就执行最后一个 catch 块后面的语句，所有 catch 块中的语句都不会被执行；
- 如果 try 块执行的过程中拋出了异常，那么拋出异常后立即跳转到第一个“异常类型”和拋出的异常类型匹配的 catch 块中执行（称作异常被该 catch 块“捕获”），执行完后再跳转到最后一个 catch 块后面继续执行。

## gcc/g++

- g++相比gcc只是多了C++程序的链接，gcc无法与C++库文件关联
- -D 指定一个宏 可以使用这种方式输出一些log信息，进行调试

## 静态库

#### 生成流程

1. 将要包含的库文件生成可执行文件.o
2. 使用 ar rcs libxxx.a xxx.o xxx.o 将可执行文件生成为库，并添加声明头文件
3. 利用gcc main.c -o app -I ./include/  -L  ./lib/ 即可生成成功
    ![Pasted image 20230331182552|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230331182552_21-21-07.png)
- 使用静态库需要给定头文件，头文件包括静态库函数的声明，例如：
    ![Pasted image 20230329212532|300](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230329212532_21-25-17.png)
- 静态文件库中的文件需要给出这些函数声明的具体实现

## 动态库

![Pasted image 20230330155259|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230330155259_21-25-44.png)

- -fpic 用于编译阶段，产生的代码没有绝对地址，全部用相对地址，这正好满足了共享库的要求，共享库被加载时地址不是固定的。
- 如果不加-fpic ，那么生成的代码就会与位置有关，当进程使用该.so文件时都需要重定位，且会产生成该文件的副本，每个副本都不同，不同点取决于该文件代码段与数据段所映射内存的位置。

动态库的生成过程：
![Pasted image 20230331182638|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230331182638_21-26-04.png)

![Pasted image 20230330161843|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230330161843_21-26-17.png)

elf加载的时机：当程序加载到调用动态库的API时

#### 动态库的配置方式：

<font color="#f79646">一、修改环境变量LD_LIBRARY_PATH</font>
1、在终端设置LD_LIBRARY_PATH（临时的，仅在单个终端有用）
2、用户级配置：在系统隐藏文件bashrc中设置LD_LIBRARY_PATH（永久）
3、系统级配置：在系统文件profile设置LD_LIBRARY_PATH 
    1. sodu vim /etc/profile
    2. 在最后一行加上export LD_LIBRARY_PATH=&LD_LIBRARY_PATH：+（动态库地址）
    3. sudo source /etc/profile
<font color="#f79646">二、修改/etc/ld.so.cache文件列表</font>
1、在系统目录下找到 ./etc/ld.so.cache
2、wim ./etc/ld.so.conf ，在文件下方加入文件路径
3、更新 sudo ldconfig
<font color="#f79646">三、将动态库文件放到/lib/和/usr/lib目录下</font>（不推荐使用）

#### 静态库和动态库的区别

- 静态库
  - 优点：
    - 加载速度相比于动态库更快
    - 发布程序无需提供静态库，移植方便
  - 缺点：
    - 消耗系统资源、浪费内存
    - 更新、部署、发布麻烦
      ![Pasted image 20230331184450|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230331184450_11-40-19.png)
- 动态库
  - 优点
    - 可以实现进程间资源共享（共享库）
    - 更新、部署、发布简单
    - 可以控制何时加载动态库
  - 缺点：
    - 加载速度比静态库慢
    - 发布程序时需要提供依赖的动态库
      ![Pasted image 20230331184623|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230331184623_11-40-40.png)

## Makefile

- Makefile定义了一系列规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更为复杂的功能操作
- 目的：实现“自动化编译”
- 文件命名：makefile/Makefile
- **规则**
  - 一个makefile文件可以有一个或多个规则
    - 目标 . . . ：依赖 . . .
        命令（Shell 命令）
        . . .
    - 目标：最终要生成的文件（伪目标除外）
    - 依赖：生成目标所需要的文件或是目标
    - 命令：通过执行命令对依赖操作生成目标（命令前必须Tab缩进）
  - makefile中的其他规则一般都是为第一条规则服务的
- **工作原理**
  - 命令在执行之前，需要先检查规则中的依赖是否存在（默认执行第一条规则，如果下面规则和第一条规则无关则不会执行，想要执行下面的规则需要使用 make+目标名称 指定）  
    - 如果存在，执行命令  
    - 如果不存在，向下检查其它的规则，检查有没有一个规则是用来生成这个依赖的，  如果找到了，则执行该规则中的命令
  - 检测更新，在执行规则中的命令时，会比较目标和依赖文件的时间的新旧程度
    - 如果依赖的时间比目标的时间晚，需要重新生成目标  
    - 如果依赖的时间比目标的时间早，目标不需要更新，对应规则中的命令不需要被执行
- **常用变量**
  - 自定义变量
    - 变量名 = 变量值
  - 预定义变量
    - AR：归档维护程序的名称，默认为ar
    - CC：C编译器的名称，默认为gcc
    - CXX：C++编译器的名称，默认为g++
  - 系统自带变量（自动变量，只能在规则的命令中使用）
    - $@：获取目标的完整名称
    - $<：第一个依赖文件的名称
    - $^：所有的依赖文件
    - 举例：$(CC) -c $^ -o $@ 将所有,c文件编译成目标文件
  - 获取变量的值
    - $（变量名）
- **模式匹配**
  - % . o : % . c
    - %：通配符，匹配一个字符串
    - 该式中两个%匹配的是同一个字符串
- **常用函数**
  - $(wildcard PATTERN . . .)
    - 功能：获取指定目录下指定类型的文件类型
    - 参数：PATTREN 指的是某个或多个目录下对应的某种类型的文件，如果有多个目录，一般用空格隔开
    - 返回：得到若干个文件的文件列表，文件名之间使用空格间隔
    - 示例：
      - $(wildcard  * .c  ./sub/* . c)
      - 返回值：a.c b.c c.c d.c e.c f.c
  - $(patsubst < pattern > , < replacement > , < text >)  
    - 功能：查找< text >中的单词(单词以“空格”、“Tab”或“回车”“换行”分隔)是否符合模式< pattern >，如果匹配的话，则以< replacement >替换。
    - < pattern >可以包括通配符`%`，表示任意长度的字串。如果< replacement >中也包含`%`，那么，< replacement >中的这个`%`将是< pattern >中的那个% 所代表的字串。(可以用`\`来转义，以`\%`来表示真实含义的`%`字符)  
    - 返回：函数返回被替换过后的字符串  
    - 示例：  
      - $(patsubst %.c, %.o, x.c bar.c) 
      - 返回值格式: x.o bar.o
- **伪目标**
  - 作用：执行不会生成文件的命令
  - 在目标前面加上 .PHONY：+目标名称

## GDB

- GDB是GNU软件系统社区提供的调试工具，同GCC配套组成了一套完整的开发环境，GDB是Linux和众多Unix系统中的标准开发环境

- 功能
  
  - 启动程序。可以按照自定义的要求随心所欲地运行程序
  - 可以让被调试的程序在所指定的调试的断点停住（断点可以是条件表达式）
  - 当程序被停住时，可以检查此时程序中发生的事
  - 可以改变程序，将一个BUG产生的影响修正从而测试其他BUG

- 准备工作
  
  - 通常在为调试而编译时通常会关掉编译器的优化选项“-o”，并打开调试选项（`-g`）。另外，`-Wall`在尽量不影响程序行为的情况下选项打开所有warning，也可以发现许多问题，避免一些不必要的 BUG。  
  - gcc -g -Wall program.c -o program  
  - `-g` 选项的作用是在可执行文件中加入源代码的信息，比如可执行文件中第几条机器指令对应源代码的第几行，但并不是把整个源文件嵌入到可执行文件中，所以在调试时必须保证 gdb 能找到源文件。

- 常用命令
  
  - 启动和退出
    
    - gdb 可执行程序  
    - quit/q 退出GDB
  
  - 给程序设置参数/获取设置参数  
    
    - set args 10 20  
    - show args  
  
  - GDB 使用帮助  
    
    - help  
  
  - 查看当前文件代码（显示当前指定的上下五行）
    
    - list/l （从默认位置显示）
    - list/l 行号 （从指定的行显示）
    - list/l 函数名（从指定的函数显示）  
  
  - 查看非当前文件代码
    
    - list/l 文件名:行号
    - list/l 文件名:函数名  
  
  - 设置显示的行数  
    
    - show list/listsize（查看当前设置行数）
    - set list/listsize 行数（设置显示行数）
  
  - 设置断点
    
    - b/break 行号
    - b/break 函数名
    - b/break 文件名:行号
    - b/break 文件名:函数  
  
  - 查看断点  
    
    - i/info b/break  
  
  - 删除断点
    
    - d/del/delete 断点编号  
  
  - 设置断点无效  
    
    - dis/disable 断点编号  
  
  - 设置断点生效  
    
    - ena/enable 断点编号  
  
  - 设置条件断点（一般用在循环的位置）  
    
    - b/break 10 if i == 5
  
  - 运行GDB程序  
    
    - start（程序停在第一行）  
    - run（遇到断点才停）  
  
  - 继续运行，到下一个断点停  
    
    - c/continue  
  
  - 向下执行一行代码（不会进入函数体）  
    
    - n/next  
  
  - 变量操作
    
    - p/print 变量名（打印变量值）
    - ptype 变量名（打印变量类型）  
  
  - 向下单步调试（遇到函数进入函数体）
    
    - s/step  
    - finish（跳出函数体）  
  
  - 自动变量操作
    
    - display 变量名（设置为自动变量后运行时会自动打印自身的值） 
    - i/info display
    - undisplay 编号 （删除自动变量）
  
  - 其它操作
    
    - set var 变量名=变量值 （循环中用的较多）
    
    - until （跳出循环）——跳出循环条件的前提是循环里没有断点或断点不可用且当前代码已执行完
      
      ## 解决虚拟机连不上网的问题
      
      [ubuntu-22.04无法连接网络-sudo service network-manager stop说不行\_多云的夏天的博客-CSDN博客](https://blog.csdn.net/aggie4628/article/details/125765055)

- nmcli network on

## 文件I/O

![Pasted image 20230402113219](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230402113219_20-54-46.png)

- 缓冲区可以降低我们读写磁盘的次数，标准大小为8K
- 进行网络通信时为了保证效率需要使用Linux系统的I/O函数；在读写磁盘时需要使用标准C库的I/O函数

区别：标准C库的I/O函数是可以跨平台的。会针对不同的API有不同的接口

- 标准库I/O和Linux I/O的关系：
  
  - 标准库I/O时需要调用Linux I/O
    ![Pasted image 20230403110043](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230403110043_20-54-46.png)

- 虚拟地址空间——解释一些编程中的问题
    ![Pasted image 20230403143407](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230403143407_20-54-46.png)
  
  - 受保护的地址——null、nullptr

- 文件描述符
    ![Pasted image 20230403143845](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230403143845_20-54-46.png)
  
  - 文件描述符表默认的大小为1024
  
  - 前三个默认被占用，为标准输入、标准输出和标准错误；这三个对应的文件指向当前的同一个终端
  
  - 不同的文件描述符可以对应同一个文件；
    
    ### Linux系统I/O函数

- int open(const char \*pathname, int flags);  
  
  - **作用**：打开文件，返回文件描述符

- int open(const char \*pathname, int flags, mode_t mode);  
  
  - **作用**：创建一个新文件；通过可变参数实现，不是函数重载
  - 参数：
    - **pathname**：要创建的文件的路径
    - **flags**：对文件的操作权限和其他的设置
      - 必选项：O_RDONLY,  O_WRONLY, O_RDWR  这三个之间是互斥的
      - 可选项：O_CREAT 文件不存在，创建新文件；
      - 为什么O_RDWR和O_CREAT要用按位或？
        - flags参数是一个int类型的数据，占4个字节，32位。
        - flags 32个位，每一位就是一个标志位。
    - **mode**：八进制的数，表示创建出的新的文件的操作权限，比如：0775
      - 最终的权限是：mode & ~umask   //按位与：0和任何数都为0
      - 如：0777   ->   111111111  
      - // 1~3位表示当前用户的权限 4~6位表示当前用户所在组对这个文件的权限 7~9位表示其他组对这个文件的权限
      - 0777  &   0775   ->   111111111 & 111111101   得到111111101
      - umask的作用就是抹去某些权限，让创建的文件的权限更为合理，umask由系统设置

- int close(int fd);  
  
  - **作用**：关闭fd指向的文件

- void perror(const char \*s); 
  
  - **作用**：打印errno对应的错误描述
  - 需要include <stdio.h>
  - s参数：用户设置的描述，比如hello，最终输出的内容是  hello:xxx  (实际的错误描述)

- ssize_t read(int fd, void \*buf, size_t count);  
  
  - **作用**：从文件读取到内存中
  - 需要#include <unistd.h>
  - 参数：
    - fd：文件描述符，open得到的，通过这个文件描述符操作某个文件
    - buf：需要读取数据存放的地方，数组的地址（传出参数）
    - count：指定的数组的大小
  - 返回值：
    - 成功：
      - > 0 ：返回实际的读取到的字节数
      - = 0 ：文件已经读取完了
    - 失败：-1 ，并且设置errno

- ssize_t write(int fd, const void \*buf, size_t count);  
  
  - **作用**：从内存写入文件
  - 需要#include <unistd.h>
  - 参数：
    - fd：文件描述符，open得到的，通过这个文件描述符操作某个文件
    - buf：要往磁盘写入的数据，数据
    - count：要写的数据的实际的大小
  - 返回值
    - 成功：实际写入的字节数
    - 失败：返回-1，并设置errno

- off_t lseek(int fd, off_t offset, int whence); 
  
  - 参数
    - fd：文件描述符，通过open得到的，通过这个fd操作某个文件
    - offset：偏移量
    - whence:标记
      - SEEK_SET 设置文件指针的偏移量
      - SEEK_CUR 设置偏移量：当前位置 + 第二个参数offset的值
      - SEEK_END 设置偏移量：文件大小 + 第二个参数offset的值
  - 返回值：返回文件指针的位置
  - **作用**：
    - 移动文件指针到文件头  
      - lseek(fd, 0, SEEK_SET);
    - 获取当前文件指针的位置
      - lseek(fd, 0, SEEK_CUR);
    - 获取文件长度
      - lseek(fd, 0, SEEK_END);
    - 拓展文件的长度
      - lseek(fd, 100, SEEK_END) //将当前文件10b到110b, 增加了100个字节
      - 注意：需要写一次数据

- int stat(const char \*pathname, struct stat \*statbuf);  
  
  - **作用**：获取一个文件相关的一些信息
  
  - 参数:
    
    - pathname：操作的文件的路径
    - statbuf：结构体变量，传出参数，用于保存获取到的文件的信息
  
  - 返回值：
    
    - 成功：返回0
    
    - 失败：返回-1 设置errno
      ![Pasted image 20230403164841|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230403164841_11-41-12.png)
      
      ![Pasted image 20230403165053|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230403165053_11-41-24.png)

- int lstat(const char \*pathname, struct stat \*statbu  
  
  - **作用**：获取软链接文件的信息
  - 参数:
    - pathname：操作的文件的路径
    - statbuf：结构体变量，传出参数，用于保存获取到的文件的信息
  - 返回值：
    - 成功：返回0
    - 失败：返回-1 设置errno

- 模拟实现 ls -l 的功能
  
  ```cpp
  #include <stdio.h>
  #include <sys/types.h>
  #include <sys/stat.h>
  #include <unistd.h>
  #include <pwd.h>
  #include <grp.h>
  #include <time.h>
  #include <string.h>
  ```
  
    // 模拟实现 ls -l 指令
    // 输出格式：
    // -rw-rw-r-- 1 nowcoder nowcoder 12 12月  3 15:48 a.txt
  
    int main(int argc, char * argv[]) {
        // 判断输入的参数是否正确
        if(argc < 2) {
            printf("%s filename\n", argv[0]);
            return -1;
        }
        // 通过stat函数获取用户传入的文件的信息
        struct stat st;
        int ret = stat(argv[1], &st);
        if(ret == -1) {
            perror("stat");
            return -1;
        }
        // 获取文件类型和文件权限
        char perms[11] = {0};   // 用于保存文件类型和文件权限的字符串
        
        switch(st.st_mode & S_IFMT) {//获取文件类型
            case S_IFLNK:
                perms[0] = 'l';
                break;
            case S_IFDIR:
                perms[0] = 'd';
                break;
            case S_IFREG:
                perms[0] = '-';
                break;
            case S_IFBLK:
                perms[0] = 'b';
                break;
            case S_IFCHR:
                perms[0] = 'c';
                break;
            case S_IFSOCK:
                perms[0] = 's';
                break;
            case S_IFIFO:
                perms[0] = 'p';
                break;
            default:
                perms[0] = '?';
                break;
        }
  
        // 判断文件的访问权限
        // 文件所有者
        perms[1] = (st.st_mode & S_IRUSR) ? 'r' : '-';
        perms[2] = (st.st_mode & S_IWUSR) ? 'w' : '-';
        perms[3] = (st.st_mode & S_IXUSR) ? 'x' : '-';
  
        // 文件所在组
        perms[4] = (st.st_mode & S_IRGRP) ? 'r' : '-';
        perms[5] = (st.st_mode & S_IWGRP) ? 'w' : '-';
        perms[6] = (st.st_mode & S_IXGRP) ? 'x' : '-';
  
        // 其他人
        perms[7] = (st.st_mode & S_IROTH) ? 'r' : '-';
        perms[8] = (st.st_mode & S_IWOTH) ? 'w' : '-';
        perms[9] = (st.st_mode & S_IXOTH) ? 'x' : '-';
  
        // 硬连接数
        int linkNum = st.st_nlink;
  
        // 文件所有者
        char * fileUser = getpwuid(st.st_uid)->pw_name;
  
        // 文件所在组
        char * fileGrp = getgrgid(st.st_gid)->gr_name;
  
        // 文件大小
        long int fileSize = st.st_size;
  
        // 获取修改的时间
        char * time = ctime(&st.st_mtime);
  
        //获取时间
  
        char mtime[512] = {0};
        strncpy(mtime, time, strlen(time) - 1);
  
        //字符串拼接
  
        char buf[1024];
        sprintf(buf, "%s %d %s %s %ld %s %s", perms, linkNum, fileUser, fileGrp, fileSize, mtime, argv[1]);
  
        printf("%s\n", buf);
  
        return 0;
  
    }
  
  ```
  
  ```

### 文件属性操作函数

- int access(const char * pathname, int mode);  
  
  - 头文件：#include <unistd.h>
  - **作用：判断某个文件是否有某个权限，或者判断文件是否存在
  - 参数：
    - pathname: 判断的文件路径
    - mode:
      - R_OK: 判断是否有读权限
      - W_OK: 判断是否有写权限
      - X_OK: 判断是否有执行权限
      - F_OK: 判断文件是否存在
  - 返回值：成功返回0， 失败返回-1

- int chmod(const char * filename, int mode);  
  
  - 头文件：#include <sys/stat.h>
  - **作用**：修改文件的权限
  - 参数：
    - filename: 需要修改的文件的路径
    - mode:需要修改的权限值，八进制的数
  - 返回值：成功返回0，失败返回-1

- int chown(const char * path, uid_t owner, gid_t group);  
  
  - **作用**：修改文件所在组或所有者
  - 头文件：#include <unistd.h>
  - 参数：
    - path ：需要修改的文件的路径
    - owner：所有者的uid——vim /etc/passwd查看owner
    - group：所在组组名——vim /etc/group 查看组

- int truncate(const char * path, off_t length);
  
  - 头文件：#include <unistd.h> 、#include <sys/types.h>
  
  - **作用**：缩减或者扩展文件的尺寸至指定的大小
  
  - 参数：
    
    - path: 需要修改的文件的路径
    - length: 需要最终文件变成的大小
  
  - 返回值： 成功返回0， 失败返回-1
    
    ### 目录操作函数

- int rename(const char * oldpath, const char * newpath);  
  
  - **作用**：将旧的文件民 oldpath 改成新的 newpath
  - 头文件：#include <stdio.h>

- int chdir(const char * path);  
  
  - **作用**：修改进程的工作目录
    - 比如在/home/nowcoder 启动了一个可执行程序a.out, 进程的工作目录 /home/nowcoder
  - 头文件：#include <unistd.h>
  - 参数：
    - path : 需要修改的工作目录

- char * getcwd(char * buf, size_t size);  
  
  - **作用**：获取当前工作目录
  - 头文件：#include <unistd.h>
  - 参数：
    - buf : 存储的路径，指向的是一个数组（传出参数）
    - size: 数组的大小
  - 返回值： 返回的指向的一块内存，这个数据就是第一个参数

- int mkdir(const char * pathname, mode_t mode);  
  
  - **作用**：创建一个目录
  - 头文件：#include <sys/stat.h>、#include <sys/types.h>
  - 参数：
    - pathname: 创建的目录的路径
    - mode: 权限，八进制的数
  - 返回值： 成功返回0， 失败返回-1

- int rmdir(const char * pathname);
  
  - 作用：空的目录
    
    ### 目录遍历函数

- DIR * opendir(const char * name);  
  
  - **作用**：打开指定目录
  - 头文件：#include <sys/types.h> 、#include <dirent.h>
  - 参数：
    - name: 需要打开的目录的名称
  - 返回值：
    - DIR * 类型，理解为目录流
    - 错误返回NULL

- struct dirent * readdir(DIR * dirp);  
  
  - **作用**：读取目录中的数据
  - 头文件：#include <dirent.h>
  - 参数：dirp是opendir返回的结果
  - 返回值：
    - struct dirent，代表读取到的文件的信息
    - 读取到了末尾或者失败了，返回NULL

- dirent结构体和d_type
     ![Pasted image 20230404094851|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230404094851_11-41-49.png)

- int closedir(DIR * dirp);
  
  - **作用**：关闭目录
  - 头文件：#include <sys/types.h> 、#include <dirent.h>

- dup、dup2函数
  
  - int dup(int oldfd)    
    - **作用**：复制一个新的文件描述符
      - fd=3，int fd1 = dup(fd),
      - fd指向的是a.txt, fd1也是指向a.txt
    - 头文件：#include <unistd.h>
    - 从空闲的文件描述符表中找一个最小的，作为新的拷贝的文件描述符
  - int dup2(int oldfd, int newfd);   
    - **作用**：重定向文件描述符
      - oldfd 指向 a.txt，newfd 指向 b.txt
      - 调用函数成功后：newfd 和 b.txt取消关联，newfd 指向了 a.txt
      - oldfd 必须是一个有效的文件描述符
      - oldfd和newfd值相同，相当于什么都没有做
    - 头文件：#include <unistd.h>

- **fcntl函数**
  
  - int fcntl(int fd, int cmd, ... / * arg * / ); 
    - **作用**：复制文件描述符、设置/获取文件的状态标志
    - 头文件：#include <unistd.h>、#include <fcntl.h>
    - 参数：
      - fd : 表示需要操作的文件描述符
      - cmd: 表示对文件描述符进行如何操作
        - F_DUPFD : 复制文件描述符,复制的是第一个参数fd，得到一个新的文件描述符（返回值）     
          - int ret = fcntl(fd, F_DUPFD);  ret为新复制的文件描述符
        - F_GETFL : 获取指定的文件描述符文件状态flag  
          - 获取的flag和我们通过open函数传递的flag是一个东西。
        - F_SETFL : 设置文件描述符文件状态flag
          - 必选项：O_RDONLY, O_WRONLY, O_RDWR 不可以被修改
          - 可选性：O_APPEND, O_NONBLOCK
            - O_APPEND 表示追加数据，只有有写数据权限时才能写入追加数据
            - O_NONBLOK 设置成非阻塞——阻塞和非阻塞：描述的是函数调用的行为。

## 进程

### 程序和进程

**程序是包含一系列信息的文件**，这些信息描述了如何在运行时创建一个进程：  

- 二进制格式标识：每个程序文件都包含用于描述可执行文件格式的元信息。内核利用此信息来解释文件中的其他信息。（ELF可执行连接格式）  
- 机器语言指令：对程序算法进行编码。  
- 程序入口地址：标识程序开始执行时的起始指令位置。  
- 数据：程序文件包含的变量初始值和程序使用的字面量值（比如字符串）。  
- 符号表及重定位表：描述程序中函数和变量的位置及名称。这些表格有多重用途，其中包括调试和运行时的符号解析（动态链接）。  
- 共享库和动态链接信息：程序文件所包含的一些字段，列出了程序运行时需要使用的共享库，以及加载共享库的动态连接器的路径名。  
- 其他信息：程序文件还包含许多其他信息，用以描述如何创建进程。

**进程的概念**：

- 进程是正在运行的程序的实例。是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。它是操作系统动态执行的基本单元，在传统的操作系统中，进程既是基本的分配单元，也是基本的执行单元。  
- 可以用一个程序来创建多个进程，进程是由内核定义的抽象实体，并为该实体分配用以执行程序的各项系统资源。从内核的角度看，进程由用户内存空间和一系列内核数据结构组成，其中用户内存空间包含了程序代码及代码所使用的变量，而内核数据结构则用于维护进程状态信息。记录在内核数据结构中的信息包括许多与进程相关的标识号（IDs）、虚拟内存表、打开文件的描述符表、信号传递及处理的有关信息、进程资源使用及限制、当前工作目录和大量的其他信息。

### 单道、多道程序设计

- 单道程序，即在计算机内存中只允许一个的程序运行。  
- 多道程序设计技术是在计算机内存中同时存放几道相互独立的程序，使它们在管理程序控制下，相互穿插运行，两个或两个以上程序在计算机系统中同处于开始到结束之间的状态, 这些程序共享计算机系统资源。引入多道程序设计技术的根本目的是为了提高 CPU 的利用率。  
- 对于一个单 CPU 系统来说，程序同时处于运行状态只是一种宏观上的概念，他们虽然都已经开始运行，但就微观而言，任意时刻，CPU 上运行的程序只有一个。  
- 在多道程序设计模型中，多个进程轮流使用 CPU。而当下常见 CPU 为纳秒级，1秒  可以执行大约 10 亿条指令。由于人眼的反应速度是毫秒级，所以看似同时在运行

### 时间片

- 时间片（timeslice）又称为“量子（quantum）”或“处理器片（processor slice）”是操作系统分配给每个正在运行的进程微观上的一段 CPU 时间。事实上，虽然一台计算机通常可能有多个 CPU，但是同一个 CPU 永远不可能真正地同时运行多个任务。在只考虑一个 CPU 的情况下，这些进程“看起来像”同时运行的，实则是轮番穿插地运行，由于时间片通常很短（在 Linux 上为 5ms－800ms），用户不会感觉到。  
- 时间片由操作系统内核的调度程序分配给每个进程。首先，内核会给每个进程分配相等的初始时间片，然后每个进程轮番地执行相应的时间，当所有进程都处于时间片耗尽的状态时，内核会重新为每个进程计算并分配时间片，如此往复。

### 并发和并行

- 并行(parallel)：指在同一时刻，有多条指令在多个处理器上同时执行。  
- 并发(concurrency)：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。
- ![Pasted image 20230404103733|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230404103733_11-42-11.png)

### 进程控制块（PCB）

- 为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。内核为每个进程分配一个 PCB(Processing Control Block)进程控制块，维护进程相关的信息，Linux 内核的进程控制块是 task_struct 结构体。  
- 在 /usr/src/linux-headers-xxx/include/linux/sched.h 文件中可以查看 struct task_struct 结构体定义。其内部成员有很多，我们只需要掌握以下部分即可：  
  - 进程id：系统中每个进程有唯一的 id，用 pid_t 类型表示，其实就是一个非负整数  
  - 进程的状态：有就绪、运行、挂起、停止等状态  
  - 进程切换时需要保存和恢复的一些CPU寄存器  
  - 描述虚拟地址空间的信息  
  - 描述控制终端的信息
  - 当前工作目录（Current Working Directory）  
  - umask 掩码  
  - 文件描述符表，包含很多指向 file 结构体的指针  
  - 和信号相关的信息  
  - 用户 id 和组 id  
  - 会话（Session）和进程组  
  - 进程可以使用的资源上限（Resource Limit）

### 进程的状态

进程状态反映进程执行过程的变化。这些状态随着进程的执行和外界条件的变化而转换。在三态模型中，进程状态分为三个基本状态，即就绪态，运行态，阻塞态。在五态模型中，进程分为新建态、就绪态，运行态，阻塞态，终止态。

- **运行态**：进程占有处理器正在运行  

- **就绪态**：进程具备运行条件，等待系统分配处理器以便运行。当进程已分配到除CPU以外的所有必要资源后，只要再获得CPU，便可立即执行。在一个系统中处于就绪状态的进程可能有多个，通常将它们排成一个队列，称为就绪队列  

- **阻塞态**：又称为等待(wait)态或睡眠(sleep)态，指进程不具备运行条件，正在等待某个事件的完成
    ![Pasted image 20230404104304](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230404104304_11-42-30.png)

- **新建态**：进程刚被创建时的状态，尚未进入就绪队列  

- **终止态**：进程完成任务到达正常结束点，或出现无法克服的错误而异常终止，或被操作系统及有终止权的进程所终止时所处的状态。进入终止态的进程以后不再执行，但依然保留在操作系统中等待善后。一旦其他进程完成了对终止态进程的信息抽取之后，操作系统将删除该进程。
  
    ![Pasted image 20230404104355|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230404104355_11-42-40.png)

### 进程相关命令

- 查看进程  
  - **ps aux / ajx**  
  - a：显示终端上的所有进程，包括其他用户的进程  
  - u：显示进程的详细信息  
  - x：显示没有控制终端的进程  
  - j：列出与作业控制相关的信息
  - **ps aux显示信息**
      ![Pasted image 20230404105128|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230404105128_11-43-00.png)
    - PID 进程ID
    - TTY表示当前所属终端
    - STAT参数意义：  
      - D 不可中断 Uninterruptible（usually IO）  
      - R 正在运行，或在队列中的进程  
      - S(大写) 处于休眠状态  
      - T 停止或被追踪  
      - Z 僵尸进程  
      - W 进入内存交换（从内核2.6开始无效）  
      - X 死掉的进程  
      - < 高优先级  
      - N 低优先级  
      - s 包含子进程 
      - + 位于前台的进程组
  - **ps ajx 显示信息**
      ![Pasted image 20230404105511|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230404105511_11-43-20.png)
    - PPID 当前进程父进程的ID
    - PGID 当前进程所属进程组
- 实时显示进程动态  
  - **top**  
  - 可以在使用 top 命令时加上 -d 来指定显示信息更新的时间间隔，在 top 命令执行后，可以按以下按键对显示的结果进行排序：  
    - M 根据内存使用量排序  
    - P 根据 CPU 占有率排序  
    - T 根据进程运行时间长短排序  
    - U 根据用户名来筛选进程  
    - K 输入指定的 PID 杀死进程
- 杀死进程  
  - kill \[ -signal]  pid
  - kill –l 列出所有信号
  - kill –SIGKILL 进程ID
  - kill -9 进程ID  //彻底杀死进程
  - killall name 根据进程名杀死进程

### 进程号和相关函数

- **每个进程都由进程号来标识**，其类型为 pid_t（整型），进程号的范围：0～32767。进程号总是唯一的，但可以重用。当一个进程终止后，其进程号就可以再次使用。  
- 任何进程（除 init 进程）都是由另一个进程创建，该进程称为被创建进程的父进程，对应的进程号称为父进程号（PPID）。  
- **进程组是一个或多个进程的集合**。他们之间相互关联，进程组可以接收同一终端的各种信号，关联的进程有一个进程组号（PGID）。默认情况下，当前的进程号会当做当前的进程组号。  
- 进程号和进程组相关函数：  
  - pid_t getpid(void);   //获取当前进程号
  - pid_t getppid(void);  //获取父进程号
  - pid_t getpgid(pid_t pid); //获取进程组号

### 创建进程

系统允许一个进程创建新进程，新进程即为子进程，子进程还可以创建新的子进程，形成进程树结构模型。  

- pid_t fork(void);  
  
  - **作用**：创建子进程
  - 头文件：#include <sys/types.h>、#include <unistd.h>
  - 返回值：  
    - fork()的返回值会返回两次。一次是在父进程中，一次是在子进程中
    - 成功：子进程中返回 0，父进程中返回子进程 ID  
    - 失败：返回 -1 
      - 在父进程中返回-1，表示创建子进程失败，并且设置errno
      - 失败的两个主要原因： 
        - 当前系统的进程数已经达到了系统规定的上限，这时 errno 的值被设置为 EAGAIN 
        - 系统内存不足，这时 errno 的值被设置为 ENOMEM
    - 如何区分父进程和子进程：通过fork的返回值。
  - **父子进程之间的关系**：
    - 父子进程时交替运行的
    - 区别：
      - .fork()函数的返回值不同
        - 父进程中: >0 返回的子进程的ID
        - 子进程中: =0
      - pcb中的一些数据
        - 当前的进程的id pid
        - 当前的进程的父进程的id ppid
        - 信号集
    - 共同点：
      - 某些状态下：子进程刚被创建出来，还没有执行任何的写数据的操作
        - 用户区的数据
        - 文件描述符表
  - **父子进程对变量是不是共享的？**
    - 读时共享（子进程被创建，两个进程没有做任何的写的操作）
    - 写时拷贝
    - 刚开始的时候，是一样的，共享的。如果修改了数据就不共享了
    - 实际上，更准确来说，Linux 的 fork() 使用是通过写时拷贝 (copy- on-write) 实现。写时拷贝是一种可以推迟甚至避免拷贝数据的技术。内核此时并不复制整个进程的地址空间，而是让父子进程共享同一个地址空间。只用在需要写入的时候才会复制地址空间，从而使各个进行拥有各自的地址空间。也就是说，资源的复制是在需要写入的时候才会进行，在此之前，只有以只读方式共享。
    - 注意：fork之后父子进程共享文件，fork产生的子进程与父进程相同的文件文件描述符指向相同的文件表，引用计数增加，共享文件偏移指针。

- **GDB 多进程调试**
  
  - 使用 GDB 调试的时候，GDB 默认只能跟踪一个进程，可以在 fork 函数调用之前，通过指令设置 GDB 调试工具跟踪父进程或者是跟踪子进程，**默认跟踪父进程**。  
  - 设置调试父进程或者子进程：set follow-fork-mode \[parent（默认）| child] 
  - 设置调试模式：set detach-on-fork \[on | off]  
    - 默认为 on，表示调试当前进程的时候，其它的进程继续运行，如果为 off，调试当前进程的时候，其它进程被 GDB 挂起。  
  - 查看调试的进程：info inferiors  
  - 切换当前调试的进程：inferior id  
  - 使进程脱离 GDB 调试：detach inferiors id

### exec函数族——进程程序替换

- exec 函数族的作用是**根据指定的文件名找到可执行文件**，并用它来取代调用进程的内容，换句话说，就是在调用进程内部执行一个可执行文件

- exec 函数族的函数执行成功后不会返回，因为正在执行的进程本身的pcb、mm_struct、页表等信息不会发生改变，仅仅用新的程序代码和数据替换了原来进程的代码和数据；只有调用失败了，它们才会返回 -1，从原程序的调用点接着往下执行

- 作用：
  
  - 当进程认为自己不能再为系统和用户做出任何贡献时，就可以调用任何exec 函数族让自己重生。
  - 如果一个进程想执行另一个程序，那么它就可以调用fork函数新建一个进程，然后调用任何一个exec函数使子进程重生。

- 标准C库函数：
  
  - int execl(const char \*path, const char \*arg, .../* (char \*) NULL \*/);  
    - **作用**：进程替换
    - 头文件：#include <unistd.h>
    - 参数：
      - path:需要指定的执行的文件的路径或者名称
        - a.out /home/nowcoder/a.out 推荐使用绝对路径
        - ./a.out hello world
      - arg:是执行可执行文件所需要的参数列表
        - 第一个参数一般没有什么作用，为了方便，一般写的是执行的程序的名称
        - 从第二个参数开始往后，就是程序执行所需要的的参数列表。
        - 参数最后需要以NULL结束（哨兵）
      - 返回值：
        - 只有当调用失败，才会有返回值，返回-1，并且设置errno
        - 如果调用成功，没有返回值。
  - int execlp(const char \*file, const char \*arg, ... /* (char \*) NULL \*/);  
    - **作用**：会到环境变量中查找指定的可执行文件，如果找到了就执行，找不到就执行不成功。
    - 参数：
      - file:需要执行的可执行文件的文件名     a.out  ps
      - arg:是执行可执行文件所需要的参数列表
        - 第一个参数一般没有什么作用，为了方便，一般写的是执行的程序的名
        - 从第二个参数开始往后，就是程序执行所需要的的参数列表。
        - 参数最后需要以NULL结束（哨兵）
    - 返回值：
      - 只有当调用失败，才会有返回值，返回-1，并且设置errno
      - 如果调用成功，没有返回值。
  - int execle(const char \*path, const char \*arg, .../\*, (char \*) NULL\*/, char \*  const envp[] );  
  - int execv(const char \*path, char \*const argv[]);  
  - int execvp(const char \*file, char \*const argv[]);  
  - int execvpe(const char \*file, char \*const argv[], char \*const envp[]); 

- Linux系统下：
  
  - int execve(const char \*filename, char \*const argv[], char \*const envp[]);  

- exec函数族分类
  
  - l（list) 参数地址以列表形式给出，以空指针结尾  
  
  - v（vector) 参数传递为构造指针数组方式
  
  - p（path)   可传递新进程环境变量
  
  - e（environment) 可传递新进程环境变量
    ![Pasted image 20230404201629](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230404201629_20-54-46.png)
    
    ### 进程退出

- 进程退出函数：
  
  - void exit(int status)
    
    - 头文件：#include <stdlib.h>
  
  - void \_exit(int status)
    
    - 头文件：#include <unistd.h>
  
  - staus参数为进程退出时的状态信息，父进程回收子进程资源时可以获得
  
  - ![Pasted image 20230404193313](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230404193313_20-54-46.png)
  
  - 二者区别:printf()中有换行时内部会自动刷新缓冲区显示数据；当没有换行符时数据只是写到缓冲区，调用exit时会刷新缓冲区从而输出，\_exit则不会，所以不会输出world。代码如下：
    
    ```cpp
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    int main() {
        printf("hello\n");
        printf("world");
        // exit(0);
        _exit(0);
        return 0;
    }
    ```

#### 孤儿进程

- 父进程运行结束，但子进程还在运行（未运行结束），这样的子进程就称为孤儿进程（Orphan Process）。  
- 每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init ，而 init进程会循环地 wait() 它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候，init 进程就会处理它的一切善后工作。因此孤儿进程并不会有什么危害
- 由于子进程是父进程的处继承了整个进程的地址空间，包括：**「进程上下文、进程堆栈、内存信息、打开的文件描述符、信号控制设置、进程优先级、进程组号、当前工作目录、根目录、资源限制、控制终端」** 等；所以孤儿子进程可以将数据输出到当前终端

#### 僵尸进程

- 每个进程结束之后, 都会释放自己地址空间中的用户区数据(栈、堆)，内核区的 PCB 没有办法自己释放掉，需要父进程去释放。  
- 进程终止时，父进程尚未回收，子进程残留资源（PCB）存放于内核中，变成僵尸  
  （Zombie）进程
- 僵尸进程不能被 kill -9 杀死
- 如果父进程不调用 wait()或 waitpid() 的话，那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，此即为僵尸进程的危害，应当避免

#### 进程回收

- 在每个进程退出的时候，内核释放该进程所有的资源、包括打开的文件、占用的内存等。但是仍然为其保留一定的信息，这些信息主要主要指进程控制块PCB的信息 （包括进程号、退出状态、运行时间等）。
- **父进程可以通过调用wait或waitpid得到它的退出状态同时彻底清除掉这个进程**。  
- wait() 和 waitpid() 函数的功能一样，区别在于，wait() 函数会阻塞，waitpid() 可以设置不阻塞，waitpid() 还可以指定等待哪个子进程结束。
- 注意：一次wait或waitpid调用只能清理一个子进程，清理多个子进程应使用循环

#### wait和waitpid函数

- pid_t wait(int \*wstatus);
  
  - **功能：等待任意一个子进程结束，如果任意一个子进程结束了，此函数会回收子进程的资源**。
  - 头文件：#include <sys/types.h>、#include <sys/wait.h>
  - 参数：int \*wstatus  进程退出时的状态信息，传入的是一个int类型的地址（可以利用引用传递），用于传出状态信息参数。
  - 返回值：
    - 成功：返回被回收的子进程的id
    - 失败：-1 (所有的子进程都结束，调用函数失败)
  - 调用wait函数的进程会被挂起（阻塞），直到它的一个子进程退出或者收到一个不能被忽略的信号时才被唤醒（相当于继续往下执行）
  - 如果没有子进程了，函数立刻返回，返回-1；如果子进程都已经结束了，也会立即返回，返回-1.

- pid_t waitpid(pid_t pid, int \*wstatus, int options);
  
  - **功能：回收指定进程号的子进程，可以设置是否阻塞**。
  - 头文件：#include <sys/types.h>、#include <sys/wait.h>
  - 参数：
    - pid：
      - pid > 0 : 某个子进程的pid
      - pid = 0 : 回收当前进程组的任意子进程    
      - pid = -1 : 回收所有的子进程，相当于 wait()  （最常用）
      - pid < -1 : 某个进程组的组id的绝对值，回收指定进程组中的子进程
    - options：设置阻塞或者非阻塞
      - 0 : 阻塞
      - WNOHANG : 非阻塞，如果pid指定的子进程没有结束，则waitpid()函数立即返回0，而不是阻塞在这个函数上等待；如果结束了，则返回该子进程的进程号。——父进程可以不挂起，继续执行
      - WUNTRACED：如果子进程进入暂停状态，则马上返回。
  - 返回值：
    - \> 0 : 返回子进程的id
    - = 0 : options=WNOHANG, 表示还有子进程活着
    - = -1 ：错误，或者没有子进程了
      - 失败原因主要有：没有子进程（errno设置为ECHILD），调用被某个信号中断（errno设置为EINTR）或选项参数无效（errno设置为EINVAL）

#### 退出信息相关宏函数

- WIFEXITED(status) 非0，进程正常退出  
- WEXITSTATUS(status) 如果上宏为真，获取进程退出的状态（exit的参数）  
- WIFSIGNALED(status) 非0，进程异常终止  
- WTERMSIG(status) 如果上宏为真，获取使进程终止的信号编号
- WIFSTOPPED(status) 非0，进程处于暂停状态  
- WSTOPSIG(status) 如果上宏为真，获取使进程暂停的信号的编号  
- WIFCONTINUED(status) 非0，进程暂停后已经继续运行

### 守护进程

#### 终端

- 在 UNIX 系统中，用户通过终端登录系统后得到一个 shell 进程，这个终端成为 shell 进程的控制终端（Controlling Terminal），进程中，控制终端是保存在 PCB 中的信息，而 fork() 会复制 PCB 中的信息，因此由 shell 进程启动的其它进程的控制终端也是这个终端
- 默认情况下（没有重定向），每个进程的标准输入、标准输出和标准错误输出都指向控制终端，进程从标准输入读也就是读用户的键盘输入，进程往标准输出或标准错误输出写也就是输出到显示器上。  
- 在控制终端输入一些特殊的控制键可以给前台进程发信号，例如 Ctrl + C 会产生 SIGINT 信号，Ctrl + \ 会产生 SIGQUIT 信号

#### 进程组

- 进程组和会话在进程之间形成了一种两级层次关系：**进程组是一组相关进程的集合，会话是一组相关进程组的集合**。进程组和会话是为支持 shell 作业控制而定义的抽象概念，用户通过 shell 能够交互式地在前台或后台运行命令。  
- 进行组由一个或多个共享同一进程组标识符（PGID）的进程组成。一个进程组拥有一个进程组首进程，该进程是创建该组的进程，其进程 ID 为该进程组的 ID，新进程会继承其父进程所属的进程组 ID。  
- 进程组拥有一个生命周期，其开始时间为首进程创建组的时刻，结束时间为最后一个成员进程退出组的时刻。一个进程可能会因为终止而退出进程组，也可能会因为加入了另外一个进程组而退出进程组。进程组首进程无需是最后一个离开进程组的成员

#### 会话

- 会话是一组进程组的集合。**会话首进程是创建该新会话的进程，其进程 ID 会成为会话 ID**。新进程会继承其父进程的会话 ID。  
- 一个会话中的所有进程共享单个控制终端。控制终端会在会话首进程首次打开一个终端设备时被建立。一个终端最多可能会成为一个会话的控制终端。 
- 在任一时刻，会话中的其中一个进程组会成为终端的前台进程组，其他进程组会成为后台进程组。只有前台进程组中的进程才能从控制终端中读取输入。当用户在控制终端中输入终端字符生成信号后，该信号会被发送到前台进程组中的所有成员。  
- 当控制终端的连接建立起来之后，会话首进程会成为该终端的控制进程

#### 进程组、会话、控制终端之间的关系

- find / 2 > /dev/null | wc -l &  
- sort < longlist | uniq -c

![Pasted image 20230408152150|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230408152150_11-43-49.png)

#### 进程组、会话操作函数

- pid_t getpgrp(void);  //获取当前进程的进程组ID
- pid_t getpgid(pid_t pid);  //获取指定进程的进程组ID
- int setpgid(pid_t pid, pid_t pgid);  //设置进程的进程组ID
- pid_t getsid(pid_t pid);  //获取指定进程的会话ID
- pid_t setsid(void);  //设置会话ID

#### 守护进程

- 守护进程（Daemon Process），也就是通常说的 Daemon 进程（精灵进程），是Linux 中的**后台服务进程**。它是一个生存期较长的进程，通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。一般采用以 d 结尾的名字。  
- 守护进程具备下列特征：
  - 生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭。  
  - 它在后台运行并且**不拥有控制终端**。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如 SIGINT、SIGQUIT）。 
- Linux 的大多数服务器就是用守护进程实现的。比如，Internet 服务器 inetd，Web 服务器 httpd 等

#### 守护进程的创建步骤

前两步和最后一步为必要步骤：

- 执行一个 fork()，之后父进程退出，子进程继续执行。 ——使用子进程建立会话ID可以避免父进程组ID和会话ID产生冲突 
- 子进程调用 setsid() 开启一个新会话。（没有建立连接，没有控制终端）
- 清除进程的 umask 以确保当守护进程创建文件和目录时拥有所需的权限。  
- 修改进程的当前工作目录，通常会改为根目录（/）。  
- 关闭守护进程从其父进程继承而来的所有打开着的文件描述符。  
- 在关闭了文件描述符0、1、2之后，守护进程通常会重定向到/dev/null（往里面写数据都会丢弃）并使用dup2()使所有这些描述符指向这个设备。  
- 核心业务逻辑

## 进程间通信

- 进程是一个独立的资源分配单元，不同进程（这里所说的进程通常指的是用户进程）之间的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源。  
- 但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要**进程间通信**( IPC：Inter Processes Communication )。  
- 进程间通信的目的：  
  - 数据传输：一个进程需要将它的数据发送给另一个进程。  
  - 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要通知父进程）。  
  - 资源共享：多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同步机制。  
  - 进程控制：有些进程希望完全控制另一个进程的执行（如 Debug 进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变
- **进程间通信方式**：
    ![Pasted image 20230405170733](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230405170733_20-54-46.png)

### 匿名管道

- 管道也叫无名（匿名）管道，它是是 UNIX 系统 IPC（进程间通信）的最古老形式，所有的 UNIX 系统都支持这种通信机制。  

- 举例：统计一个目录中文件的数目命令：ls | wc –l，为了执行该命令，shell 创建了两个进程来分别执行 ls 和 wc; **| 为管道符**
     ![Pasted image 20230405170950|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230405170950_11-44-15.png)

- 管道的特点（有名和匿名）：
  
  - 管道其实是一个在内核内存中维护的**缓冲器**，这个缓冲器的存储能力是有限的，不同的操作系统大小不一定相同。  
  - 管道拥有文件的特质：读操作、写操作，匿名管道没有文件实体，有名管道有文件实体，但不存储数据。可以按照操作文件的方式对管道进行操作。  
  - 一个管道是一个**字节流**，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。  
  - 通过管道传递的数据是**顺序**的，从管道中读取出来的字节的顺序和它们被写入管道的顺序是完全一样的。
  - 在管道中的数据的传递方向是**单向**的，一端用于写入，一端用于读取，管道是半双工的。  
  - ![Pasted image 20230405171247|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230405171247_11-44-31.png)
  - 从管道读数据是**一次性**操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据，在管道中无法使用 lseek() 来随机的访问数据。  
  - 匿名管道只能在具有**公共祖先**的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用

- 为什么可以使用管道进行进程间通信?
  
  - ![Pasted image 20230405172443|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230405172443_11-44-50.png)
  - 原因：父子进程共享文件描述符

- 管道的数据结构
  
  - ![Pasted image 20230405172629](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230405172629_11-44-57.png)
  - **环形队列结构**——更充分地利用资源

- 匿名管道的使用
  
  - 创建匿名管道  
    - int pipe(int pipefd\[2]);  
      - 头文件：#include <unistd.h>  
      - 功能：创建一个匿名管道，用来进程间通信。
      - 参数：int pipefd\[2] 这个数组是一个传出参数。
        - pipefd\[0] 对应的是管道的读端
        - pipefd\[1] 对应的是管道的写端
      - 返回值： 成功 0 失败 -1
      - 管道默认是阻塞的：如果管道中没有数据，read阻塞，如果管道满了，write阻塞
      - 注意：匿名管道只能用于**具有关系的进程之间**的通信（父子进程，兄弟进程）
  - 查看管道缓冲大小命令ulimit –a  
  - 查看管道缓冲大小函数  
    - long fpathconf(int fd, int name);
    - 头文件：#include <unistd.h>  

- 管道的读写特点：
  
  - 使用管道时，需要注意以下几种特殊的情况（假设都是阻塞I/O操作）
    1. 所有的指向管道**写端**的文件描述符都关闭了（管道写端引用计数为0），有进程从管道的读端读数据，那么管道中剩余的数据被读取以后，再次read会返回0，就像读到文件末尾一样。
    2. 如果有指向管道**写端**的文件描述符没有关闭（管道的写端引用计数大于0），而持有管道写端的进程也没有往管道中写数据，这个时候有进程从管道中读取数据，那么管道中剩余的数据被读取后，再次read会阻塞，直到管道中有数据可以读了才读取数据并返回。
    3. 如果所有指向管道**读端**的文件描述符都关闭了（管道的读端引用计数为0），这个时候有进程向管道中写数据，那么该进程会收到一个信号SIGPIPE, 通常会导致进程异常终止。
    4. 如果有指向管道**读端**的文件描述符没有关闭（管道的读端引用计数大于0），而持有管道读端的进程也没有从管道中读数据，这时有进程向管道中写数据，那么在管道被写满的时候再次write会阻塞，直到管道中有空位置才能再次写入数据并返回。
  - **总结**：
    - 读管道：
      - 管道中有数据，read返回实际读到的字节数。
      - 管道中无数据：
        - 写端被全部关闭，read返回0（相当于读到文件的末尾）
        - 写端没有完全关闭，read阻塞等待
    - 写管道：
      - 管道读端全部被关闭，进程异常终止（进程收到SIGPIPE信号）
      - 管道读端没有全部关闭：
        - 管道已满，write阻塞
        - 管道没有满，write将数据写入，并返回实际写入的字节数

- 设置管道非阻塞
  
  - int flags = fcntl(fd\[0], F_GETFL);  // 获取原来的flag
  - flags |= O_NONBLOCK;            // 修改flag的值
  - fcntl(fd\[0], F_SETFL, flags);   // 设置新的flag

### 有名管道

- 匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了**有名管道（FIFO）**，也叫命名管道、FIFO文件。  

- 有名管道（FIFO）不同于匿名管道之处在于**它提供了一个路径名与之关联**，以 **FIFO的文件形式**存在于文件系统中，并且其打开方式与打开一个普通文件是一样的，这样即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过 FIFO 相互通信，因此，通过 FIFO 不相关的进程也能交换数据。  

- 一旦打开了 FIFO，就能在它上面使用与操作匿名管道和其他文件的系统调用一样的I/O系统调用了（如read()、write()和close()）。与管道一样，FIFO 也有一个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。FIFO 的名称也由此而来：先入先出

- 有名管道（FIFO)和匿名管道（pipe）有一些特点是相同的，不一样的地方在于：  
  
  1. FIFO 在文件系统中作为一个特殊文件存在，FIFO 中的内容却存放在内存中。  
  2. 当使用 FIFO 的进程退出后，FIFO 文件将继续保存在文件系统中以便以后使用。  
  3. FIFO 有名字，不相关的进程可以通过打开有名管道进行通信。

- 通过命令创建有名管道
  
  - **mkfifo 管道名字**  

- 通过函数创建有名管道  
  
  - int mkfifo(const char \*pathname, mode_t mode);  
    - 头文件：#include <sys/types.h> 、#include <sys/stat.h>  
    - 参数：
      - pathname: 管道名称的路径
      - mode: 文件的权限 和 open 的 mode 是一样的；是一个八进制的数
    - 返回值：成功返回0，失败返回-1，并设置错误号

- 一旦使用 mkfifo 创建了一个 FIFO，就可以使用 open 打开它，常见的文件I/O 函数都可用于 fifo。如：close、read、write、unlink 等。  

- FIFO 严格遵循先进先出（First in First out），对管道及 FIFO 的读总是从开始处返回数据，对它们的写则把数据添加到末尾。它们不支持诸如 lseek()等文件定位操作

- 有名管道的注意事项：
    1.一个为只读而打开一个管道的进程会阻塞，直到另外一个进程为只写打开管道
    2.一个为只写而打开一个管道的进程会阻塞，直到另外一个进程为只读打开管道

- 利用有名管道实现聊天功能（一来一回）
  
  - 思路（一来一回）
    ![Pasted image 20230406103603](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230406103603_11-45-16.png)
  
  - chatA.c
    
    ```cpp
    #include <stdio.h>
    #include <unistd.h>
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <string.h>
    
    int main() {
    
        // 1.判断有名管道文件是否存在
        int ret = access("fifo1", F_OK);
        if(ret == -1) {
            // 文件不存在
            printf("管道不存在，创建对应的有名管道\n");
            ret = mkfifo("fifo1", 0664);
            if(ret == -1) {
                perror("mkfifo");
                exit(0);
            }
        }
    
        ret = access("fifo2", F_OK);
        if(ret == -1) {
            // 文件不存在
            printf("管道不存在，创建对应的有名管道\n");
            ret = mkfifo("fifo2", 0664);
            if(ret == -1) {
                perror("mkfifo");
                exit(0);
            }
        }
    
        // 2.以只写的方式打开管道fifo1
        int fdw = open("fifo1", O_WRONLY);
        if(fdw == -1) {
            perror("open");
            exit(0);
        }
        printf("打开管道fifo1成功，等待写入...\n");
    
        // 3.以只读的方式打开管道fifo2
        int fdr = open("fifo2", O_RDONLY);
        if(fdr == -1) {
            perror("open");
            exit(0);
        }
        printf("打开
        
        管道fifo2成功，等待读取...\n");
        
        char buf[128];
        // 4.循环的写读数据
        while(1) {
            memset(buf, 0, 128);
            // 获取标准输入的数据
            fgets(buf, 128, stdin);
            // 写数据
            ret = write(fdw, buf, strlen(buf));
            if(ret == -1) {
                perror("write");
                exit(0);
            }
    
            // 5.读管道数据
            memset(buf, 0, 128);
            ret = read(fdr, buf, 128);
            if(ret <= 0) {
                perror("read");
                break;
            }
            printf("buf: %s\n", buf);
        }
    
        // 6.关闭文件描述符
        close(fdr);
        close(fdw);
        
        return 0;
    
    }
    ```
  
  - chatB.c
    
    ```cpp
    #include <stdio.h>
    #include <unistd.h>
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <string.h>
    
    int main() {
    
        // 1.判断有名管道文件是否存在
        int ret = access("fifo1", F_OK);
        if(ret == -1) {
            // 文件不存在
            printf("管道不存在，创建对应的有名管道\n");
            ret = mkfifo("fifo1", 0664);
            if(ret == -1) {
                perror("mkfifo");
                exit(0);
            }
        }
    
        ret = access("fifo2", F_OK);
        if(ret == -1) {
            // 文件不存在
            printf("管道不存在，创建对应的有名管道\n");
            ret = mkfifo("fifo2", 0664);
            if(ret == -1) {
                perror("mkfifo");
                exit(0);
            }
        }
    
        // 2.以只读的方式打开管道fifo1
        int fdr = open("fifo1", O_RDONLY);
        if(fdr == -1) {
            perror("open");
            exit(0);
        }
        printf("打开管道fifo1成功，等待读取...\n");
    
        // 3.以只写的方式打开管道fifo2
    
        int fdw = open("fifo2", O_WRONLY);
        if(fdw == -1) {
            perror("open");
            exit(0);
        }
        printf("打开管道fifo2成功，等待写入...\n");
    
        char buf[128];
        // 4.循环的读写数据
        while(1) {
            // 5.读管道数据
            memset(buf, 0, 128);
            ret = read(fdr, buf, 128);
            if(ret <= 0) {
                perror("read");
                break;
            }
            printf("buf: %s\n", buf);
    
            memset(buf, 0, 128);
            // 获取标准输入的数据
            fgets(buf, 128, stdin);
            // 写数据
            ret = write(fdw, buf, strlen(buf));
            if(ret == -1) {
                perror("write");
                exit(0);
            }
        }
    
        // 6.关闭文件描述符
        close(fdr);
        close(fdw);
    
        return 0;
    
    }
    ```

### 内存映射

- 内存映射（Memory-mapped I/O）是将磁盘文件的数据映射到内存，用户通过修改内存就能修改磁盘文件。
    ![Pasted image 20230406105335](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230406105335_11-45-26.png)

- 内存映射实现进程间通信
     ![Pasted image 20230406143334](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230406143334_11-45-38.png)

- 内存映射相关系统调用
  
  - void \***mmap**(void \*addr, size_t length, int prot, int flags, int fd, off_t offset); 
    - 头文件：#include <sys/mman.h>
    - **功能：将一个文件或者设备的数据映射到内存中**
    - 参数：
      - void \*addr: NULL, 由内核指定
      - length : 要映射的数据的长度，这个值不能为0。建议使用文件的长度。
        - 获取文件的长度：stat lseek
      - prot : 对申请的内存映射区的操作权限
        - PROT_EXEC ：可执行的权限
        - PROT_READ ：读权限
        - PROT_WRITE ：写权限
        - PROT_NONE ：没有权限
        - 要操作映射内存，必须要有读的权限。
        - 一般为PROT_READ或PROT_READ | PROT_WRITE
      - flags :
        - MAP_SHARED : **映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项**
        - MAP_PRIVATE ：不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件。（copy on write）
      - fd: 需要映射的那个文件的文件描述符
        - 通过open得到，open的是一个磁盘文件
        - 注意：文件的大小不能为0，open指定的权限不能和prot参数有冲突。
            prot: PROT_READ                open:只读/读写
            prot: PROT_READ | PROT_WRITE   open:读写
      - offset：偏移量，一般不用。必须指定的是4k的整数倍，0表示不偏移。
    - 返回值：
      - 成功返回创建的内存的首地址；
      - 失败返回MAP_FAILED，(void \*) -1
  - int munmap(void \*addr, size_t length);
    - 头文件：#include <sys/mman.h>
    - 功能：释放内存映射
    - 参数：
      - addr : 要释放的内存的首地址
      - length : 要释放的内存的大小，要和mmap函数中的length参数的值一样。

- **使用内存映射实现进程间通信**：
  
  - 有关系的进程（父子进程）
    - 还没有子进程的时候
      - 通过唯一的父进程，先创建内存映射区
    - 有了内存映射区以后，创建子进程
    - 父子进程共享创建的内存映射区
          - 没有关系的进程间通信
    - 准备一个大小不是0的磁盘文件
    - 进程1 通过磁盘文件创建内存映射区
      - 得到一个操作这块内存的指针
    - 进程2 通过磁盘文件创建内存映射区
      - 得到一个操作这块内存的指针
    - 使用内存映射区通信
  - 注意：内存映射区通信，是非阻塞。

- **内存映射注意事项**
    1.如果对mmap的返回值(ptr)做++操作(ptr++), munmap是否能够成功?
  
        void * ptr = mmap(...);
        ptr++;  可以对其进行++操作，但要保存好原地址用于后续释放
  
    2.如果open时O_RDONLY, mmap时prot参数指定PROT_READ | PROT_WRITE会怎样?
  
        错误，返回MAP_FAILED
        open()函数中的权限建议和prot参数的权限保持一致。
  
    3.如果文件偏移量为1000会怎样?
  
        偏移量必须是4K的整数倍，返回MAP_FAILED
  
    4.mmap什么情况下会调用失败?
  
        第二个参数：length = 0
        第三个参数：prot
            1.只指定了写权限
            2.prot PROT_READ | PROT_WRITE；第5个参数fd 通过open函数时指定的 O_RDONLY / O_WRONLY，权限不一致
  
    5.可以open的时候O_CREAT一个新文件来创建映射区吗?
  
        - 可以的，但是创建的文件的大小如果为0的话，肯定不行
        - 可以对新的文件进行扩展
            - lseek()
            - truncate()
  
    6.mmap后关闭文件描述符，对mmap映射有没有影响？
  
        int fd = open("XXX");
        mmap(,,,,fd,0);
        close(fd); 
        映射区还存在，创建映射区的fd被关闭，没有任何影响。mmap进行了拷贝
  
    7.对ptr越界操作会怎样？
  
        void * ptr = mmap(NULL, 100,,,,,);
        分页内存大小为4K
        越界操作操作的是非法的内存 -> 段错误

- 匿名映射
  
  - 使用MAP_ANONYMOUS参数设置匿名映射
  - 在父子进程通信进行使用
  - 不需要文件实体，fd被忽略或设定为-1
  - void * ptr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED | **MAP_ANONYMOUS**, -1, 0)
  - 依然需要使用munmap(ptr, len)进行释放

### 信号

#### 信号的概念

- 信号是 Linux 进程间通信的最古老的方式之一，是事件发生时对进程的通知机制，有时也称之为软件中断，它是在软件层次上对中断机制的一种模拟，是一种**异步通信的方式**。信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某一个突发事件。  
- 发往进程的诸多信号，通常都是源于内核。引发内核为进程产生信号的各类事件如下：  
  - 对于前台进程，用户可以通过输入特殊的终端字符来给它发送信号。比如输入Ctrl+C通常会给进程发送一个中断信号。  
  - 硬件发生异常，即硬件检测到一个错误条件并通知内核，随即再由内核发送相应信号给相关进程。比如执行一条异常的机器语言指令，诸如被 0 除，或者引用了无法访问的内存区域。  
  - 系统状态变化，比如 alarm 定时器到期将引起 SIGALRM 信号，进程执行的 CPU时间超限，或者该进程的某个子进程退出。  
  - 运行 kill 命令或调用 kill 函数
- 使用信号的两个主要**目的**是：  
  - 让进程知道已经发生了一个特定的事情。  
  - 强迫进程执行它自己代码中的信号处理程序。  
- 信号的特点：  
  - 简单  
  - 不能携带大量信息  
  - 满足某个特定条件才发送  
  - 优先级比较高  
- 查看系统定义的信号列表：kill –l  
  - 前 31 个信号为常规信号，其余为实时信号

#### 信号一览表

![Pasted image 20230406150802](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230406150802_20-54-46.png)
![Pasted image 20230406150816](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230406150816_20-54-46.png)

- SIGKILL不可杀死僵尸进程
  ![Pasted image 20230406150830](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230406150830_20-54-46.png)
  ![Pasted image 20230406150846](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230406150846_20-54-46.png)

#### 信号的5种默认处理动作

- 查看信号的详细信息：man 7 signal  
- 信号的 5 中默认处理动作  
  - Term 终止进程  
  - Ign 当前进程忽略掉这个信号  
  - Core 终止进程，并生成一个Core文件（保存错误信息）
  - Stop 暂停当前进程  
  - Cont 继续执行当前被暂停的进程  
- 信号的几种状态：产生、未决(未被处理)、递达（处理完毕）  
- SIGKILL 和 SIGSTOP 信号不能被捕捉、阻塞或者忽略，只能执行默认动作。

#### 信号相关的函数

头文件：#include <sys/types.h>、#include <signal.h>

- int kill(pid_t pid, int sig);  
  
  - 功能：**给任何的进程或者进程组pid, 发送任何的信号 sig**
  - 参数：
    - pid：
      - > 0 : 将信号发送给指定的进程
      - = 0 : 将信号发送给当前的进程组
      - = -1 : 将信号发送给每一个有权限接收这个信号的进程
      - < -1 : 这个pid=某个进程组的ID取反 （-12345）
    - sig : 需要发送的信号的编号或者是宏值，0表示不发送任何信号

- int raise(int sig);  
  
  - 功能：**给当前进程发送信号**
  - 参数：
    - sig : 要发送的信号
  - 返回值：成功 0  失败 非0

- void abort(void);  
  
  - 功能： **发送SIGABRT信号给当前的进程，杀死当前进程**

- unsigned int alarm(unsigned int seconds);
  
  - 头文件：#include <unistd.h>
  - 功能：**设置定时器（闹钟）**。函数调用，开始倒计时，当倒计时为0的时候，函数会给当前的进程发送一个信号：SIGALARM；**SIGALARM** ：默认终止当前的进程，每一个进程都有且只有唯一的一个定时器。
  - 参数：
    - seconds: 倒计时的时长，单位：秒。如果参数为0，定时器无效（不进行倒计时，不发信号）。
      - 取消一个定时器，通过alarm(0)。
  - 返回值
    - 之前没有定时器，返回0
    - 之前有定时器，返回之前的定时器剩余的时间
  - 实际的时间 = 内核时间 + 用户时间 + 用户态到内核态切换的时间
  - **alarm的定时时间包含的是：用户+系统内核的运行时间**
  - 定时器与进程的状态无关（自然定时法）。无论进程处于什么状态，alarm都会计时。

- int setitimer(int which, const struct itimerval \*new_val,struct itimerval \*old_value)
  
  - 头文件：#include <sys/time.h>
  
  - 功能：**设置定时器（闹钟）**。可以替代alarm函数。精度微妙us，可以实现周期性定时
  
  - 参数：
    
    - which : 定时器以什么时间计时
      - ITIMER_REAL: 真实时间，时间到达，发送 SIGALRM   常用
      - ITIMER_VIRTUAL: 用户时间，时间到达，发送 SIGVTALRM
      - ITIMER_PROF: 以该进程在用户态和内核态下所消耗的时间来计算，时间到达，发送 SIGPROF
    - new_value: 设置定时器的属性
    - old_value ：记录上一次的定时的时间参数，一般不使用，指定NULL
  
  - 返回值：成功 0、失败 -1 并设置错误号
    
    ```cpp
    struct itimerval {      // 定时器的结构体
      struct timeval it_interval;  // 每个阶段的时间，间隔时间
      struct timeval it_value;     // 延迟多长时间执行定时器
    };
    struct timeval {        // 时间的结构体
      time_t      tv_sec;     //  秒数    
      suseconds_t tv_usec;    //  微秒    
      };
    ```

#### 信号捕捉函数

- sighandler_t signal(int signum, sighandler_t handler);  ——尽量避免使用
  - 头文件：#include <signal.h>
  - **功能：设置某个信号的捕捉行为**
  - 参数：
    - signum: 要捕捉的信号
    - handler: 捕捉到信号要如何处理
      - SIG_IGN ： 忽略信号
      - SIG_DFL ： 使用信号默认的行为
      - 回调函数 :  这个函数是内核调用，程序员只负责写，捕捉到信号后如何去处理信号。
      - 回调函数：
        -  需要程序员实现，提前准备好的，函数的类型根据实际需求，看函数指针的定义
        - 不是程序员调用，而是当信号产生，由内核调用
        - 函数指针是实现回调的手段，函数实现之后，将函数名放到函数指针的位置就可以了。
  - 返回值：
    - 成功，返回上一次注册的信号处理函数的地址。第一次调用返回NULL
    - 失败，返回SIG_ERR，设置错误号
  -  SIGKILL SIGSTOP不能被捕捉，不能被忽略。
- int sigaction(int signum, const struct sigaction \*act,struct sigaction \*oldact);
  - 功能：**检查或修改与指定信号相关联的处理动作（可同时两种操作**
  - 参数：
    - signum：需要捕捉的信号的编号或者宏值（信号的名称）
    - act：需要捕捉的信号的编号或者宏值（信号的名称）
    - oldact：需要捕捉的信号的编号或者宏值（信号的名称）
    - 如果act指针非空，则要改变指定信号的处理方式，如果oldact指针非空 则系统将此前指定信号的处理方式引入 oldact
  - 返回值：0 表示成功，-1 表示有错误发生

```cpp
struct sigaction {
    void (*sa_handler)(int);
    void (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t sa_mask;
    int sa_flags;
    void (*sa_restorer)(void);
```

- sa_handler此参数和signal()的参数handler相同，函数指针，指向的函数就是信号捕捉到之后的处理函数
- sa_mask 临时阻塞信号集，在信号捕捉函数执行过程中，临时阻塞某些信号。
- sa_flags 用来设置信号处理的其他相关操作，下列的数值可用。 
  - SA_RESETHAND：当调用信号处理函数时，将信号的处理函数重置为缺省值SIG_DFL
  - SA_RESTART：如果信号中断了进程的某个系统调用，则系统自动启动该系统调用
  - SA_NODEFER ：一般情况下， 当信号处理函数运行时，内核将阻塞该给定信号。但是如果设置了 SA_NODEFER标记， 那么在该信号处理函数运行时，内核将不会阻塞该信号
  - 这个值可以是0，表示使用sa_handler,也可以是SA_SIGINFO表示使用sa_sigaction

```ad-note
用sigaction只能捕捉一次SIGALRM的原因在于getchar被中断；
打印getchar的返回值为-1，perror打印其错误信息：Interrupted system call，对应的错误码为EINTR
在man文档第7章
man 7 signal

Interruption of system calls and library functions by signal handlers

If  a signal handler is invoked while a system call or library function call is blocked, then either:  
 * the call is automatically restarted after the signal handler returns; or  
 * the call fails with the error EINTR.

我们的sigaction中的sa_flags未设置SA_RESTART所以会是第二种行为，系统调用被中断，下面有讲到read从终端读取数据是会被signal handler中断，getchar底层应该调用的是read。
```

#### 内核实现信号捕捉的过程

![Pasted image 20230406160053](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230406160053_20-54-46.png)

#### 信号集

- 许多信号相关的系统调用都需要能表示一组不同的信号，多个信号可使用一个称之为**信号集的数据结构**来表示，其系统数据类型为 **sigset_t**。  

- 在 PCB 中有两个非常重要的信号集。一个称之为 “阻塞信号集” ，另一个称之为“未决信号集” 。这两个信号集都是内核使用位图机制来实现的。但操作系统不允许我们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数来对 PCB 中的这两个信号集进行修改。  

- 信号的 “未决” 是一种状态，指的是从信号的产生到信号被处理前的这一段时间。  

- 信号的 “阻塞” 是一个开关动作，指的是阻止信号被处理，但不是阻止信号产生。  

- 信号的阻塞就是让系统暂时保留信号留待以后发送。由于另外有办法让系统忽略信号，所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作
    ![Pasted image 20230406155848|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230406155848_11-45-52.png)

- 用户通过键盘  Ctrl + C, 产生2号信号SIGINT (信号被创建)

- 信号产生但是没有被处理 （未决）
  
  - 在内核中将所有的没有被处理的信号存储在一个集合中 （未决信号集）
  - SIGINT信号状态被存储在第二个标志位上
    - 这个标志位的值为0， 说明信号不是未决状态
    - 这个标志位的值为1， 说明信号处于未决状态

- 这个未决状态的信号，需要被处理，处理之前需要和另一个信号集（阻塞信号集），进行比较
  
  - 阻塞信号集默认不阻塞任何的信号
  - 如果想要阻塞某些信号需要用户调用系统的API

- 在处理的时候和阻塞信号集中的标志位进行查询，看是不是对该信号设置阻塞了
  
  - 如果没有阻塞，这个信号就被处理
  - 如果阻塞了，这个信号就继续处于未决状态，直到阻塞解除，这个信号就被处理

#### 信号集相关函数

- int sigemptyset(sigset_t \*set); 
  
  - **功能**：清空信号集中的数据,将信号集中的所有的标志位置为0
  - 参数：set,传出参数，需要操作的信号集
  - 返回值：成功返回0， 失败返回-1

- int sigfillset(sigset_t \*set);  
  
  - **功能**：将信号集中的所有的标志位置为1
  - 参数：set,传出参数，需要操作的信号集
  - 返回值：成功返回0， 失败返回-1

- int sigaddset(sigset_t \*set, int signum);  
  
  - **功能**：设置信号集中的某一个信号对应的标志位为1，表示阻塞这个信号
  - 参数：
    - set：传出参数，需要操作的信号集
    - signum：需要设置阻塞的那个信号
  - 返回值：成功返回0， 失败返回-1

- int sigdelset(sigset_t \*set, int signum);  
  
  - **功能**：设置信号集中的某一个信号对应的标志位为0，表示不阻塞这个信号
  - 参数：
    - set：传出参数，需要操作的信号集
    - signum：需要设置不阻塞的那个信号
  - 返回值：成功返回0， 失败返回-1

- int sigismember(const sigset_t \*set, int signum);  
  
  - **功能**：判断某个信号是否阻塞
  - 参数：
    - set：需要操作的信号集
    - signum：需要判断的那个信号
  - 返回值：
      1 ： signum被阻塞
      0 ： signum不阻塞
      -1 ： 失败

- int sigprocmask(int how, const sigset_t \*set, sigset_t \*oldset);  
  
  - **功能**：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
  - 参数：
    - how : 如何对内核阻塞信号集进行处理
      - SIG_BLOCK: 将用户设置的阻塞信号集添加到内核中，内核中原来的数据不变
          假设内核中默认的阻塞信号集是mask， mask | set
      - SIG_UNBLOCK: 根据用户设置的数据，对内核中的数据进行解除阻塞
          mask &= ~set
      - SIG_SETMASK:覆盖内核中原来的值
    - set ：已经初始化好的用户自定义的信号集
    - oldset : 保存设置之前的内核中的阻塞信号集的状态，可以是 NULL
  - 返回值：
                成功：0   失败：-1
                设置错误号：EFAULT、EINVAL

- int sigpending(sigset_t \*set);
  
  - **功能**：获取内核中的未决信号集
  - 参数：set,传出参数，保存的是内核中的未决信号集中的信息。

- SIGCHLD信号
  
  - SIGCHLD信号产生的条件  
    - 子进程终止时  
    - 子进程接收到 SIGSTOP 信号停止时  
    - 子进程处在停止态，接受到SIGCONT后唤醒时  
  - 以上三种条件都会给父进程发送 SIGCHLD 信号，父进程默认会忽略该信号

### 共享内存

#### 定义

- 共享内存允许两个或者多个进程共享物理内存的**同一块区域**（通常被称为段）。由于一个共享内存段会称为一个进程用户空间的一部分，因此这种 **IPC 机制无需内核介入**。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其他所有共享同一个段的进程可用。  
- 与管道等要求发送进程将数据从用户空间的缓冲区复制进内核内存和接收进程将数据从内核内存复制进用户空间的缓冲区的做法相比，这种 IPC 技术的速度更快

#### 共享内存使用步骤

- 调用 shmget() **创建**一个新共享内存段或取得一个既有共享内存段的标识符（即由其他进程创建的共享内存段）。这个调用将返回后续调用中需要用到的**共享内存标识符**。  
- 使用 shmat() 来**附加共享内存段**，即使该段成为调用进程的虚拟内存的一部分。 
- 此刻在程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存，程序需要使用由 shmat() 调用返回的 addr 值，它是一个指向进程的虚拟地址空间中该共享内存段的起点的指针。  
- 调用 shmdt() 来**分离共享内存段**。在这个调用之后，进程就无法再引用这块共享内存了。这一步是可选的，并且在进程终止时会自动完成这一步。  
- 调用 shmctl() 来**删除共享内存段**。只有当当前所有附加内存段的进程都与之分离之后内存段才会销毁。只有一个进程需要执行这一步

#### 共享内存操作函数

- 头文件：#include <sys/ipc.h>、#include <sys/shm.h>

- int shmget(key_t key, size_t size, int shmflg);
  
  - **功能**：创建一个新的共享内存段，或者获取一个既有的共享内存段的标识。新创建的内存段中的数据都会被初始化为0
  - 参数：
    - key : key_t类型是一个整形，通过这个找到或者创建一个共享内存。一般使用16进制表示，非0值
    - size: 共享内存的大小
    - shmflg: 属性
      - 访问权限
      - 附加属性：创建/判断共享内存是不是存在
        - 创建：IPC_CREAT
        - 判断共享内存是否存在： IPC_EXCL , 需要和IPC_CREAT一起使用
            IPC_CREAT | IPC_EXCL | 0664
  - 返回值：
    - 失败：-1 并设置错误号
    - 成功：>0 返回共享内存的引用的ID，后面操作共享内存都是通过这个值。

- void \*shmat(int shmid, const void \*shmaddr, int shmflg);
  
  - **功能**：和当前的进程进行关联
  - 参数：
    - shmid : 共享内存的标识（ID）,由shmget返回值获取
    - shmaddr: 申请的共享内存的起始地址，指定NULL，内核指定
    - shmflg : 对共享内存的操作
      - 读 ： SHM_RDONLY, 必须要有读权限
      - 读写： 0
  - 返回值：
      成功：返回共享内存的首（起始）地址。  失败(void *) -1

- int shmdt(const void \*shmaddr);
  
  - **功能**：解除当前进程和共享内存的关联
  - 参数：
      shmaddr：共享内存的首地址
  - 返回值：成功 0， 失败 -1

- int shmctl(int shmid, int cmd, struct shmid_ds \*buf);
  
  - **功能**：对共享内存进行操作。删除共享内存，共享内存要删除才会消失，创建共享内存的进行被销毁了对共享内存是没有任何影响。
  - 参数：
    - shmid: 共享内存的ID
    - cmd : 要做的操作
      - IPC_STAT : 获取共享内存的当前的状态
      - IPC_SET : 设置共享内存的状态
      - IPC_RMID: 标记共享内存被销毁
    - buf：需要设置或者获取的共享内存的属性信息
      - IPC_STAT : buf存储数据
      - IPC_SET : buf中需要初始化数据，设置到内核中
      - IPC_RMID : 没有用，NULL

- key_t ftok(const char \*pathname, int proj_id);
  
  - **功能**：根据指定的路径名，和int值，生成一个共享内存的key
  
  - 参数：
    
    - pathname:指定一个存在的路径
        /home/nowcoder/Linux/a.txt
    
    - proj_id: int类型的值，但是这系统调用只会使用其中的1个字节
      
           范围 ： 0-255  一般指定一个字符 'a'

问题1：操作系统如何知道一块共享内存被多少个进程关联？

- 共享内存维护了一个结构体struct shmid_ds 这个结构体中有一个成员 shm_nattch
- shm_nattach 记录了关联的进程个数

问题2：可不可以对共享内存进行多次删除 shmctl

- 可以的
- 因为shmctl 标记删除共享内存，不是直接删除
- 什么时候真正删除呢?
    当和共享内存关联的进程数为0的时候，就真正被删除
- 当共享内存的key为0的时候，表示共享内存被标记删除了
    如果一个进程和共享内存取消关联，那么这个进程就不能继续操作这个共享内存。也不能进行关联。

#### 共享内存操作命令

- ipcs 用法  
  - ipcs -a // 打印当前系统中所有的进程间通信方式的信息  
  - ipcs -m // 打印出使用共享内存进行进程间通信的信息  
  - ipcs -q // 打印出使用消息队列进行进程间通信的信息  
  - ipcs -s // 打印出使用信号进行进程间通信的信息  
- ipcrm 用法  
  - ipcrm -M shmkey // 移除用shmkey创建的共享内存段  
  - ipcrm -m shmid // 移除用shmid标识的共享内存段  
  - ipcrm -Q msgkey // 移除用msqkey创建的消息队列  
  - ipcrm -q msqid // 移除用msqid标识的消息队列  
  - ipcrm -S semkey // 移除用semkey创建的信号  
  - ipcrm -s semid // 移除用semid标识的信号

#### 共享内存和内存映射的区别

1.共享内存可以直接创建，内存映射需要磁盘文件（匿名映射除外）
2.共享内存效果更高
3.内存
    所有的进程操作的是同一块共享内存。
    内存映射，每个进程在自己的虚拟地址空间中有一个独立的内存。
4.数据安全

- 进程突然退出
    共享内存还存在
    内存映射区消失

- 运行进程的电脑死机，宕机了
    数据存在在共享内存中，没有了
    内存映射区的数据 ，由于磁盘文件中的数据还在，所以内存映射区的数据还存在。
  5.生命周期

- 内存映射区：进程退出，内存映射区销毁

- 共享内存：进程退出，共享内存还在，标记删除（所有的关联的进程数为0），或者关机
  
        如果一个进程退出，会自动和共享内存进行取消关联。

## 线程

### 概述

- 与进程（process）类似，线程（thread）是允许应用程序并发执行多个任务的一种机制。**一个进程可以包含多个线程。同一个程序中的所有线程均会独立执行相同程序，且共享同一份全局内存区域，其中包括初始化数据段、未初始化数据段，以及堆内存段**。（传统意义上的 UNIX 进程只是多线程程序的一个特例，该进程只包含一个线程）  
- 进程是 CPU 分配资源的最小单位，线程是操作系统调度执行的最小单位。  
- 线程是轻量级的进程（LWP：Light Weight Process），在 Linux 环境下线程的本质仍是进程。  
- 查看指定进程的 LWP 号(线程)：ps –Lf pid

### 线程和进程区别

- **进程间的信息难以共享**。由于除去只读代码段外，父子进程并未共享内存，因此必须采用一些进程间通信方式，在进程间进行信息交换。  
- 调用 fork() 来创建进程的代价相对较高，即便利用写时复制技术，仍然需要复制诸如内存页表和文件描述符表之类的多种进程属性，这意味着 fork() 调用在时间上的开销依然不菲。  
- **线程之间能够方便、快速地共享信息**。只需将数据复制到共享（全局或堆）变量中即可。  
- **创建线程比创建进程通常要快 10 倍甚至更多**。线程间是共享虚拟地址空间的，无需采用写时复制来复制内存，也无需复制页表

### 线程和进程虚拟地址空间

- ./text和栈空间会根据线程分配好子空间，其他空间可以共享

### 线程之间共享和非共享资源

- 共享资源  
  - 进程 ID 和父进程 ID  
  - 进程组 ID 和会话 ID  
  - 用户 ID 和 用户组 ID  
  - 文件描述符表  
  - 信号处置  
  - 文件系统的相关信息：文件权限掩码(umask）、当前工作目录  
  - 虚拟地址空间（除栈、.text）  
- 非共享资源  
  - 线程 ID  
  - 信号掩码  
  - 线程特有数据  
  - error 变量  
  - 实时调度策略和优先级  
  - 栈，本地变量和函数的调用链接信息

### NPTL

- 当 Linux 最初开发时，在内核中并不能真正支持线程。但是它的确可以通过 clone()系统调用将进程作为可调度的实体。这个调用创建了调用进程（calling process）的一个拷贝，这个拷贝与调用进程共享相同的地址空间。LinuxThreads 项目使用这个调用来完成在用户空间模拟对线程的支持。不幸的是，这种方法有一些缺点，尤其是在信号处理、调度和进程间同步等方面都存在问题。另外，这个线程模型也不符合 POSIX 的要求。  
- 要改进 LinuxThreads，需要内核的支持，并且重写线程库。有两个相互竞争的项目开始来满足这些要求。一个包括 IBM 的开发人员的团队开展了 NGPT（Next-Generation POSIX Threads）项目。同时，Red Hat 的一些开发人员开展了 NPTL 项目。NGPT在 2003 年中期被放弃了，把这个领域完全留给了 NPTL。  
- NPTL，或称为 Native POSIX Thread Library，是 Linux 线程的一个新实现，它克服了 LinuxThreads 的缺点，同时也符合 POSIX 的需求。与 LinuxThreads 相比，它在性能和稳定性方面都提供了重大的改进。  
- 查看当前 pthread 库版本：getconf GNU_LIBPTHREAD_VERSION

### 线程操作

- 头文件：

- int pthread_create(pthread_t \*thread, const pthread_attr_t \*attr,  void \*(\*start_routine) (void \*), void \*arg);  
  
  - **功能**：创建一个子线程
  - 参数：
    - thread：传出参数，线程创建成功后，子线程的线程ID被写到该变量中。
    - attr : 设置线程的属性，一般使用默认值，NULL
    - start_routine : 函数指针，这个函数是子线程需要处理的逻辑代码
    - arg : 给第三个参数使用，传参
  - 返回值：
      成功：0
      失败：返回错误号。这个错误号和之前errno不太一样。
      获取错误号的信息：  char * strerror(int errnum);

- pthread_t pthread_self(void); 
  
  - **功能**：获取当前的线程的线程ID

- int pthread_equal(pthread_t t1, pthread_t t2);  
  
  - **功能**：比较两个线程ID是否相等
      不同的操作系统，pthread_t类型的实现不一样，有的是无符号的长整型，有的是使用结构体去实现的。

- void pthread_exit(void \*retval); 
  
  - **功能**：终止一个线程，在哪个线程中调用，就表示终止哪个线程
  - 参数：
      retval:需要传递一个指针，作为一个返回值，可以在pthread_join()中获取到。

- int pthread_join(pthread_t thread, void \*\*retval);  
  
  - **功能**：
    - 和一个已经终止的线程进行连接
    - 回收子线程的资源
    - 这个函数是**阻塞函数**，调用一次只能回收一个子线程、一般在主线程中使用
  - 参数：
    - thread：需要回收的子线程的ID
    - retval:  接收子线程退出时的返回值
  - 返回值：
      0 : 成功
      非0 : 失败，返回的错误号

- int pthread_detach(pthread_t thread);  
  
  - **功能**：分离一个线程。被分离的线程在终止的时候，会自动释放资源返回给系统；非阻塞
    - 不能多次分离，会产生不可预料的行为。
    - 不能去连接一个已经分离的线程，会报错。
  - 参数：需要分离的线程的ID
  - 返回值：
      成功：0
      失败：返回错误号

- int pthread_cancel(pthread_t thread);
  
  - **功能**：取消线程（让线程终止）
    
    - 取消某个线程，可以终止某个线程的运行，
    
    - 但是并不是立马终止，而是当子线程执行到一个取消点，线程才会终止。
    
    - 取消点：系统规定好的一些系统调用，我们可以粗略的理解为从用户区到内核区的切换，这个位置称之为取消点。
      
      ### 线程属性

- 线程属性类型 pthread_attr_t  

- int pthread_attr_init(pthread_attr_t \*attr);  //初始化线程属性变量

- int pthread_attr_destroy(pthread_attr_t \*attr);  //释放线程属性的资源

- int pthread_attr_getdetachstate(const pthread_attr_t \*attr, int  \*detachstate);  //获取线程分离的状态属性

- int pthread_attr_setdetachstate(pthread_attr_t \*attr, int detachstate);//设置线程分离的状态属性

### 线程同步

- 线程的主要优势在于，能够通过全局变量来共享信息。不过，这种便捷的共享是有代价的：必须确保多个线程不会同时修改同一变量，或者某一线程不会读取正在由其他线程修改的变量。  
- 临界区是指访问某一共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一共享资源的其他线程不应中断该片段的执行。  
- **线程同步**：即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作，其他线程才能对该内存地址进行操作，而其他线程则处于等待状态。

#### 互斥量

- 为避免线程更新共享变量时出现问题，可以使用互斥量（mutex 是 mutual exclusion的缩写）来确保同时仅有一个线程可以访问某项共享资源。**可以使用互斥量来保证对任意共享资源的原子访问**。  
- 互斥量有两种状态：已锁定（locked）和未锁定（unlocked）。任何时候，至多只有一个线程可以锁定该互斥量。试图对已经锁定的某一互斥量再次加锁，将可能阻塞线程或者报错失败，具体取决于加锁时使用的方法。  
- 一旦线程锁定互斥量，随即成为该互斥量的所有者，只有所有者才能给互斥量解锁。一般情况下，对每一共享资源（可能由多个相关变量组成）会使用不同的互斥量，每一线程在访问同一资源时将采用如下协议：  
  - 针对共享资源锁定互斥量  
  - 访问共享资源  
  - 对互斥量解锁
- 如果多个线程试图执行这一块代码（一个临界区），事实上只有一个线程能够持有该互斥量（其他线程将遭到阻塞），即同时只有一个线程能够进入这段代码区域，如下图所示
    ![Pasted image 20230410090233|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410090233_11-46-14.png)

#### 互斥量相关操作函数

- 互斥量的类型 **pthread_mutex_t**  
- int **pthread_mutex_init**(pthread_mutex_t \*restrict mutex,const pthread_mutexattr_t \*restrict attr);  
  - 功能：初始化互斥量
  - 参数 ：
    - mutex ： 需要初始化的互斥量变量
    - attr ： 互斥量相关的属性，NULL
  - 返回值：
    - 成功：返回0
    - 错误：返回错误编号
      - EAGAIN   缺少关键资源初始化互斥量
      - ENOMEN  内存不足
      - EPERM 调用者没有执行该操作的权限
  - restrict : C语言的修饰符，被修饰的指针，不能由另外的一个指针进行操作。
      pthread_mutex_t \*restrict mutex = xxx;
      pthread_mutex_t \* mutex1 = mutex;   //操作失败
- int **pthread_mutex_destroy**(pthread_mutex_t \*mutex); 
  - 功能：释放互斥量资源
- int **pthread_mutex_lock**(pthread_mutex_t \*mutex); 
  - 功能：加锁，阻塞的，如果有一个线程加锁了，那么其他的线程只能阻塞等待
  - 返回值：成功返回0，错误返回错误代码
- int **pthread_mutex_trylock**(pthread_mutex_t \*mutex);  
  - 功能：尝试加锁，如果加锁失败，不会阻塞，会直接返回。
  - 返回值：成功返回0，错误返回错误代码
- int **pthread_mutex_unlock**(pthread_mutex_t \*mutex);
  - 功能：解锁
  - 返回值：成功返回0，错误返回错误代码

#### 死锁

- 有时，一个线程需要同时访问两个或更多不同的共享资源，而每个资源又都由不同的互斥量管理。当超过一个线程加锁同一组互斥量时，就有可能发生死锁。  
- 两个或两个以上的进程在执行过程中，因争夺共享资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁。
- 死锁的几种场景：  
  - 忘记释放锁  
  - 重复加锁  
  - 多线程多锁，抢占锁资源
    ![Pasted image 20230410090537|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410090537_11-46-32.png)

#### 读写锁

- 当有一个线程已经持有互斥锁时，互斥锁将所有试图进入临界区的线程都阻塞住。但是考虑一种情形，当前持有互斥锁的线程只是要读访问共享资源，而同时有其它几个线程也想读取这个共享资源，但是由于互斥锁的排它性，所有其它线程都无法获取锁，也就无法读访问共享资源了，但是实际上多个线程同时读访问共享资源并不会导致问题。  
- 在对数据的读写操作中，更多的是读操作，写操作较少，例如对数据库数据的读写应用。**为了满足当前能够允许多个读出，但只允许一个写入的需求，线程提供了读写锁来实现**。  
- 读写锁的特点：  
  - 如果有其它线程读数据，则允许其它线程执行读操作，但不允许写操作。  
  - 如果有其它线程写数据，则其它线程都不允许读、写操作。  
  - 写是独占的，写的优先级高

#### 读写锁相关操作函数

- 读写锁的类型 **pthread_rwlock_t**  
- int **pthread_rwlock_init**(pthread_rwlock_t \*restrict rwlock, const pthread_rwlockattr_t \*restrict attr);   //初始化
- int **pthread_rwlock_destroy**(pthread_rwlock_t \*rwlock);  //释放资源
- int **pthread_rwlock_rdlock**(pthread_rwlock_t \*rwlock);  //加读锁
- int **pthread_rwlock_tryrdlock**(pthread_rwlock_t \*rwlock);  //尝试加上读锁
- int **pthread_rwlock_wrlock**(pthread_rwlock_t \*rwlock);  //加写锁
- int **pthread_rwlock_trywrlock**(pthread_rwlock_t \*rwlock);  //尝试加写锁
- int **pthread_rwlock_unlock**(pthread_rwlock_t \*rwlock);//解锁

#### 生产者消费者模型

- 在多线程开发中，如果生产者生产数据的速度很快，而消费者消费数据的速度很慢，那么生产者就必须等待消费者消费完数据才能够继续生产数据，因为生产过多的数据可能会导致存储不足；同理如果消费者的速度大于生产者那么消费者就会经常处理等待状态，所以为了达到生产者和消费者生产数据和消费数据之间的平衡，那么就需要一个缓冲区用来存储生产者生产的数据，所以就引入了生产者-消费者模式
- 简单来说，这里缓冲区的作用就是为了平衡生产者和消费者的数据处理能力，一方面起到缓存作用，另一方面达到解耦合作用。
    ![Pasted image 20230410090911|400](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410090911_11-46-47.png)
- 特点：
  - 保证生产者不会在缓冲区满的时候继续向缓冲区放入数据，而消费者也不会在缓冲区空的时候，消耗数据
  - 当缓冲区满的时候，生产者会进入休眠状态，当下次消费者开始消耗缓冲区的数据时，生产者才会被唤醒，开始往缓冲区中添加数据；当缓冲区空的时候，消费者也会进入休眠状态，直到生产者往缓冲区中添加数据时才会被唤醒
- 优点：
  - 解耦合：将生产者类和消费者类进行解耦，消除代码之间的依赖性，简化工作负载的管理。
  - 复用：通过将生产者类和消费者类独立开来，那么可以对生产者类和消费者类进行独立的复用与扩展。
  - 调整并发数：由于生产者和消费者的处理速度是不一样的，可以调整并发数，给予慢的一方多的并发数，来提高任务的处理速度。
  - 异步：对于生产者和消费者来说能够各司其职，生产者只需要关心缓冲区是否还有数据，不需要等待消费者处理完；同样的对于消费者来说，也只需要关注缓冲区的内容，不需要关注生产者，通过异步的方式支持高并发，将一个耗时的流程拆成生产和消费两个阶段，这样生产者因为执行 put() 的时间比较短，而支持高并发。
  - 支持分布式：生产者和消费者通过队列进行通讯，所以不需要运行在同一台机器上，在分布式环境中可以通过 redis 的 list 作为队列，而消费者只需要轮询队列中是否有数据。同时还能支持集群的伸缩性，当某台机器宕掉的时候，不会导致整个集群宕掉。

#### 条件变量  — 配合互斥量使用

- 条件变量的类型 **pthread_cond_t**  
  - 满足某个条件后引起阻塞
  - 满足某个条件后释放阻塞
- int **pthread_cond_init**(pthread_cond_t \*restrict cond, const pthread_condattr_t \*restrict attr);  //初始化
- int **pthread_cond_destroy**(pthread_cond_t \*cond);  //释放资源
- int **pthread_cond_wait**(pthread_cond_t \*restrict cond,pthread_mutex_t \*restrict mutex);  //等待，调用了该函数，线程会阻塞。
- int **pthread_cond_timedwait**(pthread_cond_t \*restrict cond, pthread_mutex_t \*restrict mutex, const struct timespec \*restrict abstime);   //等待多长时间，调用了这个函数，线程会阻塞，直到指定的时间结束。
- int **pthread_cond_signal**(pthread_cond_t \*cond);    //唤醒一个或者多个等待的线程
- int **pthread_cond_broadcast**(pthread_cond_t \*cond);//唤醒所有的等待的线程

#### 信号量

- 信号量的类型 sem_t  
- int **sem_init**(sem_t \*sem, int pshared, unsigned int value);  
  - 初始化信号量
  - 参数：
    - sem : 信号量变量的地址
    - pshared : 0 用在线程间 ，非0 用在进程间
    - value : 信号量中的值
- int **sem_destroy**(sem_t \*sem);   //释放资源
- int **sem_wait**(sem_t \*sem);  //对信号量加锁，调用一次对信号量的值-1，如果值为0，就阻塞
- int **sem_trywait**(sem_t \*sem);  //尝试阻塞
- int **sem_timedwait**(sem_t \*sem, const struct timespec \*abs_timeout);  //设置阻塞时间
- int **sem_post**(sem_t \*sem);   //对信号量解锁，调用一次对信号量的值+1
- int **sem_getvalue**(sem_t \*sem, int \*sval); //获取信号量的值

## socket编程

### socket 介绍

- 所谓 socket（套接字），就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。  
- 一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位来讲，套接字上联应用进程，下联网络协议栈，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议根进行交互的接口。  
- socket 可以看成是两个网络应用程序进行通信时，各自通信连接中的端点，这是一个逻辑上的概念。它是网络环境中进程间通信的 API，也是可以被命名和寻址的通信端点，使用中的每一个套接字都有其类型和一个与之相连进程。通信时其中一个网络应用程序将要传输的一段信息写入它所在主机的 socket 中，该 socket 通过与网络接口卡（NIC）相连的传输介质将这段信息送到另外一台主机的 socket 中，使对方能够接收到这段信息。**socket 是由 IP 地址和端口结合的，提供向应用层进程传送数据包的机制**。  
- socket 本身有“插座”的意思，在 Linux 环境下，**用于表示进程间网络通信的特殊文件类型。本质为内核借助缓冲区形成的伪文件**。既然是文件，那么理所当然的，我们可以使用文件描述符引用套接字。与管道类似的，Linux 系统将其封装成文件的目的是为了统一接口，使得读写套接字和读写文件的操作一致。区别是管道主要应用于本地进程间通信，而套接字多应用于网络进程间数据的传递。  
  ```c
  // 套接字通信分两部分：  
- 服务器端：被动接受连接，一般不会主动发起连接  
- 客户端：主动向服务器发起连接  

socket是一套通信的接口，Linux 和 Windows 都有，但是有一些细微的差别

```
### 字节序
现代 CPU 的累加器一次都能装载（至少）4 字节（这里考虑 32 位机），即一个整数。那么这 4字节在内存中排列的顺序将影响它被累加器装载成的整数的值，这就是字节序问题。在各种计算机体系结构中，对于字节、字等的存储机制有所不同，因而引发了计算机通信领域中一个很重要的问题，即通信双方交流的信息单元（比特、字节、字、双字等等）应该以什么样的顺序进行传送。如果不达成一致的规则，通信双方将无法进行正确的编码/译码从而导致通信失败。 

- **字节序，顾名思义字节的顺序，就是大于一个字节类型的数据在内存中的存放顺序**(一个字节的数据当然就无需谈顺序的问题了)。  
- 字节序分为大端字节序（Big-Endian） 和小端字节序（Little-Endian）。
    - **大端字节序**是指一个整数的最高位字节（23 ~ 31 bit）存储在内存的低地址处，低位字节（0 ~ 7 bit）存储在内存的高地址处；
    - **小端字节序**则是指整数的高位字节存储在内存的高地址处，而低位字节则存储在内存的低地址处。
- 高低地址
    - C程序映射中内存的空间布局大致如下：
        - 最高内存地址 0xFFFFFFFF 
        -  栈区（从高内存地址，往低内存地址发展。即栈底在高地址，栈顶在低地址）
        - 堆区（从低内存地址 ，往高内存地址发展）
        - 全局区（常量和全局变量）
        - 代码区
        - 最低内存地址 0x0000000

- 高低字节
    - 在十进制中靠左边的是高位，靠右边的是低位，在其他进制也是如此。例如 0x12345678，从高位到低位的字节依次是0x12、0x34、0x56和0x78。
    - 网络字节序 就是 大端字节序：4个字节的32 bit值以下面的次序传输，首先是0～7bit，其次8～15bit，然后16～23bit，最后是24~31bit
    - 主机字节序 就是 小端字节序，现代PC大多采用小端字节序
    - 对于数据 0x12345678，假设从地址0x4000开始存放，在大端和小端模式下，存放的位置分别为：
    ![Pasted image 20230410104731|400](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410104731_11-47-11.png)

#### 字节序转换函数
- 当格式化的数据在两台使用不同字节序的主机之间直接传递时，接收端必然错误的解释之。解决问题的方法是：发送端总是把要发送的数据转换成大端字节序数据后再发送，而接收端知道对方传送过来的数据总是采用大端字节序，所以接收端可以根据自身采用的字节序决定是否对接收到的数据进行转换（小端机转换，大端机不转换）。
- 网络字节顺序是 TCP/IP 中规定好的一种数据表示格式，它与具体的 CPU 类型、操作系统等无关，从而可以保证数据在不同主机之间传输时能够被正确解释，网络字节顺序采用大端排序方式。  
- BSD Socket提供了封装好的转换接口，方便程序员使用。包括：
    - 从主机字节序到网络字节序的转换函数：htons、htonl；
    - 从网络字节序到主机字节序的转换函数：ntohs、ntohl。
```cpp
#include <arpa/inet.h>  
//h表示host，to为转换，n为network
//s - short unsigned short  
//l - long unsigned int

// 转换端口  
uint16_t htons(uint16_t hostshort); // 主机字节序 - 网络字节序  
uint16_t ntohs(uint16_t netshort); // 网络字节序 - 主机字节序   
// 转IP  
uint32_t htonl(uint32_t hostlong); // 主机字节序 - 网络字节序  
uint32_t ntohl(uint32_t netlong); // 网络字节序 - 主机字节序 
```

### socket 地址

socket地址其实是一个结构体，封装端口号和IP等信息。后面的socket相关的api中需要使用到这个socket地址。客户端 -> 服务器（IP, Port）

#### 通用 socket 地址

socket 网络编程接口中表示 socket 地址的是结构体 sockaddr(用于IPV4)，其定义如下：

```cpp
#include <bits/socket.h>  
struct sockaddr {  
    sa_family_t sa_family;  
    char sa_data[14];  
};  
typedef unsigned short int sa_family_t;
```

- **sa_family** 成员是地址族类型（sa_family_t）的变量。地址族类型通常与协议族类型对应。

常见的协议族（protocol family，也称 domain）和对应的地址族入下所示：
| 协议族  | 地址族   | 描述             |
| ------- | -------- | ---------------- |
| PF_UNIX | AF_UNIX  | UNIX本地域协议族 |
| PF_INE  | TAF_INET | TCP/IPv4协议族   |
| PF_INET6 | AF_INET6  |  TCP/IPv6协议族   |

 宏 PF_* 和 AF_* 都定义在 bits/socket.h 头文件中，且后者与前者有完全相同的值，所以二者通常混用。

- **sa_data** 成员用于存放 socket 地址值。但是，不同的协议族的地址值具有不同的含义和长度，如下所示:
  
  | 协议族      | 地址值含义和长度                                                   |
  | -------- | ---------------------------------------------------------- |
  | PF_UNIX  | 文件的路径名，长度可达到108字节                                          |
  | PF_INET  | 16 bit 端口号和 32 bit IPv4 地址，共 6 字节                          |
  | PF_INET6 | 16 bit 端口号，32 bit 流标识，128 bit IPv6 地址，32 bit 范围 ID，共 26 字节 |

由上表可知，14 字节的 sa_data 根本无法容纳多数协议族的地址值。因此，Linux 定义了下面这个新的通用的 socket 地址结构体，这个结构体不仅提供了足够大的空间用于存放地址值，而且是内存对齐的:

```cpp
#include <bits/socket.h>  
struct sockaddr_storage  
{  
    sa_family_t sa_family;  
    unsigned long int __ss_align;  
    char __ss_padding[ 128 - sizeof(__ss_align) ];  
};  
typedef unsigned short int sa_family_t;
```

#### 专用 socket 地址

很多网络编程函数诞生早于 IPv4 协议，那时候都使用的是 struct sockaddr 结构体，为了向前兼容，现在sockaddr 退化成了（void \*）的作用，传递一个地址给函数，至于这个函数是 sockaddr_in 还是sockaddr_in6，由地址族确定，然后函数内部再强制类型转化为所需的地址类型。
    ![Pasted image 20230410111146|400](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410111146_11-47-41.png)

- **UNIX 本地域协议族**使用如下专用的 socket 地址结构体
  
  ```cpp
  #include <sys/un.h>  
  struct sockaddr_un  
  {  
    sa_family_t sin_family; //地址族类型 
    char sun_path[108];  
  }
  ```

- **TCP/IP 协议族**有 sockaddr_in 和 sockaddr_in6 两个专用的 socket 地址结构体，它们分别用于 IPv4 和IPv6：
  
  ```cpp
  #include <netinet/in.h>  
  struct sockaddr_in  
  {  
    //地址族类型
    sa_family_t sin_family; /* __SOCKADDR_COMMON(sin_) */ 
    //端口号（2字节） 
    in_port_t sin_port; /* Port number. */  
    //IP地址（4字节）
    struct in_addr sin_addr; /* Internet address. */  
    /* Pad to size of `struct sockaddr'. */  
    unsigned char sin_zero[sizeof (struct sockaddr) - __SOCKADDR_COMMON_SIZE -  
    sizeof (in_port_t) - sizeof (struct in_addr)];  
  };  
  ```

struct in_addr  
{  
    in_addr_t s_addr;  
};  

struct sockaddr_in6  
{  
    sa_family_t sin6_family;  
    in_port_t sin6_port; /* Transport layer port # */  
    uint32_t sin6_flowinfo; /* IPv6 flow information */  
    struct in6_addr sin6_addr; /* IPv6 address */  
    uint32_t sin6_scope_id; /* IPv6 scope-id */  
};  

typedef unsigned short uint16_t;  
typedef unsigned int uint32_t;  
typedef uint16_t in_port_t;  
typedef uint32_t in_addr_t;  
#define __SOCKADDR_COMMON_SIZE (sizeof (unsigned short int)

```
```ad-tip
所有**专用 socket 地址（以及 sockaddr_storage）类型的变量在实际使用时都需要转化为通用 socket 地址类型 sockaddr（强制转化即可）**，因为所有 socket 编程接口使用的地址参数类型都是 sockaddr。
```

#### IP地址转换

通常，人们习惯用可读性好的字符串来表示 IP 地址，比如用点分十进制字符串表示 IPv4 地址，以及用十六进制字符串表示 IPv6 地址。但编程中我们需要先把它们转化为整数（二进制数）方能使用。而记录日志时则相反，我们要**把整数表示的 IP 地址转化为可读的字符串**。
下面 3 个函数可用于用点分十进制字符串表示的 IPv4 地址和用网络字节序整数表示的 IPv4 地址之间的转换(旧函数，不推荐使用)：

```c
#include <arpa/inet.h>  

//将包含 IPv4 dotted-decimal 地址的字符串转换为IN_ADDR结构的正确地址。
in_addr_t inet_addr(const char *cp); 

//将一个字符串表示的点分十进制IP地址IP转换为网络字节序存储在addr中，并且返回该网络字节序表示的无符号整数。
int inet_aton(const char *cp, struct in_addr *inp); 

// 将一个网络字节序的IP地址（也就是结构体in_addr类型变量）转化为点分十进制的IP地址（字符串）。
char *inet_ntoa(struct in_addr in);
```

下面这对新的函数也能完成前面 3 个函数同样的功能，并且它们同时适用 IPv4 地址和 IPv6 地址：

```c
#include <arpa/inet.h> 

// p:点分十进制的IP字符串，n:表示network，网络字节序的整数 
// 将点分十进制的IP地址字符串转换成网络字节序的整数
int inet_pton(int af, const char *src, void *dst);  
    af:地址族： AF_INET AF_INET6  
    src:需要转换的点分十进制的IP字符串  
    dst:转换后的结果保存在这个里面  

// 将网络字节序的整数，转换成点分十进制的IP地址字符串  
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);  
    af:地址族： AF_INET AF_INET6  
    src: 要转换的ip的整数的地址  
    dst: 转换成IP地址字符串保存的地方  
    size：第三个参数的大小（数组的大小）  
    返回值：返回转换后的数据的地址（字符串），和 dst 是一样的
```

### TCP通信流程

![Pasted image 20230410141631|400](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410141631_11-48-00.png)

#### TCP 通信的流程

- 服务器端 （被动接受连接的角色）  
  1. 创建一个用于监听的套接字  <font color="#92cddc">socket()</font>
     - 监听：监听有客户端的连接  
     - 套接字：这个套接字其实就是一个文件描述符  
  2. 将这个监听文件描述符和本地的IP和端口绑定（IP和端口就是服务器的地址信息） <font color="#92cddc"> bind()</font>
     - 客户端连接服务器的时候使用的就是这个IP和端口  
  3. 设置监听，监听的fd开始工作 —看是否有客户端发起连接 <font color="#92cddc">listen()</font>
  4. 阻塞等待，当有客户端发起连接，解除阻塞，接受客户端的连接，会得到一个和客户端通信的套接字（fd）  <font color="#92cddc">accept()</font>
  5. 通信  
     - 接收数据  <font color="#92cddc">recv()</font>
     - 发送数据  <font color="#92cddc">send()</font>
  6. 通信结束，断开连接 <font color="#92cddc">close()</font>
- 客户端  
  1. 创建一个用于通信的套接字（fd）  <font color="#92cddc">socket()</font>
  2. 连接服务器，需要指定连接的服务器的 IP 和 端口   <font color="#92cddc">connect()</font>
  3. 连接成功了，客户端可以直接和服务器通信  
     - 接收数据  <font color="#92cddc">send()</font>
     - 发送数据  <font color="#92cddc">recv()</font>
  4. 通信结束，断开连接  <font color="#92cddc">close()</font>

#### TCP滑动窗口

滑动窗口（Sliding window）是一种流量控制技术。早期的网络通信中，通信双方不会考虑网络的拥挤情况直接发送数据。由于大家不知道网络拥塞状况，同时发送数据，导致中间节点阻塞掉包，谁也发不了数据，所以就有了滑动窗口机制来解决此问题。滑动窗口协议是用来改善吞吐量的一种技术，即容许发送方在接收任何应答之前传送附加的包。接收方告诉发送方在某一时刻能送多少包（称窗口尺寸）。

- 窗口理解为缓冲区的大小  

- 滑动窗口的大小会随着发送数据和接收数据而变化。  

- 通信的双方都有发送缓冲区和接收数据的缓冲区  
  
  - 服务器：  
    - 发送缓冲区（发送缓冲区的窗口）  
    - 接收缓冲区（接收缓冲区的窗口）  
  - 客户端  
    - 发送缓冲区（发送缓冲区的窗口）  
    - 接收缓冲区（接收缓冲区的窗口）
      ![Pasted image 20230410150835|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410150835_11-48-21.png)

- 发送方的缓冲区： 
  
  - 白色格子：空闲的空间 
  - 灰色格子：数据已经被发送出去了，但是还没有被接收  
  - 紫色格子：还没有发送出去的数据  

- 接收方的缓冲区： 
  
  - 白色格子：空闲的空间 
  
  - 紫色格子：已经接收到的数据
    ![Pasted image 20230410151026|400](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410151026_11-48-44.png)
    
    ```ad-cite
    \# mss: Maximum Segment Size(一条数据的最大的数据量)  
    ```

\# win: 滑动窗口  

1. 客户端向服务器发起连接，客户单的滑动窗口是4096，一次发送的最大数据量是1460  

2. 服务器接收连接情况，告诉客户端服务器的窗口大小是6144，一次发送的最大数据量是1024  

3. 第三次握手  

4. 4-9 客户端连续给服务器发送了6k的数据，每次发送1k  

5. 第10次，服务器告诉客户端：发送的6k数据以及接收到，存储在缓冲区中，缓冲区数据已经处理了2k,窗口大小是2k  

6. 第11次，服务器告诉客户端：发送的6k数据以及接收到，存储在缓冲区中，缓冲区数据已经处理了4k,窗口大小是4k  

7. 第12次，客户端给服务器发送了1k的数据  

8. 第13次，客户端主动请求和服务器断开连接，并且给服务器发送了1k的数据  

9. 第14次，服务器回复ACK 8194, a:同意断开连接的请求 b:告诉客户端已经接受到方才发的2k的数据 c:滑动窗口2k  

10. 第15、16次，通知客户端滑动窗口的大小  

11. 第17次，第三次挥手，服务器端给客户端发送FIN,请求断开连接  

12. 第18次，第四次回收，客户端同意了服务器端的断开请求
    
    ```
    
    ```

### 套接字函数

- 头文件：
  1.#include <sys/types.h>  
  2.#include <sys/socket.h>  
  3.#include <arpa/inet.h> // 包含了这个头文件，上面两个就可以省略  

- int **socket**(int domain, int type, int protocol);  
  
  - 功能：创建一个套接字  
  - 参数：  
    - domain: 协议族  
        AF_INET : ipv4  
        AF_INET6 : ipv6  
        AF_UNIX, AF_LOCAL : 本地套接字通信（进程间通信）  
    - type: 通信过程中使用的协议类型  
        SOCK_STREAM : 流式协议  
        SOCK_DGRAM : 报式协议  
    - protocol : 具体的一个协议。一般写0  
      - SOCK_STREAM : 流式协议默认使用 TCP  
      - SOCK_DGRAM : 报式协议默认使用 UDP  
  - 返回值：  
    - 成功：返回文件描述符，操作的就是内核缓冲区。  
    - 失败：-1  

- int **bind**(int sockfd, const struct sockaddr \*addr, socklen_t addrlen); // socket命名  
  
  - 功能：绑定，将fd 和本地的IP + 端口进行绑定  
  - 参数：  
    - sockfd : 通过socket函数得到的文件描述符  
    - addr : 需要绑定的socket地址，这个地址封装了ip和端口号的信息  
    - addrlen : 第二个参数结构体占的内存大小 

- int **listen**(int sockfd, int backlog); // /proc/sys/net/core/somaxconn  
  
  - 功能：监听这个socket上的连接 ，创建两个队列（未连接和已连接）
  - 参数：  
    - sockfd : 通过socket()函数得到的文件描述符  
    - backlog : 设置未连接的和已经连接的数量和的最大值限制

- int **accept**(int sockfd, struct sockaddr \*addr, socklen_t \*addrlen);  
  
  - 功能：接收客户端连接，默认是一个阻塞的函数，阻塞等待客户端连接  
  - 参数：  
    - sockfd : 用于监听的文件描述符  
    - addr : 传出参数，记录了连接成功后客户端的地址信息（ip，port）  
    - addrlen : 指定第二个参数的对应的内存大小  
  - 返回值：  
    - 成功 ：用于通信的文件描述符  
    - -1 ： 失败  

- int **connect**(int sockfd, const struct sockaddr \*addr, socklen_t addrlen);
  
  - 功能： 客户端连接服务器  
  - 参数：  
    - sockfd : 用于通信的文件描述符  
    - addr : 客户端要连接的服务器的地址信息  
    - addrlen : 第二个参数的内存大小  
  - 返回值：成功 0， 失败 -1  

ssize_t **write**(int fd, const void \*buf, size_t count); // 写数据  
ssize_t **read**(int fd, void \*buf, size_t count); // 读数据

### 服务端和客户端socket通信

服务端：

```cpp
// TCP 通信的服务器端

#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>

int main() {

    // 1.创建socket(用于监听的套接字)
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if(lfd == -1) {
        perror("socket");
        exit(-1);
    }
    
    // 2.绑定
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    // inet_pton(AF_INET, "192.168.193.128", saddr.sin_addr.s_addr);
    saddr.sin_addr.s_addr = INADDR_ANY;  // 0.0.0.0  任意地址
    saddr.sin_port = htons(9999); //从主机字节序转换为网络字节序
    int ret = bind(lfd, (struct sockaddr *)&saddr, sizeof(saddr));

    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 3.监听
    ret = listen(lfd, 8);
    if(ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 4.接收客户端连接
    struct sockaddr_in clientaddr;
    int len = sizeof(clientaddr);
    int cfd = accept(lfd, (struct sockaddr *)&clientaddr, &len);
    if(cfd == -1) {
        perror("accept");
        exit(-1);
    }

    // 输出客户端的信息
    char clientIP[16];
    inet_ntop(AF_INET, &clientaddr.sin_addr.s_addr, clientIP, sizeof(clientIP));
    unsigned short clientPort = ntohs(clientaddr.sin_port);
    printf("client ip is %s, port is %d\n", clientIP, clientPort);

    // 5.通信
    char recvBuf[1024] = {0};
    while(1) {
        // 获取客户端的数据
        int num = read(cfd, recvBuf, sizeof(recvBuf));
        if(num == -1) {
            perror("read");
            exit(-1);
        } else if(num > 0) {
            printf("recv client data : %s\n", recvBuf);
        } else if(num == 0) {
            // 表示客户端断开连接
            printf("clinet closed...");
            break;
        }

        char * data = "hello,i am server";
        // 给客户端发送数据
        write(cfd, data, strlen(data));
    }

    // 关闭文件描述符
    close(cfd);
    close(lfd);
    return 0;

}
```

客户端：

```cpp
// TCP通信的客户端

#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>


int main() {

    // 1.创建套接字
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if(fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 2.连接服务器端
    struct sockaddr_in serveraddr;
    serveraddr.sin_family = AF_INET;
    inet_pton(AF_INET, "10.193.211.187", &serveraddr.sin_addr.s_addr);
    serveraddr.sin_port = htons(9999);
    int ret = connect(fd, (struct sockaddr *)&serveraddr, sizeof(serveraddr));

    if(ret == -1) {
        perror("connect");
        exit(-1);
    }

    // 3. 通信
    char recvBuf[1024] = {0};
    while(1) {
        char * data = "hello,i am client";
        // 给客户端发送数据
        write(fd, data , strlen(data));
        sleep(1);
        int len = read(fd, recvBuf, sizeof(recvBuf));
        if(len == -1) {
            perror("read");
            exit(-1);
        } else if(len > 0) {
            printf("recv server data : %s\n", recvBuf);
        } else if(len == 0) {
            // 表示服务器端断开连接
            printf("server closed...");
            break;
        }

    }

    // 关闭连接
    close(fd);
    return 0;

}
```

### TCP 通信并发

要实现TCP通信服务器处理并发的任务（即实现服务器的多客户端连接），需要使用多线程或者多进程来解决。  

思路：  

1. 一个父进程，多个子进程  
2. 父进程负责等待并接受客户端的连接  
3. 子进程：完成通信，接受一个客户端连接，就创建一个子进程用于通信

#### 利用进程实现

```cpp
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>
#include <wait.h>
#include <errno.h>

void recyleChild(int arg) {
    while(1) {
        int ret = waitpid(-1, NULL, WNOHANG);
        if(ret == -1) {
            // 所有的子进程都回收了
            break;
        }else if(ret == 0) {
            // 还有子进程活着
            break;
        } else if(ret > 0){
            // 被回收了
            printf("子进程 %d 被回收了\n", ret);
        }
    }
}

int main() {

    struct sigaction act;
    act.sa_flags = 0;
    sigemptyset(&act.sa_mask);
    act.sa_handler = recyleChild;
    // 注册信号捕捉
    sigaction(SIGCHLD, &act, NULL);

     // 创建socket
    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    if(lfd == -1){
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(9999);
    saddr.sin_addr.s_addr = INADDR_ANY;

    // 绑定
    int ret = bind(lfd,(struct sockaddr *)&saddr, sizeof(saddr));
    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 监听
    ret = listen(lfd, 128);
    if(ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 不断循环等待客户端连接
    while(1) {

        struct sockaddr_in cliaddr;
        int len = sizeof(cliaddr);
        // 接受连接
        int cfd = accept(lfd, (struct sockaddr*)&cliaddr, &len);
        if(cfd == -1) {
             if(errno == EINTR) {//防止客户端意外退出后无法重新连接
                 continue;
             }
            perror("accept");
            exit(-1);
        }

         // 每一个连接进来，创建一个子进程跟客户端通信
        pid_t pid = fork();
        if(pid == 0) {
            // 子进程
            // 获取客户端的信息
            char cliIp[16];
            inet_ntop(AF_INET, &cliaddr.sin_addr.s_addr, cliIp, sizeof(cliIp));
            unsigned short cliPort = ntohs(cliaddr.sin_port);
            printf("client ip is : %s, prot is %d\n", cliIp, cliPort);

            // 接收客户端发来的数据
            char recvBuf[1024];
            while(1) {
                int len = read(cfd, &recvBuf, sizeof(recvBuf));
                if(len == -1) {
                    perror("read");
                    exit(-1);
                }else if(len > 0) {
                    printf("recv client : %s\n", recvBuf);
                } else if(len == 0) {
                    printf("client closed....\n");
                    break;
                }
                write(cfd, recvBuf, strlen(recvBuf) + 1);//考虑换行符
            }
            close(cfd);
            exit(0);    // 退出当前子进程
        }
    }
    close(lfd);
    return 0;
}
```

#### 利用线程实现

```cpp
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>

struct sockInfo {
    int fd; // 通信的文件描述符
    struct sockaddr_in addr;
    pthread_t tid;  // 线程号
};

struct sockInfo sockinfos[128];

void * working(void * arg) {
    // 子线程和客户端通信   cfd 客户端的信息 线程号
    // 获取客户端的信息
    struct sockInfo * pinfo = (struct sockInfo *)arg;

    char cliIp[16];
    inet_ntop(AF_INET, &pinfo->addr.sin_addr.s_addr, cliIp, sizeof(cliIp));
    unsigned short cliPort = ntohs(pinfo->addr.sin_port);
    printf("client ip is : %s, prot is %d\n", cliIp, cliPort);

    // 接收客户端发来的数据
    char recvBuf[1024];
    while(1) {
        int len = read(pinfo->fd, &recvBuf, sizeof(recvBuf));

        if(len == -1) {
            perror("read");
            exit(-1);
        }else if(len > 0) {
            printf("recv client : %s\n", recvBuf);
        } else if(len == 0) {
            printf("client closed....\n");
            break;
        }
        write(pinfo->fd, recvBuf, strlen(recvBuf) + 1);
    }
    close(pinfo->fd);
    return NULL;
}

int main() {

    // 创建socket
    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    if(lfd == -1){
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;
    saddr.sin_port = htons(9999);
    saddr.sin_addr.s_addr = INADDR_ANY;

    // 绑定
    int ret = bind(lfd,(struct sockaddr *)&saddr, sizeof(saddr));
    if(ret == -1) {
        perror("bind");
        exit(-1);
    }

    // 监听
    ret = listen(lfd, 128);
    if(ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 初始化数据
    int max = sizeof(sockinfos) / sizeof(sockinfos[0]);
    for(int i = 0; i < max; i++) {
        bzero(&sockinfos[i], sizeof(sockinfos[i]));
        sockinfos[i].fd = -1;
        sockinfos[i].tid = -1;
    }

    // 循环等待客户端连接，一旦一个客户端连接进来，就创建一个子线程进行通信
    while(1) {

        struct sockaddr_in cliaddr;
        int len = sizeof(cliaddr);
        // 接受连接
        int cfd = accept(lfd, (struct sockaddr*)&cliaddr, &len);

        struct sockInfo * pinfo;
        for(int i = 0; i < max; i++) {
            // 从这个数组中找到一个可以用的sockInfo元素
            if(sockinfos[i].fd == -1) {
                pinfo = &sockinfos[i];
                break;
            }
            if(i == max - 1) {
                sleep(1);
                i--;
            }
        }

        pinfo->fd = cfd;
        memcpy(&pinfo->addr, &cliaddr, len);

        // 创建子线程
        pthread_create(&pinfo->tid, NULL, working, pinfo);

        pthread_detach(pinfo->tid);
    }

    close(lfd);
    return 0;
}
```

### TCP 状态转换

![[Pasted image 20230410162244.png|]]

![Pasted image 20230410162308](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230410162308_11-49-15.png)

- 2MSL（Maximum Segment Lifetime）  
    主动断开连接的一方, 最后进入一个 TIME_WAIT状态, 这个状态会持续: 2msl  
  - msl: 官方建议: 2分钟, 实际是30s  
    当 TCP 连接主动关闭方接收到被动关闭方发送的 FIN 和最终的 ACK 后，连接的主动关闭方必须处于TIME_WAIT 状态并持续 2MSL 时间。这样就能够让 TCP 连接的主动关闭方在它发送的 ACK 丢失的情况下重新发送最终ACK。主动关闭方重新发送的最终 ACK 并不是因为被动关闭方重传了 ACK（它们并不消耗序列号，被动关闭方也不会重传），而是因为被动关闭方重传了它的 FIN。事实上，被动关闭方总是重传 FIN 直到它收到一个最终的 ACK。  
- 半关闭  左边调用close右边没有
    当 TCP 链接中 A 向 B 发送 FIN 请求关闭，另一端 B 回应 ACK 之后（A 端进入FIN_WAIT_2状态），并没有立即发送 FIN 给 A，A 方处于半连接状态（半开关），此时 A **可以接收** B 发送的数据，但是 A 已经不能再向 B 发送数据。 

从程序的角度，可以使用 API 来控制实现半连接状态，控制关闭写和读

- int **shutdown**(int sockfd, int how);  
  
  - 头文件：#include <sys/socket.h>  
  - 参数
    - sockfd: 需要关闭的socket的描述符  
    - how: 允许为shutdown操作选择以下几种方式:  
      - SHUT_RD(0)： 关闭sockfd上的**读功能**，此选项将不允许sockfd进行读操作。该套接字不再接收数据，任何当前在套接字接受缓冲区的数据将被无声的丢弃掉。  
      - SHUT_WR(1): 关闭sockfd的**写功能**，此选项将不允许sockfd进行写操作。进程不能在对此套接字发出写操作。  
      - SHUT_RDWR(2):关闭sockfd的**读写功能**。相当于调用shutdown两次（相当于close）：首先是以SHUT_RD,然后以SHUT_WR

- 使用 close 中止一个连接，但它只是减少描述符的引用计数，并不直接关闭连接，只有当描述符的引用计数为 0 时才关闭连接。

- shutdown 不考虑描述符的引用计数，直接关闭描述符。也可选择中止一个方向的连接，只中止读或只中止写。  
  注意:  
1. 如果有多个进程共享一个套接字，close 每被调用一次，计数减 1 ，直到计数为 0 时，也就是所用进程都调用了 close，套接字将被释放。  
2. 在多进程中如果一个进程调用了 shutdown(sfd, SHUT_RDWR) 后，其它的进程将无法进行通信。但如果一个进程 close(sfd) 将不会影响到其它进程。

### 端口复用

端口复用最常用的用途是:  
    防止服务器重启时之前绑定的端口还未释放  
    程序突然退出而系统没有释放端口

// 设置套接字的属性（不仅仅能设置端口复用）  

- int **setsockopt**(int sockfd, int level, int optname, const void \*optval, socklen_t  optlen);  
  - 参数：  
    - sockfd : 要操作的文件描述符  
    - level : 级别 - SOL_SOCKET (端口复用的级别)  
    - optname : 选项的名称  
      - SO_REUSEADDR  
      - SO_REUSEPORT  
    - optval : 端口复用的值（整形）  
      - 1 : 可以复用  
      - 0 : 不可以复用  
    - optlen : optval参数的大小

> 端口复用，设置的时机是在服务器绑定端口之前。  
>     setsockopt();  
>     bind();

常看网络相关信息的命令:  

- netstat  
  - 参数：
      -a 所有的socket  
      -p 显示正在使用socket的程序的名称  
      -n 直接使用IP地址，而不通过域名服务器-a 所有的socket  
      -p 显示正在使用socket的程序的名称  
      -n 直接使用IP地址，而不通过域名服务器

## I/O多路复用

**I/O 多路复用使得程序能同时监听多个文件描述符，能够提高程序的性能**，Linux 下实现 I/O 多路复用的系统调用主要有 select、poll 和 epoll。

### 阻塞等待

- 单进程阻塞等待：
    好处：不占用CPU宝贵的时间片  
    缺点：同一时刻只能处理一个操作, 效率低  
- 多线程或者多进程解决
    缺点：  
  1. 线程或者进程会消耗资源  
  2. 线程或进程调度消耗CPU资源  
     ![Pasted image 20230411102314](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230411102314_11-49-29.png)

### 非阻塞, 忙轮询

- 优点: 提高了程序的执行效率  
- 缺点: 需要占用更多的CPU和系统资源  
    ![Pasted image 20230411102444](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230411102444_11-49-45.png)
  - 需要不断遍历判断数据是否到达
- 使用IO多路转接技术select/poll/epoll进行改进
  - select/poll
    - 委托内核完成检测数据是否到达
    - select只会告诉有多少文件描述符有数据到达，但是不会告诉是哪个文件描述符
  - epoll
    - 可以通知有数据的文件描述符个数以及有数据的文件描述符

### Select函数

- 主旨思想：  
  
  1. 首先要构造一个关于文件描述符的列表，将要监听的文件描述符添加到该列表中。  
  2. 调用一个系统函数，监听该列表中的文件描述符，直到这些描述符中的一个或者多个进行I/O操作时，该函数才返回。  
      a.这个函数是阻塞  
      b.函数对文件描述符的检测的操作是由内核完成的  
  3. 在返回时，它会告诉进程有多少（哪些）描述符要进行I/O操作

- select()工作过程分析
    客户端A,B,C,D连接到服务器分别对应文件描述符3,4,100,101 ;**A,B发送了数据**
    初始化：将连接到的文件描述符置为1然后拷贝到内核
    ![Pasted image 20230411105446](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230411105446_20-54-46.png)
    内核检测数据发生的文件描述符置为1，没有变化置为0；结束后再从内核态拷贝到用户态；**内核会全部遍历，然后判断值为1才会去检测这个文件描述符是否有数据**
    ![Pasted image 20230411103810](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230411103810_20-54-46.png)

- 头文件：#include <sys/time.h>  、#include <sys/types.h>  、#include <unistd.h>  、#include <sys/select.h>  

- int **select**(int nfds, fd_set \*readfds, fd_set \*writefds, fd_set \*exceptfds, struct timeval \*timeout);  
  
  - 参数：  
    
    - nfds : 委托内核检测的最大文件描述符的值 + 1  
    
    - readfds : 要检测的文件描述符的读的集合，委托内核检测哪些文件描述符的读的属性  
      
      - 一般检测读操作 
      - 对应的是对方发送过来的数据，因为读是被动的接收数据，检测的就是读缓冲  区  
      - 是一个传入传出参数  
      - sizeof(fd_set) = 128字节 1024位
    
    - writefds : 要检测的文件描述符的写的集合，委托内核检测哪些文件描述符的写的属性  
      
      - 委托内核检测写缓冲区是不是还可以写数据（不满的就可以写）  
    
    - exceptfds : 检测发生异常的文件描述符的集合  
    
    - timeout : 设置的超时时间  
        struct timeval {  
      
           long tv_sec;       /\* seconds \*/  
           long tv_usec;      /\* microseconds \*/  
      
        };  
      
      - NULL : 永久阻塞，直到检测到了文件描述符有变化  
      - tv_sec = 0 && tv_usec = 0， 不阻塞  
      - tv_sec > 0 && tv_usec > 0， 阻塞对应的时间  
  
  - 返回值 :  
    
    - -1 : 失败  
    - > 0(n) : 检测的集合中有n个文件描述符发生了变化  

- void **FD_CLR**(int fd, fd_set \*set);  
  
  - 功能：将参数文件描述符fd对应的标志位设置为0  

- int **FD_ISSET**(int fd, fd_set \*set);  
  
  - 功能：判断fd对应的标志位是0还是1
  - 返回值 ： fd对应的标志位的值，0，返回0， 1，返回1  

- void **FD_SET**(int fd, fd_set \*set);
  
  - 功能：将参数文件描述符fd 对应的标志位，设置为1

- void **FD_ZERO**(fd_set \*set);
  
  - 功能：fd_set一共有1024 bit, 全部初始化为0  

![Pasted image 20230411112225](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230411112225_11-50-01.png)

缺点：
![Pasted image 20230411112250](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230411112250_11-50-11.png)

### Poll函数

- int **poll**(struct pollfd \*fds, nfds_t nfds, int timeout);  
  - 头文件：#include <poll.h>
  - 参数：  
    - fds : 是一个struct pollfd 结构体数组，这是一个需要检测的文件描述符的集合  
    - nfds : 这个是第一个参数数组中最后一个有效元素的下标 + 1  
    - timeout : 阻塞时长  
        0 : 不阻塞  
        -1 : 阻塞，当检测到需要检测的文件描述符有变化，解除阻塞  
        \>0 : 阻塞的时长  
  - 返回值：  
      -1 : 失败  
      \>0（n） : 成功,n表示检测到集合中有n个文件描述符发生变化  
  - <font color="#92cddc">struct pollfd</font> {   //**fds可以重用，检测events，修改revents**（和select的区别）
      int fd;              /\* 委托内核检测的文件描述符 \*/  
      short events;    /\* 委托内核检测文件描述符的什么事件 \*/  
      short revents;   /\* 文件描述符实际发生的事件 \*/  
    };  
      struct pollfd myfd;  
      myfd.fd = 5;  
      myfd.events = POLLIN | POLLOUT;
    ![Pasted image 20230411112536](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230411112536_20-54-46.png)

### Epoll函数

- 创建一个新的epoll实例。在内核中创建了一个数据，这个数据中有两个比较重要的数据，一个是需要检测的文件描述符的信息（红黑树），还有一个是就绪列表，存放检测到数据发送改变的文件描述符信息（双向链表）。
    ![Pasted image 20230411143933](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230411143933_11-50-24.png)

- int **epoll_create**(int size);  
  
  - 功能：创建一个新的epoll实例，使用完需要关闭
  - 参数：  
      size : 目前没有意义了。随便写一个数，必须大于0  
  - 返回值：  
      -1 : 失败  
      \> 0 : 文件描述符，操作epoll实例的

- typedef union epoll_data {  //联合体
  
      void \*ptr;  
      int fd;    //保存文件描述符信息
      uint32_t u32;  
      uint64_t u64;  
  
    } <font color="#92cddc">epoll_data_t</font>;

- struct <font color="#92cddc">epoll_event</font> {  
  
      uint32_t events;      // 要检测的事件
      epoll_data_t data;   // 结构体，保存用户信息
  
  };  
  
  - 常见的Epoll检测事件：  
    - EPOLLIN  
    - EPOLLOUT  
    - EPOLLERR

- int **epoll_ctl**(int epfd, int op, int fd, struct epoll_event \*event);  
  
  - 功能：对epoll实例进行管理：添加文件描述符信息，删除信息，修改信息
  
  - 参数：  
    
    - epfd : epoll实例对应的文件描述符  
    
    - op : 要进行什么操作  
      
            EPOLL_CTL_ADD: 添加  
            EPOLL_CTL_MOD: 修改  
            EPOLL_CTL_DEL: 删除  
    
    - fd : 要检测的文件描述符  
    
    - event : 检测文件描述符什么事情

- int **epoll_wait**(int epfd, struct epoll_event \*events, int maxevents, int timeout);  
  
  - 功能：检测函数
  - 参数：  
    - epfd : epoll实例对应的文件描述符  
    - events : 传出参数，保存了<u>发送了变化</u>的文件描述符的信息  
    - maxevents : 第二个参数结构体数组的大小  
    - timeout : 阻塞时间  
      - 0 : 不阻塞  
      - -1 : 阻塞，直到检测到fd数据发生变化，解除阻塞  
      - > 0 : 阻塞的时长（毫秒）  
  - 返回值：  
    - 成功，返回发送变化的文件描述符的个数 > 0  
    - 失败 -1

#### Epoll 的工作模式

- **LT 模式 （水平触发）**  — 系统默认
  
  - 假设委托内核检测读事件 -> 检测fd的读缓冲区  
  - 读缓冲区有数据 - > epoll检测到了会给用户通知  
      a.用户不读数据，数据一直在缓冲区，epoll 会一直通知  
      b.用户只读了一部分数据，epoll会通知  
      c.缓冲区的数据读完了，不通知  
  - LT（level - triggered）是缺省的工作方式，并且同时支持 block 和 no-block socket。在这  种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操作。**如果你不作任何操作，内核还是会继续通知你的**。

- **ET 模式（边沿触发）**  
  
  - 假设委托内核检测读事件 -> 检测fd的读缓冲区  
  - 读缓冲区有数据 - > epoll检测到了会给用户通知  
      a.用户不读数据，数据一致在缓冲区中，epoll下次检测的时候就<u>不通知</u>了  
      b.用户只读了一部分数据，epoll不通知  
      c.缓冲区的数据读完了，不通知  
  - ET（edge - triggered）是高速工作方式，只支持 no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪,并且不会再为那个文件描述符发送更多的就绪通知，**直到你做了某些操作**导致那个文件描述符不再为就绪状态了。但是请注意，如果一直不对这个 fd 作 IO 操作（从而导致它再次变成未就绪），内核不会发送更多的通知（only once）。  
  - ET 模式在很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。epoll工作在 ET 模式的时候，必须使用**非阻塞套接口**，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死

#### EPOLLONESHOT事件

- 即使可以使用 ET 模式，一个socket 上的某个事件还是可能被触发多次。这在并发程序中就会引起一个问题。比如一个线程在读取完某个socket 上的数据后开始处理这些数据，而在数据的处理过程中该socket 上又有新数据可读（EPOLLIN 再次被触发），此时另外一个线程被唤醒来读取这些新的数据。于是就出现了两个线程同时操作一个 socket 的局面。一个socket连接在任一时刻都只被一个线程处理，可以使用 epoll 的 EPOLLONESHOT 事件实现。  
- 对于注册了 EPOLLONESHOT 事件的文件描述符，操作系统最多触发其上注册的一个可读、可写或者异常事件，且只触发一次，除非我们使用 epoll_ctl 函数重置该文件描述符上注册的 EPOLLONESHOT 事件。这样，当一个线程在处理某个 socket 时，其他线程是不可能有机会操作该 socket 的。但反过来思考，注册了 EPOLLONESHOT 事件的 socket 一旦被某个线程处理完毕， 该线程就应该立即重置这个socket 上的 EPOLLONESHOT 事件，以确保这个 socket 下一次可读时，其 EPOLLIN 事件能被触发，进而让其他工作线程有机会继续处理这个 socket。

## UDP

### UDP

![Pasted image 20230411155055|400](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230411155055_11-50-35.png)

头文件：#include <sys/types.h>  、#include <sys/socket.h>

- ssize_t **sendto**(int sockfd, const void \*buf, size_t len, int flags, const struct sockaddr \*dest_addr, socklen_t addrlen);  
  - 参数：  
    - sockfd : 通信的fd  
    - buf : 要发送的数据  
    - len : 发送数据的长度  
    - flags : 0  
    - dest_addr : 通信的另外一端的地址信息  
    - addrlen : 地址的内存大小    
- ssize_t **recvfrom**(int sockfd, void \*buf, size_t len, int flags, struct sockaddr \*src_addr, socklen_t \*addrlen);  
  - 参数：  
    - sockfd : 通信的fd  
    - buf : 接收数据的数组  
    - len : 数组的大小
    - flags : 0  
    - src_addr : 用来保存另外一端的地址信息，不需要可以指定为NULL  
    - addrlen : 地址的内存大小

### 广播

向子网中多台计算机发送消息，并且子网中所有的计算机都可以接收到发送方发送的消息，每个广播消息都包含一个特殊的IP地址，这个IP中子网内主机标志部分的二进制全部为1。  
    a.只能在局域网中使用。  
    b.客户端需要绑定服务器广播使用的端口，才可以接收到广播消息。

- int setsockopt(int sockfd, int level, int optname,const void \*optval, socklen_t  optlen);  
  - 功能：设置广播属性的函数
  - 参数：
    - sockfd : 文件描述符  
    - level : SOL_SOCKET  
    - optname : **SO_BROADCAST**  
    - optval : int类型的值，为1表示允许广播  
    - optlen : optval的大小

### 组播(多播)

单播地址标识单个 IP 接口，广播地址标识某个子网的所有 IP 接口，多播地址标识一组 IP 接口。单播和广播是寻址方案的两个极端（要么单个要么全部），多播则意在两者之间提供一种折中方案。**多播数据报只应该由对它感兴趣的接口接收，也就是说由运行相应多播会话应用系统的主机上的接口接收**。另外，广播一般局限于局域网内使用，而多播则既可以用于局域网，也可以跨广域网使用。  
    a.组播既可以用于局域网，也可以用于广域网  
    b.客户端需要加入多播组，才能接收到多播的数据

- 组播地址  
  IP 多播通信必须依赖于 IP 多播地址，在 IPv4 中它的范围从 224.0.0.0 到 239.255.255.255 ，  
  并被划分为局部链接多播地址、预留多播地址和管理权限多播地址三类:
  
  | IP地址                      | 说明                                                  |
  | ------------------------- | --------------------------------------------------- |
  | 224.0.0.0~224.0.0.255     | 局部链接多播地址：是为路由协议和其它用途保留的地址，路由器并不转发属于此范围的IP包          |
  | 224.0.1.0~224.0.1.255     | 预留多播地址：公用组播地址，可用于Internet；使用前需要申请                   |
  | 224.0.2.0~238.255.255.255 | 预留多播地址：用户可用组播地址(临时组地址)，全网范围内有效                      |
  | 239.0.0.0~239.255.255.255 | 本地管理组播地址，可供组织内部使用，类似于私有 IP 地址，不能用于 Internet，可限制多播范围 |

- 设置组播

- int setsockopt(int sockfd, int level, int optname,const void \*optval, socklen_t optlen);  
  
  - 参数：
      // 服务器设置多播的信息，外出接口
    - level : IPPROTO_IP  
    - optname : **IP_MULTICAST_IF**  
    - optval : struct in_addr 
      // 客户端加入到多播组：  
    - level : IPPROTO_IP  
    - optname : **IP_ADD_MEMBERSHIP**  
    - optval : struct ip_mreq

- struct ip_mreq  
    {  
    struct in_addr imr_multiaddr; // 组播的IP地址  
    struct in_addr imr_interface; // 本地的IP地址  
    };  

- typedef uint32_t in_addr_t;  

- struct in_addr  
    {  
  
      in_addr_t s_addr;  
  
    };

## 本地套接字通信

本地套接字的作用：本地的进程间通信  
    有关系的进程间的通信  
    没有关系的进程间的通信  
本地套接字实现流程和网络套接字类似，一般采用TCP的通信流程

```cpp
// 本地套接字通信的流程 - tcp  
// 服务器端  
1.创建监听的套接字  
    int lfd = socket(AF_UNIX/AF_LOCAL, SOCK_STREAM, 0);  
2.监听的套接字绑定本地的套接字文件 -> server端  
    struct sockaddr_un addr;  
    // 绑定成功之后，指定的sun_path中的套接字文件会自动生成。  
    bind(lfd, addr, len);  
3.监听  
    listen(lfd, 100);  
4.等待并接受连接请求  
    struct sockaddr_un cliaddr;  
    int cfd = accept(lfd, &cliaddr, len);  
5.通信  
    接收数据：read/recv  
    发送数据：write/send  
6.关闭连接  
    close(); 

// 客户端的流程  
1.创建通信的套接字  
    int fd = socket(AF_UNIX/AF_LOCAL, SOCK_STREAM, 0);  
2.监听的套接字绑定本地的IP 端口  
    struct sockaddr_un addr;  
    // 绑定成功之后，指定的sun_path中的套接字文件会自动生成。  
    bind(lfd, addr, len);  
3.连接服务器  
    struct sockaddr_un serveraddr;  
    connect(fd, &serveraddr, sizeof(serveraddr));  
4.通信  
    接收数据：read/recv  
    发送数据：write/send  
5.关闭连接  
    close();

// 头文件: sys/un.h  
#define UNIX_PATH_MAX 108  
struct sockaddr_un {  
sa_family_t sun_family; // 地址族协议 af_local  
char sun_path[UNIX_PATH_MAX]; // 套接字文件的路径, 这是一个伪文件, 大小永远=0  
};
```

## Unix/Linux上的五种IO模型

### 阻塞/非阻塞  同步/异步

- 数据就绪：根据系统IO操作的就绪状态  
  - 阻塞  
  - 非阻塞  
- 数据读写：根据应用程序和内核的交互方式 
  - 同步
  - 异步

![阻塞、非阻塞、同步、异步](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/%E9%98%BB%E5%A1%9E%E3%80%81%E9%9D%9E%E9%98%BB%E5%A1%9E%E3%80%81%E5%90%8C%E6%AD%A5%E3%80%81%E5%BC%82%E6%AD%A5_20-54-46.png)

- 一个典型的网络IO接口调用，分为两个阶段，分别是“数据就绪” 和 “数据读写”，数据就绪阶段分为阻塞和非阻塞，表现得结果就是，阻塞当前线程或是直接返回。  
- **同步**表示A向B请求调用一个网络IO接口时（或者调用某个业务逻辑API接口时），数据的读写都是**由请求方A自己来完成的**（不管是阻塞还是非阻塞）；**异步**表示A向B请求调用一个网络IO接口时 （或者调用某个业务逻辑API接口时），向B传入请求的事件以及事件发生时通知的方式，A就可以处理其它逻辑了，**当B监听到事件处理完成后，会用事先约定好的通知方式，通知A处理结果**。
- I/O多路复用是同步还是异步？
  - 同步
- 在处理 IO 的时候，阻塞和非阻塞都是同步 IO，只有使用了特殊的 API 才是异步 IO，异步一般和非阻塞结合使用
    ![Pasted image 20230412094411|400](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412094411_11-50-54.png)

### 阻塞 blocking

- 调用者调用了某个函数，等待这个函数返回，期间什么也不做，不停的去检查这个函数有没有返回，必须等这个函数返回才能进行下一步动作。

- 阻塞非阻塞为文件描述符的属性
    ![Pasted image 20230412100436|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412100436_11-51-30.png)
  
  ### 非阻塞 non-blocking（NIO）

- 非阻塞等待，每隔一段时间就去检测IO事件是否就绪。没有就绪就可以做其他事。

- 非阻塞I/O执行系统调用总是立即返回，不管事件是否已经发生，若事件没有发生，则返回-1，此时可以根据 errno 区分这两种情况，对于accept，recv 和 send，事件未发生时，errno 通常被设置成 EAGAIN，表示这次调用还没有事件发生。
    ![Pasted image 20230412100611|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412100611_11-51-42.png)
  
  ### IO复用（IO multiplexing）

- Linux 用 select/poll/epoll 函数实现 IO 复用模型，这些函数也会使进程阻塞，但是和阻塞IO所不同的是这些函数可以同时阻塞多个IO操作。而且可以同时对多个读操作、写操作的IO函数进行检测。直到有数据可读或可写时，才真正调用IO操作函数。
    ![Pasted image 20230412100716|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412100716_11-52-04.png)
  
  ### 信号驱动（signal-driven）

- Linux 用套接口进行信号驱动 IO，安装一个信号处理函数，进程继续运行并不阻塞，当IO事件就绪，进程收到SIGIO 信号，然后处理 IO 事件。
    ![Pasted image 20230412100805|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412100805_11-52-22.png)

- 内核在第一个阶段是异步，在第二个阶段是同步；与非阻塞IO的区别在于它提供了消息通知机制，不需要用户进程不断的轮询检查，减少了系统API的调用次数，提高了效率

### 异步（asynchronous）

- Linux中，可以调用 aio_read 函数告诉内核描述字缓冲区指针和缓冲区的大小、文件偏移及通知的方式，然后立即返回，当内核将数据拷贝到缓冲区后，再通知应用程序。
    ![Pasted image 20230412100902|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412100902_11-52-43.png)

```cpp
/* Asynchronous I/O control block. */  
struct aiocb  
{  
    int aio_fildes; /* File desriptor. */  //文件描述符
    int aio_lio_opcode; /* Operation to be performed. */  
    int aio_reqprio; /* Request priority offset. */  
    volatile void *aio_buf; /* Location of buffer. */  
    size_t aio_nbytes; /* Length of transfer. */  
    struct sigevent aio_sigevent; /* Signal number and value. */  //信号机制

    /* Internal members. */  
    struct aiocb *__next_prio;  
    int __abs_prio;  
    int __policy;  
    int __error_code;  
    __ssize_t __return_value;  

    #ifndef __USE_FILE_OFFSET64  
    __off_t aio_offset; /* File offset. */  
    char __pad[sizeof (__off64_t) - sizeof (__off_t)];  
    #else  
    __off64_t aio_offset; /* File offset. */  
    #endif  
    char __glibc_reserved[32];  
};
```

## Web Server（网页服务器）

- 一个 Web Server 就是一个服务器软件（程序），或者是运行这个服务器软件的硬件（计算机）。其主要功能是通过 HTTP 协议与客户端（通常是浏览器（Browser））进行通信，来接收，存储，处理来自客户端的 HTTP 请求，并对其请求做出 HTTP 响应，返回给客户端其请求的内容（文件、网页等）或返回一个 Error 信息。
    ![Pasted image 20230412101944|600](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412101944_11-53-02.png)
- 通常用户使用 Web 浏览器与相应服务器进行通信。在浏览器中键入“域名”或“IP地址:端口号”，浏览器则先将你的域名解析成相应的 IP 地址或者直接根据你的IP地址向对应的 Web 服务器发送一个 HTTP 请求。这一过程首先要通过 TCP 协议的三次握手建立与目标 Web 服务器的连接，然后 HTTP 协议生成针对目标 Web 服务器的 HTTP 请求报文，通过 TCP、IP 等协议发送到目标 Web 服务器上。

### HTTP协议(应用层的协议)

- HTTP 是一个客户端终端（用户）和服务器端（网站）请求和应答的标准（TCP）。通过使用网页浏览器、网络爬虫或者其它的工具，客户端发起一个HTTP请求到服务器上指定端口（默认端口为80）。我们称这个客户端为用户代理程序（user agent）。应答的服务器上存储着一些资源，比如 HTML 文件和图像。我们称这个应答服务器为源服务器（origin server）。在用户代理和源服务器中间可能存在多个“中间层”，比如代理服务器、网关或者隧道（tunnel）。
- HTTP 协议定义 Web 客户端如何从 Web 服务器请求 Web 页面，以及服务器如何把 Web 页面传送给客户端。HTTP 协议采用了请求/响应模型。客户端向服务器发送一个请求报文，请求报文包含请求的方法、URL、协议版本、请求头部和请求数据。服务器以一个状态行作为响应，响应的内容包括协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。

#### HTTP 请求报文格式

![Pasted image 20230412102802|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412102802_11-53-19.png)

#### HTTP响应报文格式

![Pasted image 20230412102830|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412102830_11-53-44.png)

#### HTTP请求方法

HTTP/1.1 协议中共定义了八种方法（也叫“动作”）来以不同方式操作指定的资源：

1. **GET**：向指定的资源发出“显示”请求。使用 GET 方法应该只用在读取数据，而不应当被用于产生“副作用”的操作中，例如在 Web Application 中。其中一个原因是 GET 可能会被网络蜘蛛等随意访问。  
2. HEAD：与 GET 方法一样，都是向服务器发出指定资源的请求。只不过服务器将不传回资源的本文部分。它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中“关于该资源的信息”（元信息或称元数据）。  
3. **POST**：向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求本文中。这个请求可能会创建新的资源或修改现有资源，或二者皆有。  
4. PUT：向指定资源位置上传其最新内容。  
5. DELETE：请求服务器删除 Request-URI 所标识的资源。  
6. TRACE：回显服务器收到的请求，主要用于测试或诊断。  
7. OPTIONS：这个方法可使服务器传回该资源所支持的所有 HTTP 请求方法。用'\*'来代替资源名称，向 Web 服务器发送 OPTIONS 请求，可以测试服务器功能是否正常运作。  
8. CONNECT：HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。通常用于SSL加密服务器的链接（经由非加密的 HTTP 代理服务器）。

#### HTTP状态码

状态代码的第一个数字代表当前响应的类型：  

- 1xx消息——请求已被服务器接收，继续处理  
- 2xx成功——请求已成功被服务器接收、理解、并接受  
- 3xx重定向——需要后续操作才能完成这一请求  
- 4xx请求错误——请求含有词法错误或者无法被执行  
- 5xx服务器错误——服务器在处理某个正确请求时发生错误
    ![Pasted image 20230412103328](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412103328_11-54-03.png)

### 服务器编程基本框架

![Pasted image 20230412103449|500](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412103449_11-54-11.png)

| 模块       | 功能            |
| -------- | ------------- |
| I/O 处理单元 | 处理客户连接，读写网络数据 |
| 逻辑单元     | 业务进程或线程       |
| 网络存储单元   | 数据库、文件或缓存     |
| 请求队列     | 各单元之间的通信方式    |

- **I/O 处理单元**是服务器管理客户连接的模块。它通常要完成以下工作：等待并接受新的客户连接，接收客户数据，将服务器响应数据返回给客户端。但是数据的收发不一定在 I/O 处理单元中执行，也可能在逻辑单元中执行，具体在何处执行取决于事件处理模式。  
- 一个**逻辑单元**通常是一个进程或线程。它分析并处理客户数据，然后将结果传递给 I/O 处理单元或者直接发送给客户端（具体使用哪种方式取决于事件处理模式）。服务器通常拥有多个逻辑单元，以实现对多个客户任务的并发处理。  
- **网络存储单元**可以是数据库、缓存和文件，但不是必须的。  
- **请求队列**是各单元之间的通信方式的抽象。I/O 处理单元接收到客户请求时，需要以某种方式通知一个逻辑单元来处理该请求。同样，多个逻辑单元同时访问一个存储单元时，也需要采用某种机制来协调处理竞态条件。请求队列通常被实现为池的一部分。

### 事件处理模式

服务器程序通常需要处理三类事件：I/O 事件、信号及定时事件。有两种高效的事件处理模式：Reactor和 Proactor，**同步 I/O 模型通常用于实现 Reactor 模式，异步 I/O 模型通常用于实现 Proactor 模式**。

#### Reactor模式

- 要求主线程（I/O处理单元）**只负责监听文件描述符上是否有事件发生**，有的话就立即将该事件通知工作线程（逻辑单元），将 socket 可读可写事件放入请求队列，交给工作线程处理。除此之外，主线程不做任何其他实质性的工作。读写数据，接受新的连接，以及处理客户请求均在工作线程中完成。

- 使用同步 I/O（以 epoll_wait 为例）实现的 Reactor 模式的工作流程是：  
1. 主线程往 epoll 内核事件表中注册 socket 上的读就绪事件。  
2. 主线程调用 epoll_wait 等待 socket 上有数据可读。  
3. 当 socket 上有数据可读时， epoll_wait 通知主线程。主线程则将 socket 可读事件放入请求队列。
4. 睡眠在请求队列上的某个工作线程被唤醒，它从 socket 读取数据，并处理客户请求，然后往 epoll内核事件表中注册该 socket 上的写就绪事件。  
5. 当主线程调用 epoll_wait 等待 socket 可写。  
6. 当 socket 可写时，epoll_wait 通知主线程。主线程将 socket 可写事件放入请求队列。  
7. 睡眠在请求队列上的某个工作线程被唤醒，它往 socket 上写入服务器处理客户请求的结果。
    ![Pasted image 20230412104219](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230412104219_20-54-46.png)

#### Proactor模式

Proactor 模式将**所有 I/O 操作都交给主线程和内核来处理**（进行读、写），工作线程仅仅负责业务逻辑。使用异步 I/O 模型（以 aio_read 和 aio_write 为例）实现的 Proactor 模式的工作流程是：  

1. 主线程调用 aio_read 函数向内核注册 socket 上的读完成事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序（这里以信号为例）。  
2. 主线程继续处理其他逻辑。  
3. 当 socket 上的数据被读入用户缓冲区后，内核将向应用程序发送一个信号，以通知应用程序数据已经可用。  
4. 应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求后，调用 aio_write 函数向内核注册 socket 上的写完成事件，并告诉内核用户写缓冲区的位置，以及写操作完成时如何通知应用程序。  
5. 主线程继续处理其他逻辑。  
6. 当用户缓冲区的数据被写入 socket 之后，内核将向应用程序发送一个信号，以通知应用程序数据已经发送完毕。  
7. 应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理，比如决定是否关闭 socket。
    ![Pasted image 20230412104311](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230412104311_20-54-46.png)
- 读写操作均在主线程，工作线程没有参与

#### 模拟 Proactor 模式——项目使用

使用同步 I/O 方式模拟出 Proactor 模式。原理是：主线程执行数据读写操作，读写完成之后，主线程向工作线程通知这一“完成事件”。那么从工作线程的角度来看，它们就直接获得了数据读写的结果，接下来要做的只是对读写的结果进行逻辑处理。  

使用同步 I/O 模型（以 epoll_wait为例）模拟出的 Proactor 模式的工作流程如下：  

1. 主线程往 epoll 内核事件表中注册 socket 上的读就绪事件。  
2. 主线程调用 epoll_wait 等待 socket 上有数据可读。  
3. 当 socket 上有数据可读时，epoll_wait 通知主线程。主线程从 socket 循环读取数据，直到没有更多数据可读，然后将读取到的数据封装成一个**请求对象**并插入请求队列。  
4. 睡眠在请求队列上的某个工作线程被唤醒，它获得请求对象并处理客户请求，然后往 epoll 内核事件表中注册 socket 上的写就绪事件。  
5. 主线程调用 epoll_wait 等待 socket 可写。  
6. 当 socket 可写时，epoll_wait 通知主线程。主线程往 socket 上写入服务器处理客户请求的结果。
    ![Pasted image 20230412104359](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/15/Pasted%20image%2020230412104359_20-54-46.png)

### 线程池

线程池是由服务器预先创建的一组子线程，线程池中的线程数量应该和 CPU 数量差不多。线程池中的所有子线程都运行着相同的代码。当有新的任务到来时，主线程将通过某种方式选择线程池中的某一个子线程来为之服务。相比与动态的创建子线程，选择一个已经存在的子线程的代价显然要小得多。至于主线程选择哪个子线程来为新任务服务，则有多种方式：  

- 主线程使用某种算法来主动选择子线程。最简单、最常用的算法是**随机算法**和 **Round Robin（轮流选取）** 算法，但更优秀、更智能的算法将使任务在各个工作线程中更均匀地分配，从而减轻服务器的整体压力。  
- 主线程和所有子线程通过一个**共享的工作队列**来同步，子线程都睡眠在该工作队列上。当有新的任务到来时，主线程将任务添加到工作队列中。这将唤醒正在等待任务的子线程，不过只有一个子线程将获得新任务的”接管权“，它可以从工作队列中取出任务并执行之，而其他子线程将继续睡眠在工作队列上。
    ![Pasted image 20230412105547](https://cdn.jsdelivr.net/gh/Huyang-yang/obs-picbed/img/2023/04/16/Pasted%20image%2020230412105547_11-54-36.png)

> 线程池中的线程数量最直接的限制因素是中央处理器(CPU)的处理器(processors/cores)的数量N ：如果你的CPU是4-cores的，对于CPU密集型的任务(如视频剪辑等消耗CPU计算资源的任务)来说，那线程池中的线程数量最好也设置为4（或者+1防止其他因素造成的线程阻塞）；对于IO密集型的任务，一般要多于CPU的核数，因为线程间竞争的不是CPU的计算资源而是IO，IO的处理一般较慢，多于cores数的线程将为CPU争取更多的任务，不至在线程处理IO的过程造成CPU空闲导致资源浪费。

- 线程池总结：
  - 空间换时间，浪费服务器的硬件资源，换取运行效率。  
  - 池是一组资源的集合，这组资源在服务器启动之初就被完全创建好并初始化，这称为静态资源。
  - 当服务器进入正式运行阶段，开始处理客户请求的时候，如果它需要相关的资源，可以直接从池中获取，无需动态分配。  
  - 当服务器处理完一个客户连接后，可以把相关的资源放回池中，无需执行系统调用释放资源。
- 改进
  - 动态地改进线程池的数量

### 有限状态机

- 逻辑单元内部的一种高效编程方法：有限状态机（finite state machine）。  

- 有的应用层协议头部包含数据包类型字段，每种类型可以映射为逻辑单元的一种执行状态，服务器可以根据它来编写相应的处理逻辑。如下是一种状态独立的有限状态机：
  
  ```cpp
  STATE_MACHINE( Package _pack )  
  {  
    PackageType _type = _pack.GetType();  
    switch( _type )  
    {  
        case type_A:  
        process_package_A( _pack );  
        break;  
        case type_B:  
        process_package_B( _pack );  
        break;  
    }  
  }
  ```
  
  这是一个简单的有限状态机，只不过该状态机的每个状态都是**相互独立**的，即状态之间没有相互转移。**状态之间的转移是需要状态机内部驱动**，如下代码
  
  ```cpp
  STATE_MACHINE()  
  {  
    State cur_State = type_A;
    while( cur_State != type_C )  
    {  
        Package _pack = getNewPackage();  
        switch( cur_State )  
        {  
        case type_A:  
            process_package_state_A( _pack );  
            cur_State = type_B;  
            break;  
        case type_B:  
            process_package_state_B( _pack );  
            cur_State = type_C;  
            break;  
        }  
    }  
  }
  ```

- 该状态机包含三种状态：type_A、type_B 和 type_C，其中 type_A 是状态机的开始状态，type_C 是状态机的结束状态。状态机的当前状态记录在 cur_State 变量中。在一趟循环过程中，状态机先通过getNewPackage 方法获得一个新的数据包，然后根据 cur_State 变量的值判断如何处理该数据包。数据包处理完之后，状态机通过给 cur_State 变量传递目标状态值来实现状态转移。那么当状态机进入下一趟循环时，它将执行新的状态对应的逻辑。

## Web Server（Code)

- 分析事件类型对应代码：
  ![[9D24E832B4375C8BBB6F86328270169C.png|600]]

## 可改进部分

### 触发模式

文件：http_conn.cpp
函数：void addfd(int epollfd, int fd, bool one_shot)
可改进：

```cpp
    epoll_event event;
    event.data.fd = fd;
    event.events = EPOLLIN | EPOLLRDHUP;   //检测EPOLLIN，默认为水平触发模式
```

```ad-tip
这里设置默认触发模式为水平触发模式，可以改进为可配置参数；支持水平触发和边沿触发
```

### 通知客户端

![[Pasted image 20230413160207.png|600]]

这里可以给客户端回传一个信息，告诉客户端服务器正忙（响应报文）

# 面经