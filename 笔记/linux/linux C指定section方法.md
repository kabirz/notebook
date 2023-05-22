获取user端链接脚本方法(out.lds)
  ```Shell
  ld --verbose >out.lds
  ```
  
源码（section.c)：有section
```c
#include <stdio.h>

#define __maybe_unused	__attribute__((__unused__))
#define __section(S)	__attribute__((__section__(S)))

int a __maybe_unused __section(".ss.b") = 100;
int b __maybe_unused __section(".ss.b") = 200;


int main(int argc, char **argv)
{
	static int tmp __maybe_unused __section(".ss.b") = 300;
	static int start[0] __maybe_unused __section(".ss.a");
	static int end[0] __maybe_unused __section(".ss.c");

	for(int *s=start; s<end; s++)
		printf("item: %p, %d\n", s, s[0]);

	printf("start: %p, %d\n", start, start[0]);
	printf("end: %p\n", end);

	return 0;
}
```

out.lds上在.data里面加上seciton ss

  ```Shell
  148   .data           :
  149   {
  150     *(.data .data.* .gnu.linkonce.d.*)
  151     *(SORT(.ss.*)) # 加入这一行
  152     SORT(CONSTRUCTORS)
  153   }
  ```
  

**编译** (gcc section.c -o section -Wl,-T out.lds)和**执行**

  ```Shell
❯  ./section
item: 0x55b4857ff010, 100
item: 0x55b4857ff014, 200
item: 0x55b4857ff018, 300
start: 0x55b4857ff010, 100
end: 0x55b4857ff01c
```



