
```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <execinfo.h>

void func0(void)
{
        printf("This is func0\n");
        int *p = NULL;
        *p = 1234;
}

void func1(void)
{
        printf("This is func1\n");
        func0();
}

void func2(void)
{
        printf("This is func2\n");
        func1();
}

void dump(int signo)
{
        void *array[100];
        size_t size;
        char **strings;

        size = backtrace(array, 100);
        strings = backtrace_symbols(array, size);

        printf("Obtained %zd stacks.\n", size);
        for (int i = 0; i < size; ++i) {
                printf("%s\n", strings[i]);
        }

        free(strings);
        exit(0);
}

int main(int argc, char *argv[])
{
        printf("====segmentation fault test4============\n");

        signal(SIGSEGV, dump);

        func2();
        return 0;
}
```

编译需要指定
```shell
gcc a.c -o a -rdynamic
```
有关于dynamic的
>   gcc选项-g与-rdynamic的异同
[gcc](http://lenky.info/tag/gcc/ "查看 gcc 中的全部文章") 的 [-g](http://lenky.info/tag/g/ "查看 -g 中的全部文章") ，应该没有人不知道它是一个调试选项，因此在一般需要进行程序调试的场景下，我们都会加上该选项，并且根据调试工具的不同，还能直接选择更有针对性的说明，比如 [-ggdb](http://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Debugging-Options.html) 。-g是一个编译选项，即在源代码编译的过程中起作用，让gcc把更多调试信息（也就包括符号信息）收集起来并将存放到最终的可执行文件内。   
相比-g选项， [-rdynamic](http://lenky.info/tag/rdynamic/ "查看 -rdynamic 中的全部文章") 却是一个 [连接选项](http://gcc.gnu.org/onlinedocs/gcc/Link-Options.html) ，它将指示连接器把所有符号（而不仅仅只是程序已使用到的外部符号）都添加到动态符号表（即.dynsym表）里，以便那些通过 [dlopen()](http://man7.org/linux/man-pages/man3/dlopen.3.html) 或 [backtrace()](http://www.kernel.org/doc/man-pages/online/pages/man3/backtrace.3.html) （这一系列函数使用.dynsym表内符号）这样的函数使用。

执行后可查看栈信息
```shell
====segmentation fault test4============                                                                                                                                                      This is func2                                                                                                                                                                                 This is func1                                                                                                                                                                                 This is func0                                                                                                                                                                                 Obtained 8 stacks.                                                                                                                                                                            ./a(dump+0x38) [0x563efc98a2a6]                                                                                                                                                               /lib/x86_64-linux-gnu/libc.so.6(+0x43090) [0x7fe213d00090]                                                                                                                                    ./a(func0+0x24) [0x563efc98a22d]                                                                                                                                                              ./a(func1+0x19) [0x563efc98a24f]                                                                                                                                                              ./a(func2+0x19) [0x563efc98a26b]                                                                                                                                                              ./a(main+0x35) [0x563efc98a382]                                                                                                                                                               /lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf3) [0x7fe213ce1083]                                                                                                                      ./a(_start+0x2e) [0x563efc98a14e]
```

