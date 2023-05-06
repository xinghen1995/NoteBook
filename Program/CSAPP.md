## 七、链接
### 7.1 编译器驱动程序
GNU编译工具链
1. cpp预处理器 `cpp [other arguments] main.c /tmp/main.i`
2. ccl编译器   `ccl /tmp/main.i -Og [other arguments] -o /tmp/main.s`
3. as汇编器    `as [other arguments] -o /tmp/main.o /tmp/main.s`
4. ld链接器    `ld -o prog [system object file and args] /tmp/main.o /tmp/main.s`
5. loader加载器`sh> ./prog`
#### 7.6.2 与静态库链接
1. `gcc -static -o prog2c main2.o -L. -lvector`编译报错(mac)找不到-lcrt0.o，修改<b>-static</b>为<b>-Bstatic</b>
#### 7.6.3 链接器如何使用静态库来解析引用
- 
- 静态库是多个目标文件(.o)的打包，在链接时，只会链接用到的部分，多余部分会被丢弃
- 这导致了一个问题，如果b.a依赖a.a，但在链接时a.a先被解析，b.a依赖的函数可能会被丢弃。所以一般把静态库放在最后链接，并且保持静态库之间不相互依赖
### 7.9 加载执行目标文件
- 程序的入口点是_start函数，定义在系统目标文件ctrl.o中。它调用系统启动函数_libc_start_main，该函数在libc.so，它初始化程序环境，并调用程序的main函数，处理main函数的返回值
- 运行时的内存映像(从高位地址到低位地址)
	+ 内核区域(2^48 - 1)
	+ 运行时栈帧
	+ 共享库的内存映射区域
	+ 运行时堆内存
	+ 读/写数据段
	+ 只读代码段(0x400000)
### 7.10 动态链接共享库
- `gcc -shared -fpic -o libvector.so addvec.c multvec.c`
- -shared告诉链接器创建一个共享的目标文件，-fpic指示编译器创建位置无关的代码
- `gcc -o prog21 main2.c ./libvector.so
### 7.11 从应用程序中加载和链接共享库
- Linux下动态链接器接口
	- 加载动态库接口
	```c
	#include <dlfcn.h>

	void* dlopen(const char* filename, int flag);
	```
	- 从加载的动态库中获取指定函数
	```c
	#include <dlfcn.h>

	void* dlsym(void* handle, char* symbol);
	```
	- 关闭动态库，如果没有其他程序使用，则卸载
	```c
    #include <dlfcn.h>
    
    void dlclose(void* handle);
	```
    - 查看最近一次dlopen、dlsym、dlclose返回的错误
    ```c
    #include <dlfcn.h>

    const char* dlerror(void);
    ```
- 编译参数 `gcc -rdynamic -o prog2r dll.c -ldl`
#### 7.12 位置无关代码
- 对于本模块内的函数，可以通过PC相对寻址。但动态库通常时跨模块的
- 位置无关的代码，即运行时加载共享库代码，而不用修改代码段(本质上是运行时修改了数据段)
- 模块内有个特性，即代码段和数据段的位置固定。利用这个特性在为每个全局函数生成一个重定位条目，记录函数到GOT的偏移
- 在运行时，通过修改GOT数组，存放具体函数的地址，来实现调整。
- 每个外部函数都有自己的PLT数组，每个模块都有自己的GDT数组
  1. 代码访问到共享库函数，会先调用到它对于应的PLT代码段
  ```c
  # PLT[0]: call dynamic linker
  4005a0: pushq *GDT[1] // 给动态链接器的第一参数，是重定位段的地址 addr of
  reloc entries
  4005a6: jumpq *GDT[2] // 跳转到动态链接器 addr of dynamic linker
  ...
  # PLT[2]: call addvec()
  4005c0: jumpq *GDT[4] // addvec在GDT中的条目位置
  4005c6: pushq $0x1    // addvec在GDT用户函数的索引，0分配给系统启动函数sys
  startup，该例中addvec是第一个
  4005cb: jumpq 4005a0 // 跳转到PLT[0]执行
  ```
  2. PLT段在第一次调用时, 跳转到GDT条目，此时GDT条目只是简单但跳转回PLT的下一条指令。
  3. 先跳转到PLT[0]，押入重定位地址段，然后跳转到动态链接器执行
  4. 动态链接器根据重定位段和PLT传入的索引，计算出运行时函数地址，填入GDT条目。
- 之后每次执行，直接从GDT条目跳转到函数，不用再定位
